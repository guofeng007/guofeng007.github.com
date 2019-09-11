---
layout: post
title: 微信安卓端小程序逆向分析
categories: Blog
description: 小程序容器原理
keywords: 微信、小程序
---



# 微信安卓端小程序逆向分析--它山之石可以攻玉

 

**作者：国风**

​        最近微信小程序在互联网上火了一把，具体未来怎么样暂时还不是那么清晰，但是从大家的关注度来看，还是非常可观的，微信小程序有这么大的用户量以及关注度，可见其设计独到之处，因此本文对安卓端小程序进行逆向分析。在Android应用开发中，软件安全和逆向分析非常重要。俗话说：知己知彼，百战不殆。如何知己，就需要对应用程序进行反编译。反编译就是逆向工程(ReverseEngineering), 通过解析App的内容, 可以查看并分析应用的构造与实现，当然也可以学习参考其优秀的设计实现方案。本文首先对安卓系统下APP逆向方法和工具进行介绍，然后对微信小程序进行逆向分析、并总结其设计方案。

#1. APP反编译流程

## 1.1 APP的构成

​        APP本质是一个包含代码和资源文件的压缩包，所以可以先对其进行解压以分析其构成，其内部结构如下图1。

![img](/images/posts/wechat-mini-app/clip_image002.jpg)

图 1 APP内部结构 

- META-INF目录
  ​     META-INF目录下存放的是签名信息，用来保证apk包的完整性和系统的安全。在eclipse编译生成一个apk包时，会对所有要打包的文件做一个校验计算，并把计算结果放在META-INF目录下。这就保证了apk包里的文件不能被随意替换。比如拿到一个apk包后，如果想要替换里面的一幅图片，一段代码， 或一段版权信息，想直接解压缩、替换再重新打包，基本是不可能的。如此一来就给病毒感染和恶意修改增加了难度，有助于保护系统的安全。


- res目录
  ​     res目录存放资源文件。包括图片，字符串等等。解包后，几乎所有可能的修改和编辑工作基本都在这里。


- assets目录
  ​     assets目录可以存放一些配置文件，这些文件的内容在程序运行过程中可以通过相关的API获得。 


- resources.arsc 
  ​     编译后的二进制资源文件。 


- classes.dex文件 
  ​     classes.dex是java源码编译后生成的Dalvik字节码文件。dex是Dalvik VM
  ​     executes的全称，即Android
  ​     Dalvik执行程序，并非Java
  ​     ME的字节码而是Dalvik字节码 (5.0以后是ART)。这里需要注意的是dex文件并不是java字节码、而是smali语言的字节码文件。因为在安卓系统中运行的是Dalvik虚拟机，是谷歌专门为移动端设计的基于寄存器的虚拟机，经过特殊优化，其性能优于JVM。而Java虚拟机基于**栈**。 基于栈的机器必须使用指令来载入和操作栈上数据，所需指令更多,并不适合移动端环境。


- AndroidManifest.xml
  ​     该文件是每个应用程序都必须定义和包含的文件，它描述了应用程序的名字、版本、权限、引用的库文件等等信息。需要解包后才能加以阅读。

## 1.2 APP子模块反编译流程

​        **代码反编译**
​        APP中class.dex是代码编译后的产物，所以代码反编译主要是处理class.dex文件。 APP中class.dex是代码编译后的产物，所以代码反编译主要是处理class.dex文件,其处理流程如下

- 代码反编译工具: **dex2jar**

```java
d2j-dex2jar.bat classes.dex
```

 

- 代码查看工具:**jd-gui**

![img](/images/posts/wechat-mini-app/clip_image003.jpg)

图2 代码查看工具jd-gui

​        **资源反编译**
​        由于APP中的资源在打包的时候都进行了编码、压缩等处理，所以无法直接查看界面布局xml文件，或者资源常量等文件。因此需要对资源进行还原，还原的工具有很多，工具的选取由具体需求决定。

manifest文件查看

​        如果只是查看APP的组件声明文件，大可不必全部反编译整个APP，因为全部反编译比较耗时，同时也会产生很多不需要的文件。在这里我们使用的工具是**AXMLPrinter2.jar**，我们只需要从APP的压缩包中取出AndroidManifest.xml文件，然后执行如下命令即可，最终产物为文本文件，可以直接使用各种常见工具查看：

```java
java -jar AXMLPrinter2.jar AndroidManifest.xml > PrimAndroidManifest.xml
```

​        **一键反编译**

​        如果只是反编译查看整个APP，则可以使用Jadx，ClassyShark，Smali2Java等优秀的工具。如果上面两个工具不能满足需求，那么剩下的就交给**ApkTool**。这个工具可以完整的解开一个APP，不过代码部分反编译为smali语言，因为最终APP编译产物是运行在Dalvik(5.0以后是ART)虚拟机上面，而该虚拟机执行的语言是优化过的基于寄存器的Smali语言，所以我们反编译也不能跨级。建议大家有空学习一下smali的语法，和汇编语言类似。（dex2jar其实是做了一个中转操作，将dex格式的smail转换为java）

```java
java -jar apktool.jar d wechat.apk wechat
```

​        如果有特殊需求，譬如破解APK之类的，可以直接修改smaili文件，然后再次通过ApkTool在打包成原来的APP(个人学习使用，建议大家谨慎使用，不要用来获取非法利益)。

```java
java -jar apktool.jar b wechat
```

​        当然打包之后签名肯定没法和以前一模一样，只能使用自己的签名给APK文件签名。由于安卓的开放机制，GooglePlay或者国内的应用商店并没有强制审核或者颁发签名，每个公司或者组织都可以自己生成签名。签名在系统中的保护机制也仅仅是覆盖安装的时候，校验安装前后签名是否一直。所以我们重新打包签名的APP不能覆盖安装，只能卸载掉原来的APP，然后才能安装我们修改过的APP。

​        这里在稍微深入一点点，介绍一下如何保证破解之后的APP能够正常运行。很多APP都有一定的防止二次编译校验功能，如果你修改了APP，那么你修改后的APP签名肯定和原来的不一样，所以APP会增加一层签名校验，如果校验不通过，会提示用户安装包已经被篡改，请重新下载，其校验方式有三种（这里只是简单介绍，具体问题需要具体分析）：

- Java层校验:有两种方式可以绕过此校验：一、hook     getPackageInfo API，在返回签名的时候，返回原APP签名；二、直接修改其逻辑，在smali代码中找到if eq     getPackageInfo的逻辑，修改为if     neq。
- NDKNative层校验:寻找其调用签名验证的地方，然后修改其跳转逻辑（具体方法可以在实战中寻找）。
- 服务器校验:查找其关键请求参数，在请求之前，将原来的APP正常参数上传，以绕过服务端校验。

​    **DIY工具集**

​    上述介绍的工具均为官方或者正规渠道，开源的安卓系统工具链也衍生了一些列“傻瓜工具集”，如：一键拖拽ApkTool，以及各种论坛中出现的APKIDE等工具。这些工具通过包装基本的apktool，dex2jar，jdgui等工具，形成功能更加丰富的反编译工具集合，也不失为反编译工具链的一个补充。

​        有了以上反编译的基础知识和工具，接下来将对微信小程序模块进行分析和总结。

# 2. 微信小程序实现逆向分析

​        微信小程序是一种不需要下载安装即可使用的应用，它实现了应用“触手可及”的梦想，用户扫一扫或者搜一下即可打开应用，也体现了“用完即走”的理念，用户不用关心是否安装太多应用的问题。应用将无处不在，随时可用，但又无需下载。其本质为订制化的H5容器框架，需要单独开发、适配。但是微信对小程序的入口也增加了一些额外的限制，如不支持二维码、不希望线上的公众号图文页和朋友圈成为小程序入口、不支持分享到朋友圈不支持挂在公众号菜单和阅读原文中等，限制了其传播方式。以上是对微信小程序的简单介绍，接下来将对微信小程序模块进行逆向分析。

## 2.1 望闻问切

​        望闻问切是扁鹊创的四诊法，是指观察病人的气色，听身体的声音，询问病状，把脉。我们对APP的某一个模块进行反编译分析的时候，也需要首先从外观上来看，找到其入口，因此我们也需要望闻问切，望闻问切的工具就是adbshell dumpsys，查询当前安卓系统中运行的各个APP的各个模块的状态，通过对运行状态的分析，我们可以找到小程序的切人点，然后再进行详细的分析。

​        首先我们将微信小程序的使用流程梳理如下（升级最新版本的微信）：

| **![img](/images/posts/wechat-mini-app/clip_image005.png)****** | **![img](/images/posts/wechat-mini-app/clip_image007.png)****** | **![img](/images/posts/wechat-mini-app/clip_image009.png)****** | **![img](/images/posts/wechat-mini-app/clip_image011.png)****** | **![img](/images/posts/wechat-mini-app/clip_image013.png)****** |
| ---------------------------------------- | ---------------------------------------- | ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| 图 3 小程序入口                                | 图4 小程序列表                                 | 图5 小程序-摩拜单车                              | 图6 小程序切换                                 | 图7 小程序后台清理                               |

​       

​        我们依次按照图6 打开多个小程序，然后使用adb shell来查看当前微信模块的运行状态：

```
adb shell dumpsys activity processes
```

​        执行结果经过筛选如下：

```java
*APP* UID 10109 ProcessRecord{6f6e009 3382:com.tencent.mm:appbrand0/u0a109, isShadow:false}
    Activities:
      - ActivityRecord{a08500d u0 com.tencent.mm/.plugin.appbrand.ui.AppBrandUI, isShadow:false t236}
    Connections:
      - ConnectionRecord{ce94e6 u0 CR com.tencent.mm/.plugin.appbrand.ipc.AppBrandMainProcessService:@edd8741}
      - ConnectionRecord{3b584d4 u0 CR com.tencent.mm/.plugin.appbrand.ipc.AppBrandMainProcessService:@edd8741}
      - ConnectionRecord{937aa27 u0 CR com.tencent.mm/.plugin.appbrand.ipc.AppBrandMainProcessService:@edd8741}
    Connected Providers:
      - b4f589c/com.android.providers.settings/.SettingsProvider->3382:com.tencent.mm:appbrand0/u0a109, isShadow:false s1/1 u0/0 +2m51s698ms
    Receivers:
, isShadow:false
*APP* UID 10109 ProcessRecord{980eb8a 3555:com.tencent.mm:appbrand1/u0a109, isShadow:false} 略...    
*APP* UID 10109 ProcessRecord{1dab708 3185:com.tencent.mm:appbrand2/u0a109, isShadow:false} 略... 
*APP* UID 10109 ProcessRecord{38abeb4 3726:com.tencent.mm:appbrand3/u0a109, isShadow:false} 略...
*APP* UID 10109 ProcessRecord{a55d54b 2928:com.tencent.mm:appbrand4/u0a109, isShadow:false} 略... 
       
```

​        大家注意每个段落开头的这一行:**APP\* UID 10109 ProcessRecord{a55d54b 2928:com.tencent.mm:appbrand[0-4]****，**微信给每个运行的小程序都分配了一个独立的进程，这样做有两个好处：一、由于微信小程序底层使用定制过的webview来展示，安卓系统中webview历来在内存上消耗比较大，独立进程之后，可以单独使用本进程的内存，关闭时直接杀掉此进程，不会造成内存泄漏。二、独立进程运行的小程序崩溃不会对微信造成任何影响，这也贴合了微信一贯简单可用的思想。

​        还有一个比较重要的一点就是每个小程序所在进程使用的系统组件比较一致，如下表：

| **组件类型******    | **组件实例******                             | **组件实例数目******         |
| --------------- | ---------------------------------------- | ---------------------- |
| Activity        | com.tencent.mm/.plugin.appbrand.ui.AppBrandUI【0-4】 | 1                      |
| Service(Binder) | com.tencent.mm/.plugin.appbrand.ipc.AppBrandMainProcessService | client conn:3 server:1 |
| ContentProvider | com.android.providers.settings/.SettingsProvider | 1                      |
| Receiver        | com.tencent.mm:appbrand0/10109/u0        | 若干                     |

表1 微信小程序进程组件分析

## 2.2 庖丁解牛

​        通过上表对小程序运行的进程进行分析之后，找到了小程序的界面入口点com.tencent.mm/.plugin.appbrand.ui.AppBrandUI，接下来我们先反编译其manifet组件声明部分如下

```java
    	<activity            android:name="com.tencent.mm.plugin.appbrand.ui.AppBrandLauncherUI"/>
        <activity            android:name="com.tencent.mm.plugin.appbrand.ui.AppBrandSearchUI"/>
        <activity            android:name="com.tencent.mm.plugin.appbrand.ipc.AppBrandProcessProxyUI"/>
        <activity            android:name="com.tencent.mm.plugin.appbrand.ui.AppBrandTBSDownloadProxyUI" .../>
        <activity            android:name="com.tencent.mm.plugin.appbrand.ui.AppBrand404PageUI"/>
		<service            android:name="com.tencent.mm.plugin.appbrand.ipc.AppBrandMainProcessService" />
        <activity
            android:name="com.tencent.mm.plugin.appbrand.ui.AppBrandUI"
            android:process=":appbrand0"
            android:taskAffinity=".AppBrandUI"
            android:launchMode="2"
            android:configChanges="0x4a0" />
        <activity
            android:name="com.tencent.mm.plugin.appbrand.ipc.AppBrandTaskProxyUI"
            android:process=":appbrand0"
            android:taskAffinity=".AppBrandUI"
            android:configChanges="0x4a0" />
        <receiver
            android:name="com.tencent.mm.plugin.appbrand.task.AppBrandTaskPreloadReceiver"
            android:exported="false"
            android:process=":appbrand0" />
        <activity
            android:name="com.tencent.mm.plugin.appbrand.ui.AppBrandUI1"
            android:process=":appbrand1"
            android:taskAffinity=".AppBrandUI1"
            android:launchMode="2"
            android:configChanges="0x4a0" />
        <activity
            android:name="com.tencent.mm.plugin.appbrand.ipc.AppBrandTaskProxyUI1"
            android:process=":appbrand1"
            android:taskAffinity=".AppBrandUI1"
            android:configChanges="0x4a0" />
        <receiver
            android:name="com.tencent.mm.plugin.appbrand.task.AppBrandTaskPreloadReceiver1"
            android:exported="false"
            android:process=":appbrand1" />
        AppBrandUI2，3.4略...   
           
```

 

​        通过对上述组件文件的分析可以将小程序的组件划分为两大类：公共基础组件和小程序运行组件，详细说明如下：

| **小程序公共组件**                              | **说明**        |
| ---------------------------------------- | ------------- |
| com.tencent.mm.plugin.appbrand.ui.AppBrandLauncherUI | 小程序入口         |
| com.tencent.mm.plugin.appbrand.ui.AppBrandSearchUI | 小程序搜索         |
| com.tencent.mm.plugin.appbrand.ipc.AppBrandProcessProxyUI | 启动独立进程小程序的过渡页 |
| com.tencent.mm.plugin.appbrand.ui.AppBrandTBSDownloadProxyUI | BSDiff增量下载    |
| com.tencent.mm.plugin.appbrand.ui.AppBrand404PageUI | 本地的404页面      |
| com.tencent.mm.plugin.appbrand.ipc.AppBrandMainProcessService | 小程序服务管理进程     |
| **每个小程序共有的组件**                           |               |
| com.tencent.mm.plugin.appbrand.ui.AppBrandUI | 小程序界面载体       |
| com.tencent.mm.plugin.appbrand.ipc.AppBrandTaskProxyUI | unknown       |
| com.tencent.mm.plugin.appbrand.task.AppBrandTaskPreloadReceiver | unknown       |

表2 小程序公共组件

​        小程序的多进程和多任务是通过Manifest中的process标签和taskAffinity实现的，每个小程序对应的进程标签android:process=":appbrand[0-4]"，任务标签android:taskAffinity=".AppBrandUI[0-4]"。因为在安卓系统中，普通的应用程序是没有办法直接fork一个新的进程，只能使用系统提供的标准进程，所以多进程使用组件进程标签Process声明，这样系统在启动该组件的时候，就会新建一个对应的进程。**同时这里也发现一个问题，由于微信是静态声明的进程标签属性，而且只有**0-4**，也就是说微信顶多只能同时运行**5个小程序。那么已经运行5个小程序之后，在打开第6个小程序会怎么办呢？经过测试分析，发现微信小程序会以管理内存LRU类似的算法来管理运行的小程序，当点击第6个的时候，会按照一定的权重删掉某一个（测试的时候观察到时按照启动的时间排序的）。

​        接下来是分析单个小程序运行之后的构成，因此我们对摩拜单车小程序进行界面组成分析，我们可以打开开发者设置中的显示布局边界选项，然后界面组成如下图：

![img](/images/posts/wechat-mini-app/clip_image015.jpg)

图 8 摩拜单车界面分析

​        从上图可知小程序主要UI界面渲染为一个整体的WebView,底部部分组件是Native，从微信的官方介绍也可以得知微信小程序是基于WebView+JSCore，部分Native是直接覆盖在WebView之上，其运行环境为Android- X5 JS解析器，使用VirtualDOM，进行局部更新。这里简单说明下X5内核是腾讯浏览器团队对外开放的浏览器SDK，用于取代系统默认的webview，以保障其性能、安全可靠、兼容性好等特点，具体特性以及接入方式可以参照[腾讯X5浏览器内核官网](http://x5.tencent.com/tbs/guide/sdkInit.html)。

* **小程序的Native输入组件以及键盘分析**

​        小程序中嵌入了很多Native的组件，用以改善用户体验，这里重点介绍一下小程序中的输入控件以及对应的键盘样式，如下图9：

![img](/images/posts/wechat-mini-app/clip_image017.jpg)

图 9 小程序键盘以及输入组件

​        h5中的input组件被封装为可调起NaitveEditText组件，这个是通过小程序的库来实现的，Native组件直接覆盖在webview的组件之上，所以整个input组件的使用流程如下：

​        1.input获取焦点的时候，通过jsBridge调起Native输入组件，在这个过程中需要传递input的坐标，宽度，键盘类型。

​        2.Naitve接收到调用请求之后，先根据坐标计算是否需要滑动webview，以弹起键盘。

​        3.在计算滑动之后的位置，附加EditText，同时获取焦点，并弹起自定义键盘。

​        4.用户输入的字符都显示在NativeView之上。

​        5.用户输入完成之后会点击其他区域（失去焦点）或者收起键盘，此时将输入框的内容回传给h5。

​       

​        小程序剩余的逻辑封装在X5内核中，通讯的JsBridge也是在该内核中，这里暂时先贴一小部分反编译的X5内核代码：

```java
final boolean hv(boolean z) {
        String e;
        try {
            e = be.e(this.kWC.getContext().getAssets().open("jsapi/wxjs.js"));
            if (this.lcw) {
                e = e.replace("isUseMd5_check", "yes").replace("xx_yy", this.lcx);
            }
        } catch (Throwable e2) {
            com.tencent.mm.sdk.platformtools.v.a("MicroMsg.JsLoader", e2, "", new Object[0]);
            e = null;
        }
        if (e == null) {
            com.tencent.mm.sdk.platformtools.v.e("MicroMsg.JsLoader", "loadJavaScript fail, jsContent is null");
            return false;
        } else if (this.kWC == null) {
            com.tencent.mm.sdk.platformtools.v.e("MicroMsg.JsLoader", "loadJavaScript, viewWV is null");
            return false;
        } else {
            this.kWC.evaluateJavascript("javascript:" + e, new C148251(this));
            if (this.kYe == null) {
                com.tencent.mm.sdk.platformtools.v.e("MicroMsg.JsLoader", "loadJavaScript, jspai is null");
                return false;
            }
            if (!this.lcT) {
                this.kWC.evaluateJavascript("javascript:WeixinJSBridge._isBridgeByIframe = false", null);
            }
            C16607f c16607f = this.kYe;
            com.tencent.mm.sdk.platformtools.v.v("MicroMsg.JsApiHandler", "jsapi init, preInit = %b", new Object[]{Boolean.valueOf(z)});
            if (z) {
                c16607f.lco.evaluateJavascript("javascript:WeixinJSBridge._handleMessageFromWeixin(" + C14852a.m22842a("sys:preInit", c16607f.lcs, c16607f.lcw, c16607f.lcx) + ")", null);
            } else {
                c16607f.lco.evaluateJavascript("javascript:WeixinJSBridge._handleMessageFromWeixin(" + C14852a.m22842a("sys:init", c16607f.lcs, c16607f.lcw, c16607f.lcx) + ")", null);
            }
            c16607f.lco.evaluateJavascript("javascript:WeixinJSBridge._handleMessageFromWeixin(" + C14852a.m22842a("sys:bridged", null, c16607f.lcw, c16607f.lcx) + ")", null);
            c16607f.lcu = true;
            c16607f.biP();
            if (!(be.kS(c16607f.lcz) || c16607f.lco == null)) {
                c16607f.lco.evaluateJavascript(c16607f.EJ(c16607f.lcz), null);
                c16607f.lcz = null;
            }
            com.tencent.mm.sdk.platformtools.v.i("MicroMsg.JsLoader", "jsapi init done");
            return true;
        }
    }

```

​        上述代码读取了本地的jsapi/wxjs.js文件，然后通过evaluateJavascript将微信通信协议对象WeixinJSBridge注入到当前页面，我们在分析下wxjs.js内部实现逻辑

```java 
//将消息添加到发送队列，iframe的准备队列为weixin://dispatch_message/
    function _sendMessage(message) {
        _sendMessageQueue.push(message);
    	if (_readyMessageIframe) {
             if(window.WeixinJSBridge._isBridgeByIframe){
                _readyMessageIframe.src = _CUSTOM_PROTOCOL_SCHEME + '://' + _QUEUE_HAS_MESSAGE;
             }else{
                console.log(_CUSTOM_PROTOCOL_SCHEME + '://' + _QUEUE_HAS_MESSAGE);
             }
        }
    	
        // var ifm = _WXJS('iframe#__WeixinJSBridgeIframe')[0];
        // if (!ifm) {
        //   ifm = _createQueueReadyIframe(document);
        // }
        // ifm.src = _CUSTOM_PROTOCOL_SCHEME + '://' + _QUEUE_HAS_MESSAGE;
    };

```

​       这里截取了改文件的核心通信部分代码，通过分析可知，微信是通过iframe的方式，加载一个自定义的微信Scheme协议_CUSTOM_PROTOCOL_SCHEME= 'weixin'，即weixin://message。然后通过console.log发送通信协议数据，然后再X5webview内核的onConsoleMessage中解析协议，并执行H5发起的请求，Native端解析的代码如下：

```java
public final boolean onConsoleMessage(ConsoleMessage consoleMessage) {
                String message = consoleMessage != null ? consoleMessage.message() : null;
                v.i("MicroMsg.WebViewUI", "onConsoleMessage : %s", new Object[]{message});
                if (be.kS(message) || this.kZH.kYA == null) {
                    return false;
                }
                if (message.equalsIgnoreCase("weixin://preInjectJSBridge/start")) {
                    this.kZH.kZa = true;
                    v.i("MicroMsg.WebViewUI", "now inject js library");
                    if (this.kZH.kYA != null) {
                        this.kZH.kYA.bja();
                    }
                    return true;
                } else if (message.equalsIgnoreCase("weixin://preInjectJSBridge/fail")) {
                    if (this.kZH.kZa) {
                        v.e("MicroMsg.WebViewUI", "preInjectJSBridge fail");
                        com.tencent.mm.plugin.report.service.g.iiu.a(156, 3, 1, false);
                    }
                    this.kZH.kZa = false;
                    return true;
                } else if (message.equalsIgnoreCase("weixin://preInjectJSBridge/ok")) {
                    v.d("MicroMsg.WebViewUI", "preInjectJSBridge ok");
                    return true;
                } else if (this.kZH.kYA.lcT) {
                    if (s.Ho(message).booleanValue()) {
                        v.i("MicroMsg.WebViewUI", "onConsoleMessage,set by console handle");
                        this.kZH.kYA.lcT = false;
                        this.kZH.kYc = false;
                        return true;
                    } else if (!s.Hn(message).booleanValue()) {
                        return false;
                    } else {
                        v.i("MicroMsg.WebViewUI", "onConsoleMessage preinject ,set by console handle");
                        this.kZH.kYA.lcT = false;
                        this.kZH.kYc = false;
                        this.kZH.kYA.lcU = true;
                        this.kZH.kYA.bja();
                        return true;
                    }
                } else if (!s.dn(message, "weixin://private/setresult/") && !s.dn(message, "weixin://dispatch_message/") && !s.dn(message, "weixin://gethtml/")) {
                    return false;
                } else {
                    if (message.equals(this.kZH.kZA)) {
                        this.kZH.kZA = "";
                        return true;
                    } else if (this.kZI.size() > 200) {
                        return true;
                    } else {
                        this.kZI.add(message);
                        if (!(this.kZH.handler == null || this.fBW)) {
                            this.kZH.handler.post(new C147252(this));
                        }
                        return true;
                    }
                }
            }
```

​        最终回调给H5结果是通过evaluateJavascript，具体代码可自行查阅。到这里微信小程序的整个流程已经比较清晰了，逆向分析到这里就结束了,下图9为微信团队分享的小程序总体结构图。    

![img](/images/posts/wechat-mini-app/clip_image018.png)

图 10 微信总体架构

# 2.3 小程序的思考

​        微信小程序、支付宝小程序、轻应用等容器，其核心包括：

1. 为提供一个稳定的jsBridge，以供H5和Native进行相互调用，在此基础上提供各种端基础能力和业务能力。
2. 提供订制化的UI组件以优化性能、改善用户体验。
3. 提供Hybrid离线化能力，优化加载时间。

 **当前hybird**的趋势是：**h5–>h5+jsBridge --> h5 +jsBridge +部分NativeUI组件-->ReactNative 全量Native组件化。**如果是第三方APP也希望能够具备这样的能力，可以参考如下方案：

​        1.Android内嵌的webview内存消耗问题，这个问题伴随着安卓系统版本的演进而存在，而且各个版本兼容性也不一致，而微信小程序则是直接使用腾讯X5浏览器内核，提供一致的兼容性保障以及动态升级能力，同时将整个小程序组件放在独立进程，关闭时直接kill进程，能够很好的解决内存占用问题。当然，这个改造过程比较大，很多端能力都没有提供跨进程调用服务，所以需要将所有端能力进行一次服务封装，封装为统一的跨进程调用，这样就能在独立进程中调起端能力。

​	2.订制化键盘、输入框等原生UI组件

​        3.对现有H5业务进行完整的Hybrid离线化，即：发版内置初始版本，后续可通过服务端增量更新，具体版本更新细则可以参照插件化增量更新机制，这样对于快速更新的业务有非常大的优势，可以在极短的时间内动态上线，而且可以做到多端统一，如下为参照某Hybrid的目录结构：

```
webapp //根目录
├─
├─business1 //业务1
│  │  index.html //业务入口html资源，如果不是单页应用会有多个入口
│  │  main.js //业务所有js资源打包
│  │
│  └─static //静态样式资源
│      ├─css 
│      ├─hybrid //存储业务定制化类Native Header图标
│      └─images
├─libs
│      libs.js //框架所有js资源打包
│
└─static //框架静态资源样式文件
    ├─css
    └─images
```

​          

# 2.4 总结

<<<<<<< 79d4588a46c1388b1ce27b346387542502ca1c5b
​       本文首先通过介绍了安卓APP的基本结构、以及对应的逆向分析方式，然后对微信的小程序模块进行了逆向分析，从微观角度对其进行阐释，总结其优点以及可以参照的地方，最后对当前热门小程序APP进行拓展思考总结，抛砖引玉。
=======
​        本文首先通过介绍了安卓APP的基本结构、以及对应的逆向分析方式，然后对微信的小程序模块进行了逆向分析，从微观角度对其进行阐释，总结其优点以及可以参照的地方，最后对当前热门小程序APP进行拓展思考总结，抛砖引玉。
>>>>>>> fix word error

