---
layout: post
title: BoostMutiDex心得
categories: [Blog,Cat]
description: 完整看了一下BoostMutiDex的文章和源码，写下心得体会
keywords: BoostMutiDex，MultiDex
---

儿童节快乐，一大早想起来BoostMultiDex还没有学习，趁着上午精神好，学习了一下，心得体会如下：



# 一、.JNI hook

要想做些黑操作，JNI必须要熟悉，特别是JNI层的hook：

1. dlopen 打开一个动态链接库

2. dlsym 根据动态链接库操作句柄与符号，返回符号对应的地址。dlsym根据动态链接库操作句柄(handle)与符号(symbol)，返回符号对应的地址。使用这个函数不但可以获取函数地址，也可以获取变量地址。handle是由[dlopen](http://baike.baidu.com/view/2907309.htm)打开[动态链接库](http://baike.baidu.com/view/887.htm)后返回的[指针](http://baike.baidu.com/view/159417.htm)，symbol就是要求获取的函数或[全局变量](http://baike.baidu.com/view/261041.htm)的名称。

3. dlclose用于关闭指定句柄的动态链接库，只有当此动态链接库的使用计数为0时,才会真正被系统卸载。

   

   

# 二、Android Dex

Android <=4.4 dalvik虚拟机只能加载单个dex，方法65535，超过之后，需要使用MultiDex，来操作pathlist，但是MultiDex阻塞加载，时间很长，很容易ANR。



Lody很早就有个一Boost方案，秒加载Dex，本质就是直接加载dex，不做dexopt。这个在启动的时候很有用，但是运行效率不高。



抖音的BoostMultiDex借鉴了这个方案，首次启动直接加载dex，同时在后台dexopt。

能够在启动和运行时同时获得较好的均衡。

抖音BoostMultiDex地址

https://mp.weixin.qq.com/s/0gtkc7IQdhKjFxrHCuNZwQ

https://github.com/bytedance/BoostMultiDex