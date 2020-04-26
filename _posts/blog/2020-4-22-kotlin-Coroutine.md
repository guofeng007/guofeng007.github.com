---
layout: post
title: Kotlin协程扫盲
categories: Blog
description:  Kotlin 协程
keywords:  Kotlin,协程

---

大佬三篇视频甩过来，学不会那是你笨

Kotlin 协程  https://www.bilibili.com/video/BV164411C7FK

Kotlin 协程的挂起  https://www.bilibili.com/video/BV1KJ41137E9

https://www.bilibili.com/video/BV1JE411R7hp

1. suspend只是一个标记作用，提示只能在协程使用
2. 协程高效的本质在于能够方便的切换和回到调用线程(能够通过context知道调用之前的线程，所以很方便切换)
3. 功能上与ExecutorService线程池一样，只是更方便高效
4. coroutineScope mainScope/GlobalScope/lifecycleScope，处理协程内存泄露，cancel()
5. 子范围

oroutineScope{

launch1

Launch2

}



6. Async  await 能够实现rxjava的各种异步合并等操作
