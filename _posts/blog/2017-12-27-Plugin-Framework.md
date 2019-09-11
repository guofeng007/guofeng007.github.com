---
layout: post
title: Android 插件原理解析
categories: Blog
description:  Android 插件原理解析
keywords:   Android,插件，原理，解析

---



# Android 插件原理解析

 

**作者：国风**


# 一、背景



  作为 B/S 架构的移动端 APP，从其诞生开始，便一直面临着发版更新问题。每次有新业务上线、或者 bug 修复，都必须重新打包，上线。整个流程非常耗时，对于业务高速迭代的产品而言，简直就是个灾难。有没有一种方式可以动态的下载业务模块呢？插件化就是解决这一问题的最佳方案。

引用主席的一句话"完成一个插件 demo 很简单，但是真正完整一整套插件平台并持续维护很难"（差不多是这么说的）

# 二、插件化简介

### 2.1 传统Java时代的插件化

![JavaPlugin](/images/posts/plugin/JavaPlugin.png)

 从服务端下载一个 jar，然后构造一个对应路径的 ClassLoader，在直接调用 main 方法，或者反射其他入口方法即可。 Jar 直接运行在 JVM 上，直接和 JVM 交互。

### 2.2 Android 插件化

![AndroidPlugin-java-way](/images/posts/plugin/AndroidPlugin-java-way.png)

Android 能不能也直接参考这种方式来干呢？答案是否定的！![error](/images/posts/plugin/error.png)

为什么不能直接从服务端加载一个 dex 呢？因为 Android App 的运行环境变了，Android App 运行在 JVM(Dalvik or Art)+AndroidFramework。一个 APP 是找不到 Main 方法的入口（这些都在 Framework 中），App 入口成了某一个 Activity。

![AndroidComponent](/images/posts/plugin/AndroidComponent.png)

如上图，整个 APP 由 Framework 提供的组件拼装而成。

这也给我们的插件化提供了实现标注：完成 Framework 对 APP 组件的控制，即可实现插件化。

接下来我们在从整体上看一下 AndroidFramework ：

![AndroidFramework](/images/posts/plugin/AndroidFramework.png)

各层标注如上，Android APP直接面对的是 Android API。所以插件框架必须在应用层模拟一套 Android API,保证插件运行环境。

# 三、当前热门开源插件框架

- 2012年 AndroidDynamicLoader，用Fragment作为插件化的载体，发现资源访问的方法

- 2013年 23Code:动态下载自定义控件（View级别）。

- 2014年 阿里 Altas:Hook机制，内部原理未公开。

- 2014年 OpenAltas/ACCD: 参照阿里Altas hook机制，同时利用aapt扩展资源分区。

- 2014年 DyanmicLoadApk 人玉刚，静态代理实现插件化

- 2015年 DroidPlugin,360,Hook 始祖，最强悍的 Hook 插件框架。

- 2015年手机助手GPT插件框架:（AOP静态织入、支持独立进程）。

- 2015年 基于Hook机制的AndFix （类似iOS JSPatch）, Nuwa 等hotfix框架。

- 2015年 Small:资源/Class与Host一体，编译后资源ID重排、携程DynamicApk:编译前资源重排。

- 2016年VirtualApp（DirectLoadApk），Lody，宁波中学，支持双开，多开。

- 2017年 滴滴 VirtuakAPK 任玉刚；360 Replugin;美团

  **有重大突破的插件框架**

  | 时间    | 框架名称                 | 核心思想                  | 主要贡献                  |
  | ----- | -------------------- | --------------------- | --------------------- |
  | 2012  | AndroidDynamicLoader | 用Fragment作为插件化的载体     | 资源访问                  |
  | 2014  | DynamicLoadApk       | 通过Activity静态代理方式实现插件化 | 静态代理                  |
  | 2015  | DroidPlugin          | 黑科技Hook+动态代理+占坑       | Hook+动态代理             |
  | 2017年 | Replugin             | Classloader 模式        | MultiDex方案减少 Hook 兼容性 |

  # 四、基于 Hook 框架的插件原理

  基于 Hook 的插件核心原理分为三个方面，如下图：

  ​

  ![PluginCore](/images/posts/plugin/PluginCore.png)

## 4.1 进程共享
插件寄生于 Host 的进程中，pid,uid 都复用 Host,以保证能够正常的运行，并且能够绕过 Android System Server各种检测。

![ProcessShare](/images/posts/plugin/ProcessShare.png)

## 4.2 Hook 欺骗
 
 ### 4.2.1 Hook 欺骗步骤
 如下图，哪些点是可以 Hook 的呢？----》 静态成员，单例（或者其他你可以访问的对象）
 Hook 的技术手段是动态代理，如果不太熟悉，需要恶补一下这方面的知识。
![Hook](/images/posts/plugin/Hook.png)	

下图为 Activity 启动流程中可以 Hook拦截的点，掌握了这些点，就能够偷天换日，替换插件中的 Intent,绕过系统检测。

![HookExample](/images/posts/plugin/HookExample.png)
在 Hook 的帮助下，各个插件其乐融融的运行。

![HookCommunicate](/images/posts/plugin/HookCommunicate.png)



## 4.3 代理占坑：
由于 Android 系统的各大组件都是直接在已经安装的 APP manifest 中注册的，系统重启或者安装完成会搜集这些组件。我们的插件中的四大组件是没办法被系统感知的，如何绕过这一层呢？
解决方案就是代理占坑，利用4.2的 Hook 技术，将插件中的组件替换为 Stub,在系统回调的时候在 替换为插件中的组件。
![ProxyStub](/images/posts/plugin/ProxyStub.png)

## 4.4 其他方面的设计：

### 4.4.1 数据隔离：
我们每个 APP 运行在自己的私有目录，插件也应该有自己的目录，所以我们需要对插件根据包名做一套类似系统的管理层。
![PluginDataSeperate](/images/posts/plugin/PluginDataSeperate.png)

### 4.4.2 插件安全校验：
插件加载之后，便可以直接运行在 Android 系统中，如果一不小心加载了恶意插件，后果将是很可怕的，所以在现在安装差简单 时候一定要通过 Https 加密通道，下发插件地址，md5,签名，并且这些信息也要在加密一次。
在运行之前仍然要检测一次，防止在本地被 root 后替换。
![Security](/images/posts/plugin/Security.png)



### 4.4.3 HostInvoker 互通调用：
插件可能需要和其他插件通信，所以必须制定一个插件规范，保证各个模块能够被成功调用，Invoker 便应运而生。
![APILevel](/images/posts/plugin/APILevel.png)

# 五、主流插件框架性能对比
- 插件性能对比

| \       | AndroidDyanmicLoader | DynamicAPK | DroidPlugin | VirtualApk |
| ------- | -------------------- | ---------- | ----------- | ---------- |
| 加载非独立插件 | x                    | √          | ×           | √          |
| 加载独立插件  | x                    | ×          | √           | √          |
| 四大组件    | x                    | √          | √           | √          |
| 资源分包共享  | ×                    | x          | x           | √          |
| 公共代码库共享 | ×                    | ×          | ×           | √          |
| 插件和宿主互通 | x                    | ×          | √           | √          |



- 集成透明度对比

| \            | AndroidDyanmicLoader | DynamicAPK | DroidPlugin | VirtualApk |
| ------------ | -------------------- | ---------- | ----------- | ---------- |
| 插件代码无需修改     | ×                    | ×          | √           | √          |
| 插件引用外部资源无需修改 | ×                    | ×          | ×           | √          |

# 六、总结
插件框架火于2015年，稳定于2017年，未来几乎也不会有太大的变化。我们应该珍惜现在的好时光，利用开源插件框架，巩固自己对 Android Frame 的认知。最后还是那句话：完成一个 Demo 很简单，能够搭建一整套插件平台以及持续的维护很难。