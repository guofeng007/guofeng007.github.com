---
layout: page
title: About
description: 打码改变世界
keywords: 代码, 江山
comments: true
menu: 关于
permalink: /about/
---

我是国风，码而生，码而立。

## 简历
[点我查看简历](https://guofeng007.github.io/anires/public/)

------------------------------------------------
- 本尊：国风
- 公司：百度2015-2018
- 职业：移动端负责人

------------------------------------------------

仰慕「优雅编码的艺术」。

坚信熟能生巧，努力改变人生。

## 阅读过的书籍

Android:《Android系统源代码情景分析》、《Android开发艺术探索》、《Android源码设计模式解析与实战》
设计模式:《Head First 设计模式》、《设计模式之禅》
著名:《浪潮之巅》、《数学之美》、《编程之美》
面试:《剑指offer》、《编程之美》
机器学习:《机器学习》、《人工智能》
JVM:《手写JVM》、《JVM源码解析》
源码:DroidPlugin、Android Source Code、HotFix


## 联系
tianfengjingjing@gmail.com
{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## Skill Keywords

## Android
### 1. 插件化
- [DroidPlugin-360](https://github.com/DroidPluginTeam/DroidPlugin)
- [VirtualApp-Lody(参与)](https://github.com/asLody/VirtualApp)
- [VirtualAPK(滴滴)](https://github.com/didi/VirtualAPK)
- [DL(任玉刚)](https://github.com/singwhatiwanna/dynamic-load-apk)

***插件化两大流派：***
参照 [Kugoo](https://juejin.im/entry/59cde341f265da065476f21a?utm_source=gold_browser_extension)
**业务容器-兼容派（DiDi VirtualAPK,酷狗插件化）**

1. 基本原理

合并宿主和插件的ClassLoader 需要注意的是，插件中的类不可以和宿主重复
合并插件和宿主的资源 重设插件资源的packageId，将插件资源和宿主资源合并
去除插件包对宿主的引用 构建时通过Gradle插件去除插件对宿主的代码以及资源的引用

2. 四大组件的实现原理

Activity 采用宿主manifest中占坑的方式来绕过系统校验，然后再加载真正的activity；
Service 动态代理AMS，拦截service相关的请求，将其中转给Service Runtime去处理，Service Runtime会接管系统的所有操作；
Receiver 将插件中静态注册的receiver重新注册一遍；
ContentProvider 动态代理IContentProvider，拦截provider相关的请求，将其中转给Provider Runtime去处理，Provider Runtime会接管系统的所有操作。

**完全插件化-极客派**

1. 基本原理

Hook各种binder,AMS,PMS
动态代理
宿主进程共享

2. 四大组件的实现原理

Activity 占坑
Service 静态转发
Receiver 动态注册
ContentProvider 进程内注册

### 2. Hybrid
- 糯米Hybrid
- [JsBridge](https://github.com/lzyzsd/JsBridge)

### 3. Router
[ARouter,GenericModuleRouter](https://github.com/guofeng007/GenericModuleRouter/)

### 4.热修复
[HotFix （QZone方式）](https://github.com/dodola/HotFix/),
[Tinker](https://github.com/Tencent/tinker/)
[Robust](https://github.com/Meituan-Dianping/Robust)

### 5.ReactNative
[Facebook-ReactNative](http://facebook.github.io/react-native/)

## 机器学习TensorFlow
[CNN,DNN,RNN,LSTM](http://tensorflow.org/)




{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
