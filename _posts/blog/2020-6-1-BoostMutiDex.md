---
layout: post
title: BoostMutiDex学习心得
categories: [Blog,Cat]
description: BoostMutiDex学习心得
keywords: BoostMutiDex
---

儿童节快乐，早上学习了下抖音出品的BoostMutiDex，分享下心得体会。（没保存就提交了，重写一遍心得）



要想在深度上有所作为必须掌握



# 一、JNI Hook

1. dlopen*打开一个动态链接库 

2. dlsym根据动态链接库操作句柄与符号，返回符号对应的地址。dlsym根据动态链接库操作句柄(handle)与符号(symbol)，返回符号对应的地址。使用这个函数不但可以获取函数地址，也可以获取变量地址。handle是由[dlopen](http://baike.baidu.com/view/2907309.htm)打开[动态链接库](http://baike.baidu.com/view/887.htm)后返回的[指针](http://baike.baidu.com/view/159417.htm)，symbol就是要求获取的函数或[全局变量](http://baike.baidu.com/view/261041.htm)的名称。



3. dlclose用于关闭指定句柄的动态链接库，只有当此动态链接库的使用计数为0时,才会真正被系统卸载。

# 二、Android Framework

特别是dex，高中天才Lody很久直接就分享了一个Dex秒加载的Boost方案，核心本质是在启动的时候直接加载原始Dex，跳过dexopt。有点显而易见，缺点就是在运行性能不高。



抖音的BoostMultiDex在Lody第一个方案的基础上，解决了第二个问题，后台做dexopt。

思路很简单，但是工程化落地很复杂，涉及到各个版本JNI兼容，hook。当然还需要一定体量的APP来监控验证。在业界内我想头条、抖音的产品应该是有足够的话语权来验证这个问题了吧。。



原文地址

https://mp.weixin.qq.com/s/0gtkc7IQdhKjFxrHCuNZwQ

github

https://github.com/bytedance/BoostMultiDex 

