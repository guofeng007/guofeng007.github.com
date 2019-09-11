---
layout: post
title: Facebook Sonar 移动端调试神器
categories: Blog
description:  Facebook Sonar 移动端调试神器
keywords:      Facebook Sonar 移动端调试神器

---

# Facebook Sonar 移动端调试神器
![img](https://fbsonar.com/img/SonarKit.png)

[官网](https://fbSonar.com/docs/getting-started.html)、[github](https://github.com/facebook/Sonar)、[Demo地址](https://github.com/guofeng007/FacebookSonarDemo)

## 简介

Sonar是Android/iOS开发调试工具，包含桌面查看器(仅限 Mac),Android/iOS 调试 SDK。前身是Stetho,仅支持 Android，所以 Facebook 继续深入开发，诞生了 Sonar，大家也可以去了解下Stetho： [github](https://github.com/facebook/stetho/), [官网](http://facebook.github.io/stetho/)

## 使用

### 下载 Mac 调试查看器
 [地址](https://www.facebook.com/Sonar/public/mac)
![img](https://fbsonar.com/docs/assets/initial.png)

### Android app 配置

#### 1. app build.gradle 中增加依赖配置

 ```
 repositories {
  jcenter()
}

dependencies {
  debugCompile 'com.facebook.Sonar:Sonar:0.0.1'
}

 ```

#### 2. Application onCreate 配置初始化代码

```
 public class MyApplication extends Application {
  @Override
  public void onCreate() {
    super.onCreate();
    SoLoader.init(this, 0);
    if (BuildConfig.DEBUG && SonarUtils.shouldEnableSonar(this)) {
      final SonarClient client = AndroidSonarClient.getInstance(this);
      client.addPlugin(new MySonarPlugin());
      client.start();
    }
  }
}
```

#### 3. 配置额外插件
  - Log 是不需要配置的，默认支持
  - Layout Inspector Inplugin(支持第三方界面框架 [Litho](https://fblitho.com/) 和 [ComponentKit](https://componentkit.org/))

  ```
  import com.facebook.Sonar.plugins.inspector.DescriptorMapping;
  import com.facebook.Sonar.plugins.inspector.InspectorSonarPlugin;

  final DescriptorMapping descriptorMapping = DescriptorMapping.withDefaults();
  client.addPlugin(new InspectorSonarPlugin(mApplicationContext, descriptorMapping));

  ```
  
  - Network 网络查看插件(文档只介绍如下)

  ```
  importimport com.facebook.Sonar.plugins.network.NetworkSonarPlugin;
  client.addPlugin( com.fa new NetworkSonarPlugin());
  ```
   **其实只能用 okhttp，而且需要设置 Interceptor,其他网络框架或者原生 httpurlconnection 是不支持的,看了官方 demo 才知道**

 ```
  SonarOkhttpInterceptor interceptor = new SonarOkhttpInterceptor(networkPlugin);
  okhttpClient = new OkHttpClient.Builder()
  .addNetworkInterceptor(interceptor)
  .connectTimeout(60, TimeUnit.SECONDS)
  .readTimeout(60, TimeUnit.SECONDS)
  .writeTimeout(10, TimeUnit.MINUTES)
 ```

#### 4. 自定义插件

  具体参见官网，包含桌面版本(基于 React),和手机端插件。本质就是互相收发采集或者展示的信息，查看状态。

## 踩过的坑
1. Sonar 居然把 support-v4,okhttp 等库直接打入到 aar,导致 app 不得不去掉本身的 support,okhttp,只能使用 Sonar 内置的版本，这个问题不知道后面会不会解决。
2. Sonar so 仅支持 x86,armeabi-v7a,app 中注意假如 armFilter 过滤其他 arch
3. 机型兼容性问题，Android4.4无任何提示，Android7.0上可以正常使用，问题待定 

## 经验总结
1. 有问题先看看 github 官方 demo 如何处理
2. github issue 里面是否有人解决
3. 文档仔细查阅+stackoverflow
4. google---->百度
5. ask 同事、导师、朋友、技术群