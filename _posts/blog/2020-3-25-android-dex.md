---
layout: post
title: 直接从 APK 运行嵌入式 DEX 代码
categories: Blog
description:直接从 APK 运行嵌入式 DEX 代码
keywords: 直接从 APK 运行嵌入式 DEX 代码
---

## 背景

Android使用AOT编译dex为本地代码之后，后续运行直接加载本地编译后的代码，如果攻击者设法篡改了设备上本地编译的代码，那么整个应用将收到攻击。

## 解决方案

从 Android 10 开始，可以通过编译配置来告诉系统直接从应用的 APK 文件中运行嵌入式 DEX 代码，不要再去编译为本地代码然后再运行

- 注意：启用此功能可能会影响应用的性能，因为在应用启动时 ART 必须使用 JIT 编译器（而不是读取提前编译好的原生代码）。

- 因此要先测试应用性能，然后再决定是否在已发布的应用中启用此功能。

## 步骤


1. 请在应用清单文件的 <application> 元素中将 android:useEmbeddedDex 属性的值设为 true。

2. 再gradle中添加如下配置， 以编译包含未压缩 DEX 代码的 APK：

```Java

aaptOptions {
       noCompress 'dex'
    }
    
```
