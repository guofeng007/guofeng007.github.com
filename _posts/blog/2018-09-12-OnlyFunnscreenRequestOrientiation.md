---
layout: post
title: Only fullscreen activities can request orientation？Android O开始的坑
categories: Blog
description: Only fullscreen activities can request orientation？Android O开始的坑
keywords:      Only fullscreen activities can request orientation？Android O开始的坑

---

# Only fullscreen activities can request orientation？Android O开始的坑



## 特征

当我们把targetSdkVersion升级到27，buildToolsVersion和相关的support library升级到27.0.1后，在Android 8.0（API level 26）上，部分Activity出现了一个莫名其妙的crash，异常信息如下：

java.lang.RuntimeException: Unable to start activity ComponentInfo{com.linkedin.android.XXXX.XXXX/com.linkedin.android.XXXX.XXXX.activity.LoginActivity}: java.lang.IllegalStateException: Only fullscreen activities can request orientation

当你在一个“translucent”的Activity里，试图执行setRequestedOrientation的时候就会触发这个异常。例如：

```java
setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE);
```

## 原因

这个问题貌似已经被广泛的讨论了，最终我们锁定了April 26的一个commit：

[Prevent non-fullscreen activities from influencing orientation · aosp-mirror/platform_frameworks_base@3979159](https://link.zhihu.com/?target=https%3A//github.com/aosp-mirror/platform_frameworks_base/commit/39791594560b2326625b663ed6796882900c220f%23diff-a9aa0352703240c8ae70f1c83add4bc8R981)

这个改动中抛出异常有关的代码如下：

```java
if (ActivityInfo.isFixedOrientation(requestedOrientation) 
    && !fullscreen
    && appInfo.targetSdkVersion >= O) {
    throw new IllegalStateException("Only fullscreen activities can request orientation");
}
```

基本的意思是说，透明的activity是不能锁定orientation的，否则抛出异常。下面，我们在看一下“fullscreen”如何定义的。

```java
public static boolean isTranslucentOrFloating(TypedArray attributes) { 
    final boolean isTranslucent = attributes.getBoolean(com.android.internal.R.styleable.Window_windowIsTranslucent, false); 
    final boolean isSwipeToDismiss = !attributes.hasValue( com.android.internal.R.styleable.Window_windowIsTranslucent) 
                                     && attributes.getBoolean( com.android.internal.R.styleable.Window_windowSwipeToDismiss, false); 
    final boolean isFloating = attributes.getBoolean(com.android.internal.R.styleable.Window_windowIsFloating, false);  
    return isFloating || isTranslucent || isSwipeToDismiss;    
}
```

根据上面的定义，如果一个Activity的Style符合下面三个条件之一，认为不是“fullscreen”：

1. “windowIsTranslucent”为true；
2. “windowIsTranslucent”为false，但“windowSwipeToDismiss”为true；
3. “windowIsFloating“为true；

综上可见，这个改动的目的是想阻止透明的Activity锁定屏幕旋转，因为当前Activity是透明的，浮动的或可滑动取消的，是否锁屏应该由全屏的Activity决定，而不是并没有全部占据屏幕的Activity决定。

## 修复

这个问题貌似在最新的SDK中已经修复，我们在API Level 27的设备上已经无法重现，但我们手头的API Level 26的设备还是能重现。而且根据上面的代码来看，如果想保留当前Activity的style，“isTranslucentOrFloating”的逻辑根本没法绕过，所以想绕开很难，目前能想到的大概两个方向：

1. 推迟SDK升级，等官方修复被大多数设备采用；
2. 升级SDK，但重构一下代码，看看已有的非“fullscreen” Activity是不是都是必要的，例如用Fragment实现周围半透明效果，能不能直接把Fragment加入到当前Activity（当然Detach Fragment是有重绘View的开销的）。

[Android P mr6去除限制](https://android.googlesource.com/platform/frameworks/base/+/39791594560b2326625b663ed6796882900c220f%5E%21/)