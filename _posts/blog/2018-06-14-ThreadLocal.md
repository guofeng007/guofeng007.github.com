---
layout: post
title: ThreadLocal 源码调试解毒
categories: Blog
description:  ThreadLocal 源码调试解毒
keywords:      ThreadLocal 源码调试解毒

---

# ThreadLocal 源码调试解毒
![img](/images/posts/threadlocal/ThreadLocal.webp)



## 简介

如上图所示，Thread.mThreadLocal 记录着ThreadLocalMap,这个 Map 实现类似 HashMap，使用数组实现，动态扩容，容量为2 n 次幂，保证 hash 计算可以使用高效的位计算。
## 全局观

Thread.[ThreadLocal1.value,ThreadLocal2.value]

## 本质

不同的线程上面可以挂接很多个线程关联的对象，这样就巧妙的封装了线程内数据。（这不是做多线程同步的，是线程内部数据，不要乱用）