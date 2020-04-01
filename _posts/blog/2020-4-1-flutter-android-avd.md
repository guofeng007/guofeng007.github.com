---
layout: post
title: flutter-Android-AVD-Error
categories: Blog
description:  ANDROID_SDK_ROOT
keywords:  ANDROID_SDK_ROOT

---

今天捣鼓flutter的时候突然遇到一个报错，怎么弄都不行，[原文](https://www.sunzhongwei.com/android-studio-system-environment-variables)
报错信息为：

```
Emulator: Process finished with exit code 1
Emulator: PANIC: Cannot find AVD system path. Please define ANDROID_SDK_ROOT
```

昨天还好好的，为何今天就找不到 AVD 路径了呢。。。

Android Studio 真是废柴！Google，Baidu 查了半天，无论 StackOverflow 还是 CSDN 都是无脑的解决方案，毫无帮助。

最后破釜沉舟，删除 AVD 里所有的镜像。在系统环境变量里设置：

![Emulator: PANIC: Cannot find AVD system path. Please define ANDROID_SDK_ROOT](https://cdn.sunzhongwei.com/sunzhongwei_5df792de2a0d3)

![Emulator: PANIC: Cannot find AVD system path. Please define ANDROID_SDK_ROOT](https://cdn.sunzhongwei.com/sunzhongwei_5df792e5ef3f1)

- 新增 ANDROID_SDK_HOME 环境变量。其值为 D 盘一个新建的目录
- ANDROID_HOME 原来就有，无需修改

然后重启 Android Studio，使环境变量生效。

再次打开 Tools -> AVD Manager 安装一个镜像，启动即可。

然后就可以正常启动模拟器了。

为何是设置 ANDROID_SDK_HOME 而不是 ANDROID_SDK_ROOT？ANDROID_SDK_HOME 到底有啥用？可以参考 [Android Studio 相关的系统环境变量](https://www.sunzhongwei.com/android-studio-system-environment-variables) 里的说明。概况来说，启动模拟器时会从 ANDROID_SDK_HOME 指定的目录查找 AVD 目录（正规来说应该是设置成 ANDROID_AVD_HOME）。估计是 Android Studio 这里写错成了 ANDROID_SDK_ROOT。

## Broken AVD system path

如果你按照 CSDN 上的那群小学生的建议将 ANDROID_SDK_ROOT 设置成跟 ANDROID_HOME 一样的值，会收到报错：

> Emulator: PANIC: Broken AVD system path. Check your ANDROID_SDK_ROOT value [D:\android_sdk]!

好了，一晚上就被这种低级问题给毁了。
