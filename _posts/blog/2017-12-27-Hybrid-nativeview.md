---
layout: post
title: Hybrid 原生控件体验优化
categories: Blog
description: Hybrid 原生输入控件等，优化用户体验
keywords:   Hybrid,用户体验，原生控件

---



# Hybrid 原生控件体验优化

 

**作者：国风**


# 一、背景
h5在移动时代占据着不可替代的位置，快速更新，跨平台，体验一致，但是也有不可逾越的问题，那就是输入。输入主要存在如下问题：

- H5无法自动唤起键盘

- H5无法指定特定场景需要的键盘

  - 身份证
  - 纯数字
  - 数字密码键盘
  - 带小数点的数字键盘
  - 受设备影响大
  - 同一类型键盘在不同操作系统手机中表现不一致

- H5模拟键盘体验差

系统目前值提供三种类型的键盘：普通文字键盘、数字键盘（多了一行数字，输入不方便），电话键盘（无符号）

| ![background-1](/images/posts/nativeview/background-1.png) | ![background-2](/images/posts/nativeview/background-2.png) | ![background-3](/images/posts/nativeview/background-3.png) |
| ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| 文字键盘                                     | 数字键盘                                     | 电话键盘                                     |



无论是 iOS,还是 Android，h5只能用默认的键盘，很多特殊场景下仍然是普通键盘。类比下移动客户端定制的键盘，比如输入身份证（带 X），数字键盘（多个0的按钮），或者密码键盘，体验是非常好。这些特性能不能为 h5服务呢？答案是肯定的，本文接下来将介绍如何将原生定制键盘暴露给 H5无缝使用。

# 二、设计方案

## 2.1 总体方案

分为三个角色，用户、native和js(如下图)；

用户操作的时候，js阻止系统弹起默认键盘，然后收集h5输入框的一些信息，包括位置、大小、样式（具体哪些信息咱们在商量）；

Js将这些信息传递给native，native根据JS提供的信息，在webview之上生成新的view，新的view包含一个输入框和键盘，

提供给用户输入；

用户输入完成后（native不太懂，应该会触发类似于失去焦点的事件），native告诉js用户输入完毕，js将用户输入的信息再传递给用户

![design](/images/posts/nativeview/design.png)





## 2.2 界面展示方案

从宏观上看，就是下面一层webview，上面一层是native的view，

![design-1](/images/posts/nativeview/design-1.png)

如下为 native 计算控件位置和滑动距离的方式：

![native-calc](/images/posts/nativeview/native-calc.png)



## 2.3通信设计

这里建议使用 JsBridge,github 上有开源，加入如下两个通信接口：

一个methodname是 showKeyboard，并接收JS传递的输入框信息；

一个methodname是hideKeyboard，为js告诉native主动隐藏键盘和输入框（即销毁native-view）；



Native 的输入事件通过固定回调，传递给 h5。



## 三、H5使用

我们的愿景目的是，场景方在使用中0副作用，做到真正的渐进增强；所以需要 h5对事件进行二次封装，由于这一块并不是我实现的，所以本文暂时不介绍，有兴趣的同学可以单独找我沟通。

这里只简单介绍下使用方式：场景方仅需在相关代码上添加一行代码即可，在可以使用廊桥安全键盘的版本下会自动使用安全键盘，否则使用场景方默认逻辑，可以做到0侵入；

对于业务方来说，是0成本的；d

![bridge](/images/posts/nativeview/use1.png)	

![use2](/images/posts/nativeview/use2.png)



# 总结

本文介绍了一种使用原生控件来改善 h5的输入体验，类似微信和支付包小程序。当然后续可扩展的方式很多，比如图片控件，列表控件，或者各种动画，自定义控件等，最终是向着 ReactNative 方向的一个兼容方案。