---
layout: post
title:  OkHttpDNS实现原理
categories: Blog
description:   OkHttpDNS实现原理
keywords:    OkHttpDNS,实现原理

---

# OkHttpDNS实现原理

[原文作者](https://www.jianshu.com/p/9803a6efb672)
作者在解析 HttpDNS之后，并没有把解析的结果包装为 InetAddress，而是直接调用了 InetAddress.getByName 方法，方法仍然是调用系统原生 dns 查询，所以我写了一个 IPUtils 库，安全的解析返回结果（出现任何异常都会走系统解析，保证安全）

The Application of  HTTP DNS on OkHttp.

------

## 1. HTTP DNS 的介绍

HTTP DNS通过将域名查询请求放入http中的一种域名解析方式，而不用系统自带的libc库去查询运营商的DNS服务器，有更大的自由度。目前微信，qq邮箱、等业务均使用了HTTP DNS，详见[这里](https://link.jianshu.com?t=http://mp.weixin.qq.com/s?__biz=MzA3ODgyNzcwMw==&mid=201837080&idx=1&sn=b2a152b84df1c7dbd294ea66037cf262&scene=2&from=timeline&isappinstalled=0&utm_source=tuicool)

主要优点

- 能够准确地将站点解析到离用户最近的CDN站点，方便进行流量调度
- 解决部分运营商DNS无法解析国外站点的问题
- TCP在一定程度可以防止UDP无校验导致的DNS欺诈（比如墙，运营商广告，404导航站），当然基于HTTP的话本质还是**不安全**的。

## 2. HTTP DNS 与 native DNS 的流程对比

使用http与native的区别主要在：

- 需要实现native 自带的 LRU缓存（DNS默认是600s，原生的缓存在JNI内存中，而OkHttp缓存在硬盘中）
- Socket请求拼装环节在java层，而不是在libc库中（当然有的项目非要做成JNI，也没办法）
- 需要维护一个单独的HttpClient进行DNS查询

在进行实现前，我们先看下原生情况的查询方法的strace

#### 2.1. Android下native DNS查询过程

```
//JDK层调用
InetAddress.lookupHostByName()(InetAddress.java);
Libcore.os.android_getaddrinfo(InetAddress.java)
//framework下调用
libcore.io.ForwardingOs.getaddrinfo(ForwardingOs.java)
//framework层
libcore.io.Posix.getaddrinfo(Posix.java)
//JNI层，此方法为java的代理
Posix_android_getaddrinfo(Posic.cpp)
//调用BIONIC的libc标准库
android_getaddrinfofornet(getaddinfo.c)
android_getaddrinfofornetcontext(getaddinfo.c)
...

发送socket包...

```

#### 2.2. HTTP DNS流程

```
//拼装HTTP请求(OkHttp)
client.newCall
//JDK层，进行socket请求
Socket.connect(socket.java)
//framework层
libcore.io.IoBridge.connect
//进行tcp连接(JNI/C)
....

```

## 3. HTTP DNS的实现

在OkHttp中，提供了DNS的接口（这个接口基本没人知道，甚至网上都没有讨论），方便用户自己实现lookup方法。下文实现了两个OkHttpClient单例，一个负责dns，一个负责业务。

> 关于OkHttp，可以看[以前的文章](https://www.jianshu.com/p/aad5aacd79bf)

经过考察第三方DNS服务，看起来只有腾讯系的DNSPod还是不错的，它直接返回一个ip地址，不需要解析json等乱七八糟的东西，文档在[这里](https://link.jianshu.com?t=https://www.dnspod.cn/misc/D%2B%E5%85%8D%E8%B4%B9%E7%89%88%E6%9C%AC%E6%8E%A5%E5%8F%A3%E8%AF%B4%E6%98%8E.pdf)。

DNSPod在返回的Header中，没有设置缓存，并主动断开了Socket连接，这样的话每次进行lookup时都会进行一个GET请求，相比以前原生多了40ms。我们都知道DNS默认的TTL时间一般是600s，因此我们可以通过复用OkHttp中的cache，在返回的response中加入`Cache-Control`的Header就可以实现再第二次lookup时避免再次访问网络。

```
public OkHttpClient getHTTPDnsClient() {
  if (httpDnsclient == null) {
    final File cacheDir = getExternalCacheDir();
    httpDnsclient = new OkHttpClient.Builder()
        //消费者工作线程池
        .dispatcher(getDispatcher())
        //Logger拦截器     
        .addNetworkInterceptor(getLogger())
        .addNetworkInterceptor(new Interceptor() {
          @Override public Response intercept(Chain chain) throws IOException {
            Response originalResponse = chain.proceed(chain.request());
            return originalResponse.newBuilder()
                //在返回header中加入缓存消息
                //下次将不再发送请求
                .header("Cache-Control", "max-age=600").build();
          }
        })
        //5MB的文件缓存
        .cache(new Cache(new File(cacheDir, "httpdns"), 5 * 1024 * 1024))
        .build();
  }
  return httpDnsclient;
}

```

> max-age表示可以续600s；如果缓存命中将返回数据，反之抛出IOException。

通过测试，HTTP DNS工作效果如下：

1. 在600s内，无论是否联网，都不会进行请求
2. 在600s外，没联网时会抛出IOException，反之会进行HTTP请求

接着，我们实现了DNS查询接口，值得注意的一点就是第三方服务器可能会挂，所以一定要注意所有异常并留出后路。

```
static Dns HTTP_DNS =  new Dns(){
  @Override public List<InetAddress> lookup(String hostname) throws UnknownHostException {
    //防御代码
    if (hostname == null) throw new UnknownHostException("hostname == null");
    //dnspod提供的dns服务
    HttpUrl httpUrl = new HttpUrl.Builder().scheme("http")
        .host("119.29.29.29")
        .addPathSegment("d")
        .addQueryParameter("dn", hostname)
        .build();
    Request dnsRequest = new Request.Builder().url(httpUrl).get().build();
    try {
      String s = getHTTPDnsClient().newCall(dnsRequest).execute().body().string();
      //避免服务器挂了却无法查询DNS
      if (!s.matches("\\b(?:\\d{1,3}\\.){3}\\d{1,3}\\b")) {
        return Dns.SYSTEM.lookup(hostname);
      }
      return Arrays.asList(InetAddress.getAllByName(s));
    } catch (IOException e) {
      return Dns.SYSTEM.lookup(hostname);
    }
  }
};

```

在真正的业务请求时，构建`OkHttpClient`时对DNS进行配置就ok了

```java
//真正的调用客户端，供Retrofit与Picasso使用
static public synchronized OkHttpClient getClient() {
  if (client == null) {
    final File cacheDir = GlobalContext.getInstance().getExternalCacheDir();
    client = new OkHttpClient.Builder()
        .cache(new Cache(new File(cacheDir, "okhttp"), 60 * 1024 * 1024))
        //配置DNS查询实现
        .dns(HTTP_DNS)
        .build();
  }
  return client;
}

```

## 4. 例子

Fork me on [Github](https://github.com/guofeng007/OkHttpDNS），有更多交流可以与我微信沟通，在深圳的开发者可以与我面基。