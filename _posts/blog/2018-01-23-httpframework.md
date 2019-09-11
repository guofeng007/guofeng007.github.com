---
layout: post
title: 手写SDK 项目之网络框架
categories: Blog
description:  SDK 项目之网络框架
keywords:   Android , SDK 项目，网络框架

---

# 手写Android SDK 项目网络框架

[原作者](https://juejin.im/post/5a61a848f265da3e253c3c06?utm_source=gold_browser_extension)



##  我的前言



作为一款 app 开发者而言，可能真的就是 rxjava+retrofit+okhttp 搞定网络请求了。但是假如你是做一个 sdk 的，比如地图 SDK，埋点 SDK,就不能这么搞了，因为你依赖了一个外部特定的 SDK，假如接入的 APP 也用了同样的库，但是版本不一样，新版本往往有些新的改动，这个时候你的 SDK 就不稳定了（或许有人会说，copy 一个版本，改个包名私有，只自己用不就可以了么，话虽如此，但是别人总是会一不小心被同样的类名，不同包名困惑，fuck stupid）。

所以作为一款 SDK 产品，必须自己封装一套稳定的 http 请求框架，我所在的部门就是这么干的，不过不适合分享。偶然看到有人写了这样一个东西，流程，原理，设计都是挺好的，可以给大家参考参考。这不是造轮子，这是造一体机（有些零部件必须自己生产）。

# **前言**

[手写Android网络框架——CatHttp（二）](https://link.juejin.im/?target=https%3A%2F%2Fjuejin.im%2Fpost%2F5a61a9b1518825732646e127)

在实际Android应用的开发中，网络请求往往是必不可少的。现在有很多优秀的开源网络框架如Volley、Okhttp和Retrofit等，说到框架，很多童鞋信手拈来，反手一个Okhttp+etrofit+RxJava全家桶。不就是网络请求么，so easy~

不过实际开发过程中，确实会出现各种各样的问题，比如你上传一张图片，服务器那边接收不到，怎么办呢？你看了下自己这边，完全按照标准api来写的，讲道理应该没错吧？这时候打开debug，可是框架内的代码怎么跟进？没看过也不懂啊，所以可能有些童鞋会去阅读源码，可是源码这种东西，不熟悉的话读起来晦涩难懂，当然边读边做源码分析写下几篇博客也是不错的选择。

不过其实最本质的，就是对你框架的业务熟悉，比如网络请求框架，你就必须熟悉Http协议，才可以了解你的表单是怎么封装成数据，以什么结果表示，怎么发出去，接收到的内容又是什么？了解了这些，我们完全可以参考优秀的源码，自己动手去实现一个简易版的。

![这里写图片描述](https://user-gold-cdn.xitu.io/2018/1/19/1610d7937b75704d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

# **Http协议**

谈到网络框架，就不得不说到http协议了，网络框架必须严格按照http协议才能保证客户端和服务器双方数据的正常传输。

## **Http请求**

一个http请求主要包含以下几个部分：**请求行**（request line）、**请求头**（header）、**空行**和**请求正文**四个部分。

以一个http请求为例：

```
GET /form.html HTTP/1.1 
Accept:image/gif,image/x-xbitmap,image/jpeg,application/x-shockwave-flash,application/vnd.ms-excel,application/vnd.ms-powerpoint,application/msword,*/*  Accept-Language:zh-cn 
Accept-Encoding:gzip,deflate
If-Modified-Since:Wed,05 Jan 2007 11:21:25 GMT 
If-None-Match:W/"80b1a4c018f3c41:8317" 
User-Agent:Mozilla/4.0(compatible;MSIE6.0;Windows NT 5.0)
Host:www.guet.edu.cn 
Connection:Keep-Alive 

```

- **请求行**：用来说明请求类型、要访问的资源及使用的Http版本。
- **请求头**：从第二行起为头部，用来说明服务器要是用的附加信息，一般以键值对的方式出现，中间以‘：’隔开，如 Accept-Language:zh-cn。常用的请求头请看：[Http请求头大全](https://link.juejin.im/?target=http%3A%2F%2Ftools.jb51.net%2Ftable%2Fhttp_header)，支持用户自定义请求头。
- **空行**：在请求头和请求正文中间会有一个空行用来分割，说明请求头和正文的区别。即使后面的正文为空，这里的空行也是必须的。
- **请求正文**：即我们要发送的数据，平时我们传入的表单、文件等，都会以正文的形式存在于请求正文中，如果是get、delete等请求，请求正文必须为空。

## **Http响应**

HTTP响应也由四个部分组成，分别是：**状态行**、**响应头**、**空行**和**响应正文**。 这里也以一段http响应为例：

```
HTTP/1.1 200 OK
Date: Fri, 22 May 2009 06:07:21 GMT
Content-Type: text/html; charset=UTF-8

<html>
      <head></head>
      <body>
            <!--body goes here-->
      </body>
</html>

```

- **状态行**：由HTTP协议版本号， 状态码， 状态消息 三部分组成。
- **响应头**：用来说明客户端要使用的一些附加信息，也是和上面的请求头相对应，以键值对的方式。
- **空行**：和请求的空行一样，这里的空行也是必须的，不管后面的正文是否为空。
- **响应正文**：服务器返回给客户端的文本信息（二进制形式）

![这里写图片描述](https://user-gold-cdn.xitu.io/2018/1/19/1610d793795d3352?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

# **技术选型**

大概了解了Http，我们就得选择一种具体的方式或者说一个比较底层的api来作为实现网络访问的实际参与者。大概有三种选择——Socket、HttpClient和HttpUrlConnection。如果是基于Socket那么我们需要实现的内容比较多，当然目前的OkHttp是采用这种方式来的，毕竟socket进行操作自由度比较高，如内部socket连接池的分配，长连接短连接等都可以控制，自由度较高。而HttpClient在Android6.0以后官方已经移除了这个api，而HttpUrlConnection则是一个比较好的选择，足够轻量级，又实现了一些基本需求，因此以HttpUrlConnection作为实际网络请求的参与者。

# **构建方式**

因为觉得Okhttp的构建方式很优雅，这里我们的构建方式就以OkHttp的方式进行构建，根据上面的Http协议的分析，结合OkHttp的构建方式，对对象的抽象其实也就一目了然了。必然是支持同步和异步的方式发起请求，所以我们的构建方式基本如下：

```
FormBody body = new FormBody.Builder()
                .add("username", "浩哥")
                .add("pwd", "abc")
                .build();

Request request = new Request.Builder()
                .url("http://192.168.31.34:8080/API/upkeep")
                .post(body)
                .build();
                
client.newCall(request).enqueue(new Callback() {
            @Override
            public void onResponse(Response response) {
                if (response.code() == 200) {
                    String msg = response.body().string();
                    Logger.e("response msg = " + msg);
                }
            }

            @Override
            public void onFail(Request request, IOException e) {
                e.printStackTrace();
            }
        });


```

主要抽象出的对象包括：**Request**、**RequestBody**、**Response**、**ResponseBody**、 **Call**、**CallBack**等。

## **Request请求的构建**

请求怎么构建呢？结合上面对http协议的分析，请求包括起始行、请求头、空行和请求正文。因为基于HttpUrlConnection，所以起始行和空行可以不用考虑，请求头需要一个临时的暂存空间，请求正文由于不同类型格式也不同，因此请求正文给一个抽象的基类。

### **RequestBody**

requestBody主要负责对流的写出和ContentType类型的构建，因为不同类型如表单和文件的Content-Type内容是不一致的，服务器那边解析方式自然也是不一样的。

```
public abstract class RequestBody {

    /**
     * body的类型
     *
     * @return
     */
    abstract String contentType();

    /**
     * 将内容写出去
     *
     * @param ous
     */
    abstract void writeTo(OutputStream ous) throws IOException;

}

```

## **Request**

请求这块存储了url，和请求的方法类型，用ArrayMap来存储请求头，同时持有一个RequestBody的引用，均可以通过建造者模式构建进来。

```
public class Request {

    final HttpMethod method;
    final String url;
    final Map<String, String> heads;
    final RequestBody body;

    public Request(Builder builder) {
        this.method = builder.method;
        this.url = builder.url;
        this.heads = builder.heads;
        this.body = builder.body;
    }


    public static final class Builder {

        HttpMethod method;
        String url;
        Map<String, String> heads;
        RequestBody body;

        public Builder() {
            this.method = HttpMethod.GET;
            this.heads = new ArrayMap<>();
        }

        Builder(Request request) {
            this.method = request.method;
            this.url = request.url;
        }


        public Builder url(String url) {
            this.url = url;
            return this;
        }

        public Builder header(String name, String value) {
            Util.checkMap(name, value);
            heads.put(name, value);
            return this;
        }

        public Builder get() {
            method(HttpMethod.GET, null);
            return this;
        }

        public Builder post(RequestBody body) {
            method(HttpMethod.POST, body);
            return this;
        }

        public Builder put(RequestBody body) {
            method(HttpMethod.PUT, body);
            return this;
        }

        public Builder delete(RequestBody body) {
            method(HttpMethod.DELETE, body);
            return this;
        }

        public Builder method(HttpMethod method, RequestBody body) {
            Util.checkMethod(method, body);
            this.method = method;
            this.body = body;
            return this;
        }

        public Request build() {
            if (url == null) {
                throw new IllegalStateException("访问url不能为空");
            }
            if (body != null) {
                if (!TextUtils.isEmpty(body.contentType())) {
                    heads.put("Content-Type", body.contentType());
                }
            }
            heads.put("Connection", "Keep-Alive");
            heads.put("Charset", "UTF-8");
            return new Request(this);
        }
    }


    public enum HttpMethod {
        GET("GET"),
        POST("POST"),
        PUT("PUT"),
        DELETE("DELETE");

        public String methodValue = "";

        HttpMethod(String methodValue) {
            this.methodValue = methodValue;
        }

        public static boolean checkNeedBody(HttpMethod method) {
            return POST.equals(method) || PUT.equals(method);
        }

        public static boolean checkNoBody(HttpMethod method) {
            return GET.equals(method) || DELETE.equals(method);
        }
    }

}


```

这样整个请求块也就构建完毕了。剩下的无非是对具体请求体的抽象的具体实现，我们再看看响应那边怎么实现的。

# **Response响应的构建**

## **ResponseBody**

响应体这块主要存储为字节，可以转换成String类型进行返回，不做更具体的解析，没有直接提供流的原因是设计上回调是在主线程中的，如果把流传入有需要自己做异步处理。

```
public class ResponseBody {

    byte[] bytes;

    public ResponseBody(byte[] bytes) {
        this.bytes = bytes;
    }

    public byte[] bytes() {
        return this.bytes;
    }

    public String string() {
        try {
            return new String(bytes(), "UTF-8");
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        return "";
    }
}

```

## **Response**

response相对就比较简单了，最关键的是服务端的返回码和响应正文。

```
public class Response {

    final ResponseBody body;
    final String message;
    final int code;

    public Response(Builder builder) {
        this.body = builder.body;
        this.message = builder.message;
        this.code = builder.code;
    }


    public ResponseBody body() {
        return this.body;
    }

    public int code() {
        return this.code;
    }

    public String message() {
        return this.message;
    }


    static class Builder {

        private ResponseBody body;
        private String message;
        private int code;

        public Builder body(ResponseBody body) {
            this.body = body;
            return this;
        }

        public Builder message(String message) {
            this.message = message;
            return this;
        }

        public Builder code(int code) {
            this.code = code;
            return this;
        }

        public Response build() {
            if (message == null) throw new NullPointerException("response message == null");
            if (body == null) throw new NullPointerException("response body == null");
            return new Response(this);
        }

    }

}

```

这样基本的请求和响应对象构建好了，中间需要向上面构建的方式进行调用，还需要引入Call和Callback作为请求的发起和回调接口。

# **请求发起和结果回调**

# **Call**

call支持同步和异步方式的调用，同步直接返回Response，方法内部阻塞，异步提供一个回调接口回调结果。

```
public interface Call {


    /**
     * 同步执行
     *
     * @return response
     */
    Response execute();

    /**
     * 异步执行
     *
     * @param callback 回调接口
     */
    void enqueue(Callback callback);

}

```

## **Callback**

Callback 作为回调接口，提供成功和失败的回调，当访问网络成功并成功拿到数据则进入成功的回调，否则进入失败的回调。

```
public interface Callback {

    /**
     * 当成功拿到结果时返回
     *
     * @param response &emsp;返回结果
     */
    void onResponse(Response response);


    /**
     * 当获取结果失败时
     *
     * @param request &emsp;请求
     * @param e       &emsp;Http请求过程中可能产生的异常
     */
    void onFail(Request request, IOException e);

}

```

还有一个关键的就是我们客户端——CatHttp了。

## **CatHttpClient**

CatHttpClient 主要配置了一些超时信息之类的，主要是作为客户端的抽象，作为Call(这一呼叫服务端连接动作的外部发起者)。

```
public class CatHttpClient {

    private Config config;

    public CatHttpClient(Builder builder) {
        this.config = new Config(builder);
    }

    public Call newCall(Request request) {
        return new HttpCall(config, request);
    }

    static class Config {
        final int connTimeout;
        final int readTimeout;
        final int writeTimeout;

        public Config(Builder builder) {
            this.connTimeout = builder.connTimeout;
            this.readTimeout = builder.connTimeout;
            this.writeTimeout = builder.writeTimeout;
        }
    }

    public static final class Builder {
        private int connTimeout;
        private int readTimeout;
        private int writeTimeout;

        public Builder() {
            this.connTimeout = 10 * 1000;
            this.readTimeout = 10 * 1000;
            this.writeTimeout = 10 * 1000;
        }


        public Builder readTimeOut(int readTimeout) {
            this.readTimeout = readTimeout;
            return this;
        }

        public Builder connTimeOut(int connTimeout) {
            this.connTimeout = connTimeout;
            return this;
        }

        public Builder writeTimeOut(int writeTimeout) {
            this.writeTimeout = writeTimeout;
            return this;
        }

        public CatHttpClient build() {
            return new CatHttpClient(this);
        }

    }

}


```

这样，请求和响应还有请求和回调的接口都约定好了，关键的就在于任务的执行过程和任务的调度了，因为网络请求都是耗时的，所以必然需要异步去处理网络请求才能最大的发挥框架的性能。我们需要构建一个具体的任务——Task。

# **Task的执行**

## **HttpTask**

可以看到，HttpTask实现了Runnable接口，内部实际访问网路请求的操作交给了IRequestHandler来做，回调交给了IResponseHandler来做，最终拿到了Response结果

```
public class HttpTask implements Runnable {

    private HttpCall call;
    private Callback callback;
    private IRequestHandler requestHandler;
    private IResponseHandler handler = IResponseHandler.RESPONSE_HANDLER;

    public HttpTask(HttpCall call, Callback callback, IRequestHandler requestHandler) {
        this.call = call;
        this.callback = callback;
        this.requestHandler = requestHandler;
    }

    @Override
    public void run() {
        try {
            Response response = requestHandler.handlerRequest(call);
            handler.handlerSuccess(callback, response);
        } catch (IOException e) {
            handler.handFail(callback, call.request, e);
            e.printStackTrace();
        }
    }
}

```

## **IRequestHandler**

IRequestHandler是实际网络请求的发起者，因为是面向接口编程，外部不用管内部的实现细节，只要调用方法拿到结果就行了。

```
public interface IRequestHandler {

    /**
     * 处理请求
     *
     * @param call &emsp;一次请求发起
     * @return 应答
     * @throws IOException &emsp;网络连接或者其它异常
     */
    Response handlerRequest(HttpCall call) throws IOException;

}

```

## **IResponseHandler**

看到这里应该明白，这里无非就是包装了一层，实际内部是调用了handler的post(Runnable r)方法将结果回调到主线程中，也就是Callback接口的回调方法被我们切换到了主线程中执行。

```
public interface IResponseHandler {

    /**
     * 线程切换,http请求成功时的回调
     *
     * @param callback &emsp;回调接口
     * @param response &emsp;返回结果
     */
    void handlerSuccess(Callback callback, Response response);

    /**
     * 线程切换,http请求失败时候的回调
     *
     * @param callback &emsp;回调接口
     * @param request  &emsp;请求
     * @param e        &emsp;可能产生的异常
     */
    void handFail(Callback callback, Request request, IOException e);


    IResponseHandler RESPONSE_HANDLER = new IResponseHandler() {

        Handler HANDLER = new Handler(Looper.getMainLooper());

        @Override
        public void handlerSuccess(final Callback callback, final Response response) {
            Runnable runnable = new Runnable() {
                @Override
                public void run() {
                    callback.onResponse(response);
                }
            };
            execute(runnable);
        }

        @Override
        public void handFail(final Callback callback, final Request request, final IOException e) {
            Runnable runnable = new Runnable() {
                @Override
                public void run() {
                    callback.onFail(request, e);
                }
            };
            execute(runnable);
        }


        /**
         * 移除所有消息
         */
        public void removeAllMessage() {
            HANDLER.removeCallbacksAndMessages(null);
        }

        /**
         * 线程切换
         * @param runnable
         */
        private void execute(Runnable runnable) {
            HANDLER.post(runnable);
        }

    };

}

```

# **任务调度**

可以看到，上面所有的内容，就差一点能够全部连通，就在于任务的调度，也就是调用线程的执行，必然在Call实体类的enqueue和execute方法中通过任务调度来执行Runnable内部的逻辑的。

## **HttpThreadPool**

可以看到，作为一个单例类，内部对外提供了同步执行和异步执行task的接口，内部通过线程池来实现，采用生产者-消费者模式，所有客户端提交的任务都会先进入到无界队列BlockingQueue中，线程池满的拒绝策略也是将当前无法被执行的任务放入BlockingQueue中，而在一开始就开了一个Runnable死循环从BlockingQueue中不断取任务执行。

```
public class HttpThreadPool {

    /**
     * 线程核心数
     */
    public static final int CORE_POOL_SIZE = Runtime.getRuntime().availableProcessors();

    /**
     * 最大存活时间
     */
    public static final int LIVE_TIME = 10;

    /**
     * 单例对象
     */
    private static volatile HttpThreadPool threadPool;

    /**
     * 无界队列
     */
    private BlockingQueue<Future<?>> queue = new LinkedBlockingQueue<>();

    /**
     * 线程池
     */
    private ThreadPoolExecutor executor;

    public static HttpThreadPool getInstance() {
        if (threadPool == null) {
            synchronized (HttpThreadPool.class) {
                if (threadPool == null) {
                    threadPool = new HttpThreadPool();
                }
            }
        }
        return threadPool;
    }

    private HttpThreadPool() {
	   executor = new ThreadPoolExecutor(CORE_POOL_SIZE, CORE_POOL_SIZE+1, LIVE_TIME , TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(4), rejectHandler);
       executor.execute(runnable);
    }

    /**
     * 消费者
     */
    Runnable runnable = new Runnable() {
        @Override
        public void run() {
            while (true) {
                FutureTask<?> task = null;
                try {
                    task = (FutureTask<?>) queue.take();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                if (task != null) {
                    executor.execute(task);
                }
            }
        }
    };


    /**
     * 同步提交任务
     *
     * @param task &emsp;任务
     * @return response对象
     * @throws ExecutionException
     * @throws InterruptedException
     */
    public synchronized Response submit(Callable<Response> task) throws ExecutionException, InterruptedException {
        if (task == null) throw new NullPointerException("task == null , 无法执行");
        Future<Response> future = executor.submit(task);
        return future.get();
    }


    /**
     * 添加异步任务
     *
     * @param task
     */
    public void execute(FutureTask<?> task) {
        if (task == null) throw new NullPointerException("task == null , 无法执行");
        try {
            queue.put(task);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    /**
     * 拒绝策略，如果当线程池中的阻塞队列满，则添加到link队列中
     */
    RejectedExecutionHandler rejectHandler = new RejectedExecutionHandler() {
        @Override
        public void rejectedExecution(Runnable runnable, ThreadPoolExecutor threadPoolExecutor) {
            try {
                queue.put(new FutureTask<>(runnable, null));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    };

}

```

# 结语

可以看到，上面除了调度的HttpThreadPool类，其余类基本都是抽象类或者接口，但是上面的这些接口和抽象类，相信看懂的童鞋应该明白，网络框架已经可以"运行"了。这里的运行当然不是说能在编译器或者具体的手机上运行，但是框架内部已经打通了任督二脉，可以完美的调度了。代码在github上——[传送门](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fpan-haos%2FCatHttp)