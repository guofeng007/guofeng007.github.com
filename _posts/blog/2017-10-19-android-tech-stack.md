---
layout: post
title: Android技术栈
categories: Blog
description: Android技术发展路线
keywords: Android 历史、版本、技术栈
---



# Android技术栈

 

**作者：国风**

![Android-Oreo](/images/posts/android-tech-stack/oreo-superhero_2x.png)
​      

Android 从2007年发布第一个 beta 版本，截至目前（2017-10）已经到了Android8.0,总计26个版本左右。从 Android1.0版本开始，Android 一直处于不断的进化中，作为选择了 Android 开发的我们，如何在 Android 迭代发展中学习扩展，发展良好的职业生涯呢？接下来本文将给出作者自己的学习路线作为参考，抛砖引玉。


# 1. Android 历史及特性

## 1.1 Android 诞生

Android最初是由Andry Rubin等人在2003年10月创建了Android公司，并且创建了Android团队。
2005年8月17日 Google公司用5000万美金收购了成立仅仅22个月的Android公司以及其团队。这可以说是目前Google公司历史上最成功的一次收购了，仅仅5000万美金，Google太值了。现在Android的市场占有率赶超IOS，达到（46%多）。Android系统虽然是开源免费的，但是Android系统有Google的服务是要收费的（地图、邮箱、AppStore等等服务国内无法享用）。最重要的是他打断了IOS的垄断，对公司的整体布局有着非常大的贡献。

## 1.2 Android 历史及特性

| Android 版本号                              | 代号                       | API  | 备注               |
| ---------------------------------------- | ------------------------ | ---- | ---------------- |
| Android 1.0beta   2007.11.5              |                          |      |                  |
| Android  1.0  (发行前有两个测试版本以机器人名称命名 铁臂阿童木 Astro Boy 发条机器人 Bender) | 没有代号                     | 1    |                  |
| Android  1.1                             |                          | 2    | 2009.2.2         |
| Android 1.5                              | Cupcake  纸杯蛋糕            | 3    | 2009.4.17        |
| Android  1.6                             | Dount 甜甜圈                | 4    |                  |
| Android  2.0                             | Eclair  松饼               | 5    |                  |
| Android  2.0.1                           |                          | 6    |                  |
| Android  2.1                             |                          | 7    |                  |
| Android  2.2                             | Froyo 冻酸奶                | 8    |                  |
| Android  2.3-2.3.2                       | Gingerbread  姜饼          | 9    |                  |
| Android  2.3.3-2.3.7                     |                          | 10   |                  |
| Android  3.0                             | Honeycomb                | 11   | 仅在平板上            |
| Android  3.1                             |                          | 12   |                  |
| Android 4.0                              | Ice Cream Sandwich 雪糕三明治 | 14   | 统一了手机和平板个人认为非常重要 |
| Android  4.0.3-4.0.4                     |                          | 15   |                  |
| Android  4.1                             | JellyBean 果冻豆            | 16   |                  |
| Android  4.2                             |                          | 17   |                  |
| Android  4.3                             |                          | 18   |                  |
| Android  4.4                             | KitKat 奇巧                | 19   |                  |
| Android  4.4W                            |                          | 20   | WebView版本升级      |
| Android  5.0                             | Lollipop棒棒糖              | 21   | Masterial Design |
| Android  5.1                             | Lollipop棒棒糖              | 22   |                  |
| Android  6.0                             | Marshmallow              | 23   | 动态权限             |
| Android 7.0                              | Nougat                   | 24   | 多窗口              |
| Android 7.1                              |                          | 25   |                  |
| Android 8.0                              | Oreo                     | 26   | 画中画、通知、全面屏       |

## 1.3 Android 主流版本市场占有率

![Android 版本占有率](/images/posts/android-tech-stack/Android-distribution.png)

详情可参考：[Android 各版本市场占有率](https://developer.android.com/about/dashboards/index.html)
​       

# 2. Android 技术栈

## 2.1 Android 体系架构
![软件栈](/images/posts/android-tech-stack/android-stack_2x.png)

Android 软件栈如上图所示，从下往上依次为 Linux 层，HAL硬件抽象层，Runtime&Library 层，Framework层，应用 APP 层。

我们通常的工作是在最上层应用层开发，直接面对的是 Android Framework 提供的 API,但是仅仅掌握 API 如何使用，如何写界面是不够的，顶多是一个普通码农，而不是Android开发者。这个在其他平台上也是如此，我们不能仅仅满足于完成工作任务。

要成为一个 Android 开发者，应该继续向下深入，对 Framework,Runtime 进行一定程度的学习，具体的深度决定了我们的能力。
​       

## 2.2 Andrid 学习栈
![Android-tech-stack](/images/posts/android-tech-stack/Android-tech-stack.png)
如上图所示，本文将学习栈集中于APP应用层和框架 Framework 层。将这两个层进行细化，可以得到如下四个领域的学习：
### 2.2.1 APP基础

1. 能够胜任各种自定义 View 控件的开发，熟练掌握 View 绘制的主要流程，onMeasure 测量，onLayout 位置布局,onDraw 绘制。同时熟练掌握触摸事件分发机制。



2. 对 Android 四大组件的应用得心应手，包括：


	- 熟练掌握 Activity 生命周期，任务栈
	- BroadcastReceiver 生命周期以及注意点
	- Service 特性以及作为后台任务应该注意的地方
	- ContentProvider 共享数据

3. 了解 Gradle 原理，熟悉 Android APP 打包流程，熟练常用打包方式，能够自定义 task 或者插件，完成通用打包功能。

4. 对Router 解耦框架有一定的掌握，保证 APP 能够模块化，高内聚，低耦合。

5. 性能优化：内存，磁盘，网络，APK 瘦身，埋点统计，Crash 监控。


### 2.2.2 开源库 
如下列举的开源库不仅仅要掌握怎么用，更重要的是理解其设计思想和实现原理，遇到 各种定制话需求或者bug 的时候能够找到合适的方案去解决

- ​ 网络：Retrofit,OkHttp

- ​ 异步:RxJava

- ​ 通信：EventBus

- ​ 图片加载：Glide

- ​ AOP 依赖注入框架: Dagger

### 2.2.3 设计模式

- MVC MVP MVVM
- Google Clean Architecture & Architecture Components
- HeadFirst 设计模式

### 2.2.4 热门框架技术
 - 插件化
 - 热修复
 - Hybrid 混合开发
 - ReactNative

### 2.2.5 系统底层学习
 - Android Framework 框架源码学习（四大组件源码运行机制：Activity 启动流程，BroadcastReceiver 机制，Service 机制，ContentProvider 原理）
 - Binder 原理学习
 - NDK 学习

# 3. 总结
相信掌握以上基础之后，应该可成为一个合格的 Android 开发者，找到心仪的工作，未来职业规划可以站在更高的角度去思考。

