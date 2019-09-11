---
layout: post
title: Hybrid VS ReactNative
categories: Blog
description: Hybrid VS ReactNative
keywords:   Hybrid、ReactNative
---



# Hybrid VS ReactNative

 

**作者：[转载](http://blog.csdn.net/xiangzhihong8/article/details/52623852)**

本来准备写一篇的，考虑到现有很多很好的博客，而且自己研究的也不是很深入，转载一篇吧。

多说两句：
Facebook 貌似在开源协议上想动手脚，一下子把各大 IT 公司吓得不轻，都开始自己造轮子了。所以 Hybrid VS ReactNative 当前战况是：Hybrid first,先用 Hybrid 完成业务吧。后续造轮子静候佳音。


Facebook 于2015年9月15日推出react native for Android 版本， 加上2014年底已经开源的IOS版本，至此RN （react-native）真正成为跨平台的客户端框架。本篇主要是从分析代码入手，探讨一下RN在安卓平台上是如何构建一套JS的运行框架。

### 一、 整体架构

RN 这套框架让 JS开发者可以大部分使用JS代码就可以构建一个跨平台APP。 Facebook官方说法是learn once, run everywhere， 即在Android 、 IOS、 Browser各个平台，程序画UI和写逻辑的方式都大致相同。因为JS 可以动态加载，从而理论上可以做到write once, run everywhere， 当然要做额外的适配处理。如图：

[![_1](http://img3.tbcdn.cn/L1/461/1/04602301662556708a669ff6685f8978ce668b3d)](http://img3.tbcdn.cn/L1/461/1/04602301662556708a669ff6685f8978ce668b3d?spm=5176.100239.blogcont.6.nOeP5t)
RN需要一个JS的运行环境， 在IOS上直接使用内置的javascriptcore， 在Android 则使用webkit.org官方开源的jsc.so。 此外还集成了其他开源组件，如fresco图片组件，okhttp网络组件等。

RN 会把应用的JS代码（包括依赖的framework）编译成一个js文件（一般命名为index.android.bundle), , RN的整体框架目标就是为了解释运行这个js 脚本文件，如果是js 扩展的API， 则直接通过bridge调用native方法; 如果是UI界面， 则映射到virtual DOM这个虚拟的JS数据结构中，通过bridge 传递到native ， 然后根据数据属性设置各个对应的真实native的View。 bridge是一种JS 和 JAVA代码通信的机制， 用bridge函数传入对方module 和 method即可得到异步回调的结果。

对于JS开发者来说， 画UI只需要画到virtual DOM 中，不需要特别关心具体的平台, 还是原来的单线程开发，还是原来HTML 组装UI（JSX），还是原来的样式模型（部分兼容 )。RN的界面处理除了实现View 增删改查的接口之外，还自定义一套样式表达CSSLayout，这套CSSLayout也是跨平台实现。 RN 拥有画UI的跨平台能力，主要是加入Virtual DOM编程模型，该方法一方面可以照顾到JS开发者在html DOM的部分传承， 让JS 开发者可以用类似DOM编程模型就可以开发原生APP ， 另一方面则可以让Virtual DOM适配实现到各个平台，实现跨平台的能力，并且为未来增加更多的想象空间， 比如react-cavas, react-openGL。而实际上react-native也是从react-js演变而来。

对于 Android 开发者来说， RN是一个普通的安卓程序加上一堆事件响应， 事件来源主要是JS的命令。主要有二个线程，UI main thread, JS thread。 UI thread创建一个APP的事件循环后，就挂在looper等待事件 , 事件驱动各自的对象执行命令。 JS thread 运行的脚本相当于底层数据采集器， 不断上传数据，转化成UI 事件， 通过bridge转发到UI thread, 从而改变真实的View。 后面再深一层发现， UI main thread 跟 JS thread更像是CS 模型，JS thread更像服务端， UI main thread是客户端， UI main thread 不断询问JS thread并且请求数据，如果数据有变，则更新UI界面。

### 二、 代码流程

#### 1、JS入口

[![_2015_10_26_4_40_39](http://img2.tbcdn.cn/L1/461/1/23bbf9d71a527b7c15fbc6f01526b0ca0ce560e9)](http://img2.tbcdn.cn/L1/461/1/23bbf9d71a527b7c15fbc6f01526b0ca0ce560e9?spm=5176.100239.blogcont.7.nOeP5t)

对于JS开发者来说， 整个RN APP就只有一个JS文件， 而开发者需要编写的就只有如上部分。主要是四个部分：

- require 所有依赖到的组件， 相当于java中的import 或者 c++ 中的include。
- var AwesomeProject = React.createClass 创建APP， 并且在render函数中返回UI界面结构（采用JSX ), 实际经过编译， 都会变成JS 代码， 比如 变成 React.createElement(View,{style:{flex:1}},
- var styles = StyleSheet.create({， 创建CSS 样式，实际上会直接当做参数直接反馈到上面的React.createElement
- AppRegistry.registerComponent('AwesomeProject', () => AwesomeProject); 以上三个更像是参数，这个才是JS 程序的入口。即把当前APP的对象注册到AppRegistry组件中， AppRegistry组件是js module。

接着就等待Native事件驱动渲染JS端定义的APP组件。

#### 2、Native 入口

[![_2015_10_26_5_01_52](http://img1.tbcdn.cn/L1/461/1/3ab928a93376a5df39d67dec7ce07e6cbc0eabb5)](http://img1.tbcdn.cn/L1/461/1/3ab928a93376a5df39d67dec7ce07e6cbc0eabb5?spm=5176.100239.blogcont.8.nOeP5t)

对于Android 开发者， 普通安卓程序入口是Activity.onCreate()方法 ， 主要有三个对象

- ReactRootView, Android 标准的FrameLayout对象，另外一个功能是提供react 世界的入口，函数startReactApplication实际调用attachMeasuredRootView触发react世界的初始化。
- MyReactPackage， 配置当前APP 需要加载的模块，RN 的JS框架会在初始化阶段就会把native的模块按照配置加载到JS数据结构中（MessageQueue), 从而才能在JS 层即可直接判断native是否支持某个模块。支持三种类型模块配置， native module(实际就是不需要操作View结构的API), view managers(实际是映射到virtual DOM中的View组件)， JS module 。
- ReactInstanceManager， 构建React世界的运行环境，发送事件到JS世界， 驱动整个React世界运转。 通过builder可以创建不同的React环境， 比如内置js 路径， 开发环境dev的js名字，是否支持调试等。doInBackground会加载指定的JS文件, onPostExecute会调用runApplication接口运行JS APP。
  [![_2015_10_27_8_02_04](http://img4.tbcdn.cn/L1/461/1/c3e9e520a8d98f4b61b7a2bfe4b39dd787b7c008)](http://img4.tbcdn.cn/L1/461/1/c3e9e520a8d98f4b61b7a2bfe4b39dd787b7c008?spm=5176.100239.blogcont.9.nOeP5t)

ReactRootView第一次onMeasured计算完成， 然后会利用ReactInstanceManager创建 ReactContext上下文环境。重要的是初始化bridge以及加载js文件， 利用JSBundleLoader方法加载index.android.bundle. 如图

[![_2015_10_27_1_41_19](http://img4.tbcdn.cn/L1/461/1/7c594f647aa51a0080435689022b84613ae82657)](http://img4.tbcdn.cn/L1/461/1/7c594f647aa51a0080435689022b84613ae82657?spm=5176.100239.blogcont.10.nOeP5t)

此刻进入JS 世界, 开发者的js 语句连同react js框架层被执行。该步骤最终语句是执行AppRegistry.registerComponent注册一个APP组件，但还没有到开始渲染。

当运行环境准备完毕， 则调用bridge方法运行上步注册的APP组件，触发一连串JS 和 Native相互通信，配合事件驱动， 从而完成native世界的渲染。如图利用bridge方法运行上面注册的JS APP组件的runApplication方法： 
[![_2015_10_27_1_37_35](http://img2.tbcdn.cn/L1/461/1/c136ec2fbc26629ac7bfcfa717fa830d5adec0e9)](http://img2.tbcdn.cn/L1/461/1/c136ec2fbc26629ac7bfcfa717fa830d5adec0e9?spm=5176.100239.blogcont.11.nOeP5t)

#### 3、事件循环

所有的APP在操作系统中， 最终都会使用一个事件循环来运行。

一般来说，JS 开发者只需要开发各个组件对象，监听组件事件， 然后利用framework接口调用render方法渲染组件。

而实际上，JS 也是单线程事件循环，不管是 API调用， virtural DOM同步， 还是系统事件监听， 都是异步事件，采用Observer（观察者）模式监听JAVA层事件， JAVA层会把JS 关心的事件通过bridge直接使用javascriptCore的接口执行固定的脚本， 比如"requrire (test_module).test_methode(test_args)"。此时，UI main thread相当于work thread, 把系统事件或者用户事件往JS层抛，同时，JS 层也不断调用模块API或者UI组件 ， 驱动JAVA层完成实际的View渲染。JS开发者只需要监听JS层framework定义的事件即可。如图即JS thread 的消息队列循环：

[![_2015_10_27_10_13_36](http://img4.tbcdn.cn/L1/461/1/1d667a068698fb723f7d838b20ded8d28d29fc09)](http://img4.tbcdn.cn/L1/461/1/1d667a068698fb723f7d838b20ded8d28d29fc09?spm=5176.100239.blogcont.12.nOeP5t)

分析代码可知，消息线程创建于ReactContext环境初始化时， MessageQueueThread.java当中， 该消息队列主要接收系统事件（如 Vsync、timer、doFrame、backkey）、UI事件（如键盘弹起、滚动等）以及 callback事件(JS 的回调函数)。
如图即ReactRootView往JS 传递键盘弹出的事件：

[![_2015_10_27_11_20_27](http://img2.tbcdn.cn/L1/461/1/6aa974701ca9fa045102e708a0201ecdbeac01e2)](http://img2.tbcdn.cn/L1/461/1/6aa974701ca9fa045102e708a0201ecdbeac01e2?spm=5176.100239.blogcont.13.nOeP5t)

而对于Android 开发者， Android 已经为APP创建一个默认的 Main Looper， 不管是Android System 还是JS 事件都是发送到Main thread通过UI渲染出来。如图即是MessageQueueThread.java直接使用主线程Looper。

[![_2015_10_27_11_08_11](http://img3.tbcdn.cn/L1/461/1/38939e065f76cee9815dea24237ee4040cd4931c)](http://img3.tbcdn.cn/L1/461/1/38939e065f76cee9815dea24237ee4040cd4931c?spm=5176.100239.blogcont.14.nOeP5t)

跟普通APP不同是，此时JS thread相当于work thread， JS会把对应的事件或者数据通过bridge发送到UI thread。 如图即是native Java层收到的JS事件的处理函数：
[![_2015_10_27_1_47_12](http://img4.tbcdn.cn/L1/461/1/c7ae031da1a4ebd43225f63a59c0437345f1bb70)](http://img4.tbcdn.cn/L1/461/1/c7ae031da1a4ebd43225f63a59c0437345f1bb70?spm=5176.100239.blogcont.15.nOeP5t)

### 三、 通信机制

RN框架最主要的就是实现了一套JAVA和 JS通信的方案，该方案可以做到比较简便的互调对方的接口。一般的JS运行环境是直接扩展JS接口，然后JS通过扩展接口发送信息到主线程。但RN的通信的实现机制是单向调用，Native线程定期向JS线程拉取数据， 然后转成JS的调用预期，最后转交给Native对应的调用模块。这样最终同样也可以达到Java和 JS 定义的Module互相调用的目的。

#### 1、JS调用java

JS调用java 使用通过扩展模块require('NativeModules')获取native模块，然后直接调用native公开的方法，比如require('NativeModules').UIManager.manageChildren()。 JS 调用require('NativeModules')实际上是获取MessageQueue里面的一个native模块列表的属性, 如：
[![_2015_10_27_4_03_35](http://img1.tbcdn.cn/L1/461/1/088ced5ec9cdc54f869075ccb8fe67e46985c484)](http://img1.tbcdn.cn/L1/461/1/088ced5ec9cdc54f869075ccb8fe67e46985c484?spm=5176.100239.blogcont.16.nOeP5t)

[![_2015_10_27_2_00_49](http://img4.tbcdn.cn/L1/461/1/cd563e3f4489f8fa31a75f17f947333c17aa55c3)](http://img4.tbcdn.cn/L1/461/1/cd563e3f4489f8fa31a75f17f947333c17aa55c3?spm=5176.100239.blogcont.17.nOeP5t)

使用_genModules 加载所有native module到 RemoteModules数组。RemoteModules每项都是一个映射到native module的JS对象。

[![_2015_10_27_4_08_02](http://img1.tbcdn.cn/L1/461/1/7e18b499a98e384cc9f7b24b982bdfd3436e8ad1)](http://img1.tbcdn.cn/L1/461/1/7e18b499a98e384cc9f7b24b982bdfd3436e8ad1?spm=5176.100239.blogcont.18.nOeP5t)

调用RemoteModules 的方法， 实际是把moduleID、methodId、args放入三个queue保存。

[![_2015_10_27_4_10_34](http://img4.tbcdn.cn/L1/461/1/cd3fd4786a0e8f6a06d154bc316108faae7340f3)](http://img4.tbcdn.cn/L1/461/1/cd3fd4786a0e8f6a06d154bc316108faae7340f3?spm=5176.100239.blogcont.19.nOeP5t)

至此， JS端调用完毕， queue中数据要等待Native层通过bridge来取。

native层会在一定条件下触发事件， 通过bridge调用callFunctionReturnFlushedQueue
和 invokeCallbackAndReturnFlushedQueue ，得到的返回值就是这三个queue。

[![_2015_10_27_4_39_12](http://img4.tbcdn.cn/L1/461/1/4c233eddcad914dd3ed85e09f25be9be95c80e13)](http://img4.tbcdn.cn/L1/461/1/4c233eddcad914dd3ed85e09f25be9be95c80e13?spm=5176.100239.blogcont.20.nOeP5t)

bridge会把这三个queue交给parseMethodCalls解析， 然后通过JNI回调函数转发到Java层
[![_2015_10_27_4_42_17](http://img4.tbcdn.cn/L1/461/1/53def2b0168f1256ffa684644484eaa7b72f14b5)](http://img4.tbcdn.cn/L1/461/1/53def2b0168f1256ffa684644484eaa7b72f14b5?spm=5176.100239.blogcont.21.nOeP5t)

m_callback 函数是在bridge初始化的时候设置到c++层, 如：
[![_2015_10_27_4_45_02](http://img3.tbcdn.cn/L1/461/1/cbab5b561f4da3e4efdd64f616d707bad2730f6e)](http://img3.tbcdn.cn/L1/461/1/cbab5b561f4da3e4efdd64f616d707bad2730f6e?spm=5176.100239.blogcont.22.nOeP5t)

然后在回调函数中，陆续调用ReactCallback对象的call方法，weakCallback就是java层初始化bridge时传入的NativeModulesReactCallback对象，也就是ReactCallback的子类。

[![_2015_10_27_4_53_41](http://img2.tbcdn.cn/L1/461/1/ba6de61200d5114f967ebf0197af14c3e923af61)](http://img2.tbcdn.cn/L1/461/1/ba6de61200d5114f967ebf0197af14c3e923af61?spm=5176.100239.blogcont.23.nOeP5t)

到此，转入Java层. 从native module配置表中，取到对应module和method，并执行。
[![_2015_10_27_4_58_22](http://img4.tbcdn.cn/L1/461/1/9fec1799cf6c6e05cb6a60c5395dea1d572012ed)](http://img4.tbcdn.cn/L1/461/1/9fec1799cf6c6e05cb6a60c5395dea1d572012ed?spm=5176.100239.blogcont.24.nOeP5t)

#### 2、java调用JS

之前ReactInstanceManager 中运行JS APP组件，JAVA 是调用catalystInstance.getJSModule 方法获取JS 对象，然后直接访问对象方法runApplication。实际上getJSModule 返回的是js对象在java层的映射对象。

java层可以调用的JS模块主要在CoreModulesPackage.createJSModules方法配置，有：

[![_2015_10_27_4_23_46](http://img2.tbcdn.cn/L1/461/1/18dce11fe3eb5d62475c522bc3700ec6ba1fca80)](http://img2.tbcdn.cn/L1/461/1/18dce11fe3eb5d62475c522bc3700ec6ba1fca80?spm=5176.100239.blogcont.25.nOeP5t)

如果调用JSModules对象的方法，则会动态代理跳转到(mBridge).callFunction(moduleId, methodId, arguments);

[![_2015_10_27_4_27_15](http://img2.tbcdn.cn/L1/461/1/54bf7411607b6da8da9c5a1b0485ece5bc5f61d1)](http://img2.tbcdn.cn/L1/461/1/54bf7411607b6da8da9c5a1b0485ece5bc5f61d1?spm=5176.100239.blogcont.26.nOeP5t)

接着调用ReactBridge中声明的JNI 函数，
public native void callFunction(int moduleId, int methodId, NativeArray arguments);

[![_2015_10_27_4_31_32](http://img4.tbcdn.cn/L1/461/1/b03f9b5b5dbe64be49ebe11f65002470170bc2bd)](http://img4.tbcdn.cn/L1/461/1/b03f9b5b5dbe64be49ebe11f65002470170bc2bd?spm=5176.100239.blogcont.27.nOeP5t)

[![_2015_10_27_5_01_42](http://img3.tbcdn.cn/L1/461/1/03022fded732c7b439b0c055bbdd411d706993a8)](http://img3.tbcdn.cn/L1/461/1/03022fded732c7b439b0c055bbdd411d706993a8?spm=5176.100239.blogcont.28.nOeP5t)

通过JS 的require和 apply函数拼接一段JS 代码， 然后用javascriptCore的脚本运行接口执行，并得到返回值。

[![_2015_10_27_5_03_09](http://img2.tbcdn.cn/L1/461/1/19e44379ffb7fdff0473b5fe8cbb8b4ba096d106)](http://img2.tbcdn.cn/L1/461/1/19e44379ffb7fdff0473b5fe8cbb8b4ba096d106?spm=5176.100239.blogcont.29.nOeP5t)

这样就在JS引擎中运行了一段JS代码并得到返回值，实现了JAVA层到JS层的调用。每次有JAVA对JS的访问， 则在返回值中从JS层的messageQueue.js中抓取之前累积的一堆JS calls。因为JAVA层要把时间同步、 系统帧绘制等事件传递给JS， 因此queue中的JS calls都会在很短的时间内被抓取。

### 四、 扩展机制

1、 模块扩展(native module)
官方文档操作：
[https://facebook.github.io/react-native/docs/native-modules-android.html#content](https://facebook.github.io/react-native/docs/native-modules-android.html?spm=5176.100239.blogcont.30.nOeP5t#content)

2、 组件扩展（UI component）
官方文档操作：
[https://facebook.github.io/react-native/docs/native-components-android.html#content](https://facebook.github.io/react-native/docs/native-components-android.html?spm=5176.100239.blogcont.31.nOeP5t#content)

因为react模块加载主要在ReactPackage类配置，因此扩展可以通过反射、外部依赖注入等机制，可以做到跟H5容器一样实现动态插拔的插件式扩展。比如API扩展， 通过外部传入扩展模块的类名即可反射构造函数创建新的API：

```
    @Override
    public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
        List<NativeModule> modules = new ArrayList();
        modules.addAll(Arrays.<NativeModule>asList(
                new AsyncStorageModule(reactContext),
                new FrescoModule(reactContext),
                new NetworkingModule(reactContext),
                new WebSocketModule(reactContext),
                new ToastModule(reactContext)));
        if (mModuleList != null && mModuleList.size() > 0) {
            for (int i = 0; i < mModuleList.size(); i++) {
                try {
                    Log.i("MyReactPackage", "add Module:" + mModuleList.get(i));
                    Class c = Class.forName(mModuleList.get(i));
                    Class[] parameterTypes = {ReactApplicationContext.class};
                    java.lang.reflect.Constructor constructor = c.getConstructor(parameterTypes);
                    Object[] parameters = {reactContext};
                    NativeModule module = (NativeModule) constructor.newInstance(parameters);
                    modules.add(module);
                }catch (Exception e) {
                    Log.i("MyReactPackage", "add Module Exeception:" + e);
                    e.printStackTrace();
                }
            }
        }
        return modules;
    }

```

### 五、 离线加载

#### 代码离线

- 离线包支持。 目前RN官方支持内置APK打包以及dev server在线更新。而实际上，一般的容器都会实现一套离线包发布平台。大致的实现方案是自定义一个JSBundleLoader，对接到应用管理发布平台。
  [![_2015_10_27_6_02_57](http://img2.tbcdn.cn/L1/461/1/d3845c7b892c37b661cd29976208b74252960bfc)](http://img2.tbcdn.cn/L1/461/1/d3845c7b892c37b661cd29976208b74252960bfc?spm=5176.100239.blogcont.32.nOeP5t)
- 分离react 框架代码和应用业务代码。目前官方的生产工具是把框架代码和业务代码弄成一个bundle。 但框架代码很大，需要共用， 因此要分离出框架代码单独前置加载。 应用业务代码变成很小一段JS代码单独发布。如果每次都加载框架代码， 启动业务代码会比较慢，一个helloworld都需要4秒左右。初步实践方案是把ReactInstanceManager设置成全局变量共享，在Native APP 启动初始化或者第一次进入RN APP时初始化ReactInstanceManager。这个可能会导致多个RN APP全局变量冲突。
- 在线更新
  离线包更新主要依赖应用管理发布平台，大致可以做到跟H5离线包一致。

#### 资源离线

一般说的是图片资源比较多， RN 使用控件显示图片，如：
[![_2015_10_27_5_43_26](http://img3.tbcdn.cn/L1/461/1/d5349c523fc4c025299f8c01c4a56bd1b9432053)](http://img3.tbcdn.cn/L1/461/1/d5349c523fc4c025299f8c01c4a56bd1b9432053?spm=5176.100239.blogcont.33.nOeP5t)

通过source属性设置图片资源路径， 映射到native层：
[![_2015_10_27_5_46_33](http://img2.tbcdn.cn/L1/461/1/cbabf45903b94408359cab339bb4c11fd6e54705)](http://img2.tbcdn.cn/L1/461/1/cbabf45903b94408359cab339bb4c11fd6e54705?spm=5176.100239.blogcont.34.nOeP5t)

[![_2015_10_27_5_46_52](http://img1.tbcdn.cn/L1/461/1/b0241b605f2e6bea8b14cd5125fe859d801679b8)](http://img1.tbcdn.cn/L1/461/1/b0241b605f2e6bea8b14cd5125fe859d801679b8?spm=5176.100239.blogcont.35.nOeP5t)

因此不管是离线包内资源还是系统资源，只要能转换成Android 统一资源定位URI对象，即可获取到图片。

#### 在线资源

如果是静态资源，则直接URI统一定位。如果是动态资源， 比如要通过网关获取到base64格式的图片，则需要native扩展特别接口。

### 六、 总结

1、 可能瓶颈

```
*   因为bridge,  JS和 JAVA是异步互通，如果实现复杂多API的逻辑，可能会导致部分效率损耗在多线程通信。JS 异步的编程方式多多少少带来一些不便。
*  因为bridge,  可能某些场景做不到及时响应。比如帧动画的实时控制。
*  Android版本刚推出不完善，并且目前RN版本还在不停的更新中， 可能存在暗坑。
*  加入JS引擎， 内存的控制比较麻烦，会比普通native增加不少。

```

2、 待研究

- 动态注入的API插件实现方案，能跟h5容器共用实现。
- 因为RN已经具备很多的灵活， JS也可以做到很多大型控件，所以native UI扩展需要定义JS 和 native边界， 哪些是JS 实现， 哪些是native实现。
- 动画的实现方式。
- H5容器和RN容器融合方案
- write once, 完全跨平台。
- JS 层支持 Fragment manager

#### 性能比较数据

- Demo还在实现当中，等抓完再补充。