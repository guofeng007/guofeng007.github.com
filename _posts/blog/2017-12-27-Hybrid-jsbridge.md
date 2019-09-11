---
layout: post
title: Hybrid JsBridge 通信原理概述
categories: Blog
description:  Hybrid,JsBridge 通信模型
keywords:   Hybrid,JsBridge,通信模型

---



# Hybrid JsBridge 通信原理概述

 

**作者：国风**


# 一、背景
FE(FrontEnd)大前端作为互联网的奠基人，从 www 之初到现在，从未停止创新的角度。但是传统的大前端为了跨平台，只能完成通用的业务功能，对硬件几乎无能为力，很多移动终端特性无法使用，比如扫一扫，二维码，麦克风，相机，音频，视频等。在移动互联网的浪潮下，这些都是标配，怎样让大前端具备这些能力呢？最好的解决方案便是 JsBridge（Js 桥），桥的含义就是链接这端和那端，所以 JsBridge 就是链接 JavaScript 和 终端 APP 的桥。通过这个桥，大前端可以进化为移动大前端了。

# 二、通信原理

本文移动端以安卓为基础，介绍通信原理。由于 Android 原生采用 Java 语言开发，所以核心通信问题就是 JS和 Java 的通信：

  ![bridge](/images/posts/HybridBridge/bridge.png)



## 2.1 Javascript调用Java

在Android开发中，能实现Javascript调用Native代码通信的，有4种途径：

1. JavascriptInterface(系统原生4.2以下存在注入漏洞，[参见](http://www.tuicool.com/articles/jeYVFrN))

但是这个官方提供的解决方案在Android4.2之前存在[安全漏洞](http://drops.wooyun.org/papers/548)。在Android4.2之后，加入了@JavascriptInterface才得到解决。所以考虑到兼容低版本的系统，JavascriptInterface并不适合。

2. WebViewClient.shouldOverrideUrlLoading()

这个方法的作用是拦截所有WebView的Url跳转。页面可以构造一个特殊格式的Url跳转，shouldOverrideUrlLoading拦截Url后判断其格式，然后Native就能执行自身的逻辑了。

3. WebChromeClient.onConsoleMessage()

这是Android提供给Javascript调试在Native代码里面打印日志信息的API，同时这也成了其中一种Javascript与Native代码通信的方法。在Javascript代码中调用console.log('xxx')方法。

4. WebChromeClient.onJsPrompt()

WebChromeClient.onJsPrompt()，onJsAlert()和WebChromeClient.onJsConfirm()。顾名思义，这三个 Javascript给Native代码的回调接口的作用分别是提示展示提示信息，展示警告信息和展示确认信息。鉴于，alert和confirm在 Javascript的使用率很高，所以JSBridge的解决方案中都倾向于选用onJsPrompt()。



当前主流解决方案是：

一、4.2以下系统使用第四种方式（考虑安全性） onJsPrompt,4.2以上用第一种方式（考虑到性能）

二、统一采用 onJsPrompt 方式

建议的方式是方案一，毕竟随着安卓的发展，后续原生通信方式将会更加可靠安全：

![JscallJava](/images/posts/HybridBridge/JscallJava.png)

## 2.2 Java调用Javascript

1.安卓4.4之前，通过webView.loadUrl("javascript:scriptString"); 

2.在安卓4.4以后，官方新增api webView.evaluateJavascript(scriptString,ValueCallback);

专门用于执行java调js方法，在4.4及以上系统建议使用这个方法。

注意，4.4以上 targetapi>19,如果仍然采用 loadUrl，会产生导航栈后退混乱问题，主要原因是 API19以后，loadUrl 做了严格校验，如果直接 loadUrl 执行脚本，会认为是一个空白页面，导致导航混乱。

![JavaCallJs](/images/posts/HybridBridge/JavaCallJs.png)

### 2.3 OnJsPrompt 原理解析

onJsPrompt的通信原理

1.初始化

​    通过反射注入对象的方法列表，将addJavascriptInterface保存的注入对象转换为一系列jsfunction字符串

2.注入

​    在onPageStart和onProgressChanged(进度大于25%)中，通过loadUrl或者evaluateJavascript将第一步生成的js 字符串注入到当前页面。

3.Js调用

​    Js执行注入对象的方法BLightApp.method()，在method内部，将参数封装为固定协议格式，然后调用jsPrompt()

![JsBlightapp](/images/posts/HybridBridge/JsBlightapp.png)

4.Native执行

​    Native在onJsPrompt中根据协议解析参数，并通过反射，调用Native对象的方法。

![JsInvoke](/images/posts/HybridBridge/JsInvoke.png)

#  三、安全性

webview 作为整个大前端的容器，其安全性更是重中之重，安全性主要考虑如下方面：

![safe](/images/posts/HybridBridge/safe.png)

# 四、调试方法

**调试环境:**

1.最新版本Chrome浏览器

2.安卓手机系统大于等于4.4

**步骤：**

1.在需要调试的Webview中启用调试

WebView.setWebContentsDebuggingEnabled(true);

2.确认手机已经连接到电脑（adbdevices 能看到device online）

3.在手机上用该webview打开一个网址

4.在Chrome中输入about://inspect，就能看到当前webview会话，点击需要调试的会话

5.使用chrome的调试工具（F12）,就可以像调试PC上的h5一样进行调试。

# 五、推荐的开源框架

了解到上述原理之后，并不推荐直接早轮子，因为现有很多成熟的开源框架，功能完备，如：

JsBridge,建议使用这个，然后搭配 我的另外一片博客《Router》，彻底将 Native 端能力完整暴露给 H5。