

---
layout: post
title: flutter Crash上报方案
categories: Blog
description:  上报flutter crash
keywords:  flutter,crash

---

# 一、官方

已废弃，很不好用

# 二、flutter_boot 闲鱼方案

[咸鱼flutter_boot方案详解](https://mp.weixin.qq.com/s/v-wwruadJntX1n-YuMPC7g)
[github](https://github.com/alibaba-flutter/flutter-boot)

优点：稳定，咸鱼是国内最早开始投入flutter研发的团队，能够从源码层进行定制开发

# 三、thrio 哈罗单车方案

[thrio](https://github.com/hellobike/thrio)
[接入](https://github.com/hellobike/thrio/blob/master/doc/Feature.md)

基于flutter_boot
优点：在页面复用方面进行了优化，同时导航栈也更为全面，还有3端的通信也是比较完备的

# 四、总结

建议采用thrio，在性能，功能，可用性上做的比较均衡。





