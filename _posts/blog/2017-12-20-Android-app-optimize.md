---
layout: post
title: Android 性能优化相关资料
categories: Blog
description: Android 性能优化相关资料
keywords: Android、性能、优化

---



# Android 性能优化相关资料

 

**转载**
[原文](https://link.juejin.im/?target=http%3A%2F%2Fandroidperformance.com%2F2017%2F10%2F19%2FAndroid-performance-optimization-skills-and-tools.html)
[译文](https://juejin.im/entry/5a3a1f7bf265da4319566b23?utm_source=gold_browser_extension)

# 优化心得和经验

1. [系列视频] Android Performance Patterns : [www.youtube.com/playlist?li…](https://link.juejin.im/?target=https%3A%2F%2Fwww.youtube.com%2Fplaylist%3Flist%3DPLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)
2. 给 App 提速：Android 性能优化总结 : [android.jobbole.com/81944/](https://link.juejin.im/?target=http%3A%2F%2Fandroid.jobbole.com%2F81944%2F)
3. 移动端性能监控方案 Hertz : [tech.meituan.com/hertz.html](https://link.juejin.im/?target=https%3A%2F%2Ftech.meituan.com%2Fhertz.html)
4. Android 性能优化后续 : [androidperformance.com/2015/03/31/…](https://link.juejin.im/?target=http%3A%2F%2Fandroidperformance.com%2F2015%2F03%2F31%2Fandroid-performance-case-study-follow-up.html)
5. Android性能优化之虚拟机调优 : [weishu.me/2016/12/23/…](https://link.juejin.im/?target=http%3A%2F%2Fweishu.me%2F2016%2F12%2F23%2Fdive-into-android-optimize-vm-heap%2F)
6. [译]Android UI 性能优化 : [zhuanlan.zhihu.com/p/27065828](https://link.juejin.im/?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F27065828)

# 响应速度

1. Optimizing Boot Times : [source.android.com/devices/tec…](https://link.juejin.im/?target=https%3A%2F%2Fsource.android.com%2Fdevices%2Ftech%2Fperf%2Fboot-times)
2. Android 中如何计算 App 的启动时间 : [androidperformance.com/2015/12/31/…](https://link.juejin.im/?target=http%3A%2F%2Fandroidperformance.com%2F2015%2F12%2F31%2FHow-to-calculation-android-app-lunch-time.html)

# 流畅度

1. Evaluating Performance : [source.android.com/devices/tec…](https://link.juejin.im/?target=https%3A%2F%2Fsource.android.com%2Fdevices%2Ftech%2Fdebug%2Feval_perf)
2. Understanding Systrace : [source.android.com/devices/tec…](https://link.juejin.im/?target=https%3A%2F%2Fsource.android.com%2Fdevices%2Ftech%2Fdebug%2Fsystrace)
3. Using ftrace : [source.android.com/devices/tec…](https://link.juejin.im/?target=https%3A%2F%2Fsource.android.com%2Fdevices%2Ftech%2Fdebug%2Fftrace)
4. Identifying Capacity-Related Jank : [source.android.com/devices/tec…](https://link.juejin.im/?target=https%3A%2F%2Fsource.android.com%2Fdevices%2Ftech%2Fdebug%2Fjank_capacity)
5. Identifying Jitter-Related Jank : [source.android.com/devices/tec…](https://link.juejin.im/?target=https%3A%2F%2Fsource.android.com%2Fdevices%2Ftech%2Fdebug%2Fjank_jitter)
6. 那些年我们用过的显示性能指标 : [blog.csdn.net/tencent_bug…](https://link.juejin.im/?target=http%3A%2F%2Fblog.csdn.net%2Ftencent_bugly%2Farticle%2Fdetails%2F51354517)

# 内存

1. Low RAM Configuration : [source.android.com/devices/tec…](https://link.juejin.im/?target=https%3A%2F%2Fsource.android.com%2Fdevices%2Ftech%2Fperf%2Flow-ram)
2. Linux Swap 与 Zram 详解 : [www.tinylab.cn/linux-swap-…](https://link.juejin.im/?target=http%3A%2F%2Fwww.tinylab.cn%2Flinux-swap-and-zramfs%2F%23zram-)
3. Android 加载不同 DPI 资源与内存消耗间的关系 : [www.tinylab.cn/android-loa…](https://link.juejin.im/?target=http%3A%2F%2Fwww.tinylab.cn%2Fandroid-loading-a-different-relationship-between-dpi-and-memory-consumption-of-resources%2F)
4. ZRAM SWAP 内存管理讲解 : [nekosc.com/technology/…](https://link.juejin.im/?target=https%3A%2F%2Fnekosc.com%2Ftechnology%2Fzram.html)
5. Android OOM 案例分析 : [tech.meituan.com/oom_analysi…](https://link.juejin.im/?target=https%3A%2F%2Ftech.meituan.com%2Foom_analysis.html)
6. Android 代码内存优化建议-Android 资源篇 : [androidperformance.com/2015/07/20/…](https://link.juejin.im/?target=http%3A%2F%2Fandroidperformance.com%2F2015%2F07%2F20%2FAndroid-Performance-Memory-AndroidResource.html)
7. Android 代码内存优化建议-Android 官方篇 : [androidperformance.com/2015/07/20/…](https://link.juejin.im/?target=http%3A%2F%2Fandroidperformance.com%2F2015%2F07%2F20%2FAndroid-Performance-Memory-Google.html)
8. Android 代码内存优化建议-Java 官方篇 : [androidperformance.com/2015/07/20/…](https://link.juejin.im/?target=http%3A%2F%2Fandroidperformance.com%2F2015%2F07%2F20%2FAndroid-Performance-Memory-Java.html)
9. Android 内存优化之一：MAT 使用入门 : [androidperformance.com/2015/04/11/…](https://link.juejin.im/?target=http%3A%2F%2Fandroidperformance.com%2F2015%2F04%2F11%2FAndroidMemory-Usage-Of-MAT.html)
10. Android 内存优化之二：MAT 使用进阶 : [androidperformance.com/2015/04/11/…](https://link.juejin.im/?target=http%3A%2F%2Fandroidperformance.com%2F2015%2F04%2F11%2FAndroidMemory-Usage-Of-MAT-Pro.html)
11. Android 内存优化之三：打开 MAT 中的 Bitmap 原图 : [androidperformance.com/2015/04/11/…](https://link.juejin.im/?target=http%3A%2F%2Fandroidperformance.com%2F2015%2F04%2F11%2FAndroidMemory-Open-Bitmap-Object-In-MAT.html)
12. Android 代码内存优化建议-OnTrimMemory 优化 : [androidperformance.com/2015/07/20/…](https://link.juejin.im/?target=http%3A%2F%2Fandroidperformance.com%2F2015%2F07%2F20%2FAndroid-Performance-Memory-onTrimMemory.html)
13. Android LowMemoryKiller原理分析 : [gityuan.com/2016/09/17/…](https://link.juejin.im/?target=http%3A%2F%2Fgityuan.com%2F2016%2F09%2F17%2Fandroid-lowmemorykiller%2F)
14. Android 匿名共享内存（Ashmem）原理 : [juejin.im/post/59e818…](https://link.juejin.im/?target=https%3A%2F%2Fjuejin.im%2Fpost%2F59e818bb6fb9a044fd10de38)

# 图形栈

1. Android 硬件加速原理与实现简介 ：[tech.meituan.com/hardware-ac…](https://link.juejin.im/?target=https%3A%2F%2Ftech.meituan.com%2Fhardware-accelerate.html)
2. Android6.0 显示系统（一） Surface 创建： [blog.csdn.net/kc58236582/…](https://link.juejin.im/?target=http%3A%2F%2Fblog.csdn.net%2Fkc58236582%2Farticle%2Fdetails%2F52670528)
3. Android6.0 显示系统（二） SurfaceFlinger 创建 Surface ：[blog.csdn.net/kc58236582/…](https://link.juejin.im/?target=http%3A%2F%2Fblog.csdn.net%2Fkc58236582%2Farticle%2Fdetails%2F52680288)
4. Android6.0 显示系统（三） 管理图像缓冲区 ： [blog.csdn.net/kc58236582/…](https://link.juejin.im/?target=http%3A%2F%2Fblog.csdn.net%2Fkc58236582%2Farticle%2Fdetails%2F52681363)
5. Android6.0 显示系统（五） SurfaceFlinger 服务 ： [blog.csdn.net/kc58236582/…](https://link.juejin.im/?target=http%3A%2F%2Fblog.csdn.net%2Fkc58236582%2Farticle%2Fdetails%2F52763534)
6. Android6.0 显示系统（六） 图像的输出过程 ： [blog.csdn.net/kc58236582/…](https://link.juejin.im/?target=http%3A%2F%2Fblog.csdn.net%2Fkc58236582%2Farticle%2Fdetails%2F52778333)
7. Android6.0 SurfaceControl 分析（一）SurfaceControl创建&使用 Surface创建&使用 ： [blog.csdn.net/kc58236582/…](https://link.juejin.im/?target=http%3A%2F%2Fblog.csdn.net%2Fkc58236582%2Farticle%2Fdetails%2F64918810)
8. Android6.0 SurfaceControl 分析（二）SurfaceControl和SurfaceFlinger通信
9. Android6.0 VSync 信号如何到用户进程 ： [blog.csdn.net/kc58236582/…](https://link.juejin.im/?target=http%3A%2F%2Fblog.csdn.net%2Fkc58236582%2Farticle%2Fdetails%2F65445141)
10. Android 图形系统概述 : [gityuan.com/2017/02/05/…](https://link.juejin.im/?target=http%3A%2F%2Fgityuan.com%2F2017%2F02%2F05%2Fgraphic_arch%2F)
11. Choreographer 原理 : [gityuan.com/2017/02/25/…](https://link.juejin.im/?target=http%3A%2F%2Fgityuan.com%2F2017%2F02%2F25%2Fchoreographer%2F)
12. SurfaceFlinger 启动篇 : [gityuan.com/2017/02/11/…](https://link.juejin.im/?target=http%3A%2F%2Fgityuan.com%2F2017%2F02%2F11%2Fsurface_flinger%2F)
13. SurfaceFlinger 绘图篇 : [gityuan.com/2017/02/18/…](https://link.juejin.im/?target=http%3A%2F%2Fgityuan.com%2F2017%2F02%2F18%2Fsurface_flinger_2%2F)
14. [HWUI]Android 应用程序 UI 硬件加速渲染技术简要介绍和学习计划 : [blog.csdn.net/luoshengyan…](https://link.juejin.im/?target=http%3A%2F%2Fblog.csdn.net%2Fluoshengyang%2Farticle%2Fdetails%2F45601143)
15. [HWUI]Android 应用程序 UI 硬件加速渲染环境初始化过程分析 : [blog.csdn.net/luoshengyan…](https://link.juejin.im/?target=http%3A%2F%2Fblog.csdn.net%2Fluoshengyang%2Farticle%2Fdetails%2F45769759)
16. [HWUI]Android 应用程序 UI 硬件加速渲染的预加载资源地图集服务（Asset Atlas Service）分析 : [blog.csdn.net/luoshengyan…](https://link.juejin.im/?target=http%3A%2F%2Fblog.csdn.net%2Fluoshengyang%2Farticle%2Fdetails%2F45831269)
17. [HWUI]Android 应用程序 UI 硬件加速渲染的 Display List 构建过程分析 : [blog.csdn.net/luoshengyan…](https://link.juejin.im/?target=http%3A%2F%2Fblog.csdn.net%2Fluoshengyang%2Farticle%2Fdetails%2F45943255)
18. [HWUI]Android 应用程序 UI 硬件加速渲染的 Display List 渲染过程分析 : [blog.csdn.net/luoshengyan…](https://link.juejin.im/?target=http%3A%2F%2Fblog.csdn.net%2Fluoshengyang%2Farticle%2Fdetails%2F46281499)
19. [HWUI]Android 应用程序 UI 硬件加速渲染的动画执行过程分析 : [blog.csdn.net/luoshengyan…](https://link.juejin.im/?target=http%3A%2F%2Fblog.csdn.net%2Fluoshengyang%2Farticle%2Fdetails%2F46449677)

# 虚拟机

1. ART and Dalvik : [source.android.com/devices/tec…](https://link.juejin.im/?target=https%3A%2F%2Fsource.android.com%2Fdevices%2Ftech%2Fdalvik%2F)
2. Android 8.0 ART Improvements : [source.android.com/devices/tec…](https://link.juejin.im/?target=https%3A%2F%2Fsource.android.com%2Fdevices%2Ftech%2Fdalvik%2Fimprovements)
3. Dalvik bytecode : [source.android.com/devices/tec…](https://link.juejin.im/?target=https%3A%2F%2Fsource.android.com%2Fdevices%2Ftech%2Fdalvik%2Fdalvik-bytecode)
4. Dalvik Executable format : [source.android.com/devices/tec…](https://link.juejin.im/?target=https%3A%2F%2Fsource.android.com%2Fdevices%2Ftech%2Fdalvik%2Fdex-format)
5. Dalvik Executable instruction formats : [source.android.com/devices/tec…](https://link.juejin.im/?target=https%3A%2F%2Fsource.android.com%2Fdevices%2Ftech%2Fdalvik%2Finstruction-formats)
6. Constraints : [source.android.com/devices/tec…](https://link.juejin.im/?target=https%3A%2F%2Fsource.android.com%2Fdevices%2Ftech%2Fdalvik%2Fconstraints)
7. Configuring ART : [source.android.com/devices/tec…](https://link.juejin.im/?target=https%3A%2F%2Fsource.android.com%2Fdevices%2Ftech%2Fdalvik%2Fconfigure)
8. Debugging ART Garbage Collection : [source.android.com/devices/tec…](https://link.juejin.im/?target=https%3A%2F%2Fsource.android.com%2Fdevices%2Ftech%2Fdalvik%2Fgc-debug)
9. Implementing ART Just-In-Time (JIT) Compiler : [source.android.com/devices/tec…](https://link.juejin.im/?target=https%3A%2F%2Fsource.android.com%2Fdevices%2Ftech%2Fdalvik%2Fjit-compiler)

# 系统框架

1. Task Snapshots :[source.android.com/devices/tec…](https://link.juejin.im/?target=https%3A%2F%2Fsource.android.com%2Fdevices%2Ftech%2Fperf%2Ftask-snapshots)
2. Android Input 子系统：Input 进程的创建，监听线程的启动 : [zhuanlan.zhihu.com/p/29152319](https://link.juejin.im/?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F29152319)
3. Android Input 子系统：Input 事件的产生、读取和分发，InputReader、InputDispatcher : [zhuanlan.zhihu.com/p/29386642](https://link.juejin.im/?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F29386642)
4. EventHub 与设备、Input 事件的交互 : [zhuanlan.zhihu.com/p/30127752](https://link.juejin.im/?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F30127752)
5. Android 消息机制，从Java 层到 Native 层剖析 : [zhuanlan.zhihu.com/p/29929031](https://link.juejin.im/?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F29929031)
6. 理解 Android Binder 机制(1/3)：驱动篇 : [qiangbo.space/2017-01-15/…](https://link.juejin.im/?target=http%3A%2F%2Fqiangbo.space%2F2017-01-15%2FAndroidAnatomy_Binder_Driver%2F)
7. 理解 Android Binder 机制(2/3)：C++ 层 : [qiangbo.space/2017-02-12/…](https://link.juejin.im/?target=http%3A%2F%2Fqiangbo.space%2F2017-02-12%2FAndroidAnatomy_Binder_CPP%2F)
8. 理解 Android Binder 机制(3/3)：Java 层 : [qiangbo.space/2017-03-15/…](https://link.juejin.im/?target=http%3A%2F%2Fqiangbo.space%2F2017-03-15%2FAndroidAnatomy_Binder_Java%2F)
9. Android Bander 设计与实现 - 设计篇 : [blog.csdn.net/universus/a…](https://link.juejin.im/?target=http%3A%2F%2Fblog.csdn.net%2Funiversus%2Farticle%2Fdetails%2F6211589)
10. 四大组件之综述 : [gityuan.com/2017/05/19/…](https://link.juejin.im/?target=http%3A%2F%2Fgityuan.com%2F2017%2F05%2F19%2Fams-abstract%2F)
11. 四大组件之 ActivityRecord : [gityuan.com/2017/06/11/…](https://link.juejin.im/?target=http%3A%2F%2Fgityuan.com%2F2017%2F06%2F11%2Factivity_record%2F)
12. 四大组件之 ContentProviderRecord : [gityuan.com/2017/06/04/…](https://link.juejin.im/?target=http%3A%2F%2Fgityuan.com%2F2017%2F06%2F04%2Fcontent_provider_record%2F)
13. 四大组件之 BroadcastRecord : [gityuan.com/2017/06/03/…](https://link.juejin.im/?target=http%3A%2F%2Fgityuan.com%2F2017%2F06%2F03%2Fbroadcast_record%2F)
14. 四大组件之 ServiceRecord : [gityuan.com/2017/05/25/…](https://link.juejin.im/?target=http%3A%2F%2Fgityuan.com%2F2017%2F05%2F25%2Fservice_record%2F)
15. 简述 Activity 与 Window 关系 : [gityuan.com/2017/04/16/…](https://link.juejin.im/?target=http%3A%2F%2Fgityuan.com%2F2017%2F04%2F16%2Factivity-with-window%2F)
16. 理解 Android Context : [gityuan.com/2017/04/09/…](https://link.juejin.im/?target=http%3A%2F%2Fgityuan.com%2F2017%2F04%2F09%2Fandroid_context%2F)
17. 理解 Application 创建过程 : [gityuan.com/2017/04/02/…](https://link.juejin.im/?target=http%3A%2F%2Fgityuan.com%2F2017%2F04%2F02%2Fandroid-application%2F)
18. 以 Window 视角来看 startActivity : [gityuan.com/2017/01/22/…](https://link.juejin.im/?target=http%3A%2F%2Fgityuan.com%2F2017%2F01%2F22%2Fstart-activity-wms%2F)
19. WMS—启动窗口(StartingWindow) : [gityuan.com/2017/01/15/…](https://link.juejin.im/?target=http%3A%2F%2Fgityuan.com%2F2017%2F01%2F15%2Fwms_starting_window%2F)
20. WMS—启动过程 : [gityuan.com/2017/01/08/…](https://link.juejin.im/?target=http%3A%2F%2Fgityuan.com%2F2017%2F01%2F08%2Fwindowmanger%2F)

# 进程管理

1. cpuset ： [www.kernel.org/doc/Documen…](https://link.juejin.im/?target=https%3A%2F%2Fwww.kernel.org%2Fdoc%2FDocumentation%2Fcgroup-v1%2Fcpusets.txt)
2. cgroup ： [www.kernel.org/doc/Documen…](https://link.juejin.im/?target=https%3A%2F%2Fwww.kernel.org%2Fdoc%2FDocumentation%2Fcgroup-v1%2Fcgroups.txt)
3. Android 进程调度之 adj 算法 [gityuan.com/2016/08/07/…](https://link.juejin.im/?target=http%3A%2F%2Fgityuan.com%2F2016%2F08%2F07%2Fandroid-adj%2F)
4. Linux 进程管理(一) [gityuan.com/2017/07/30/…](https://link.juejin.im/?target=http%3A%2F%2Fgityuan.com%2F2017%2F07%2F30%2Flinux-process%2F)
5. Linux 进程管理(二)–fork [gityuan.com/2017/08/05/…](https://link.juejin.im/?target=http%3A%2F%2Fgityuan.com%2F2017%2F08%2F05%2Flinux-process-fork%2F)
6. Linux 进程 pid 分配法 [gityuan.com/2017/08/06/…](https://link.juejin.im/?target=http%3A%2F%2Fgityuan.com%2F2017%2F08%2F06%2Flinux_process_pid%2F)
7. [收费培训视频] 打通 Linux 脉络系列：进程、线程和调度 : [edu.csdn.net/course/deta…](https://link.juejin.im/?target=http%3A%2F%2Fedu.csdn.net%2Fcourse%2Fdetail%2F5995)
8. Android 系统中的进程管理：进程的创建 : [qiangbo.space/2016-10-10/…](https://link.juejin.im/?target=http%3A%2F%2Fqiangbo.space%2F2016-10-10%2FAndroidAnatomy_Process_Creation%2F)
9. Android 系统中的进程管理：进程的优先级 : [qiangbo.space/2016-11-23/…](https://link.juejin.im/?target=http%3A%2F%2Fqiangbo.space%2F2016-11-23%2FAndroidAnatomy_Process_OomAdj%2F)
10. Android 系统中的进程管理：内存的回收 : [qiangbo.space/2016-12-08/…](https://link.juejin.im/?target=http%3A%2F%2Fqiangbo.space%2F2016-12-08%2FAndroidAnatomy_Process_Recycle%2F)
11. Android 系统启动：init 进程与 init 语言 : [qiangbo.space/2017-01-28/…](https://link.juejin.im/?target=http%3A%2F%2Fqiangbo.space%2F2017-01-28%2FAndroidAnatomy_Init%2F)

# 调试工具

1. 另一个 Android 性能剖析工具——simpleperf : [zhuanlan.zhihu.com/p/25277481](https://link.juejin.im/?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F25277481)
2. 如何调试 Android Framework : [weishu.me/2016/05/30/…](https://link.juejin.im/?target=http%3A%2F%2Fweishu.me%2F2016%2F05%2F30%2Fhow-to-debug-android-framework%2F)
3. 如何调试 Android Native Framework : [weishu.me/2017/01/14/…](https://link.juejin.im/?target=http%3A%2F%2Fweishu.me%2F2017%2F01%2F14%2Fhow-to-debug-android-native-framework-source%2F)
4. Catapult : [catapult.gsrc.io/README.md](https://link.juejin.im/?target=https%3A%2F%2Fcatapult.gsrc.io%2FREADME.md)
5. 手把手教你使用Systrace（一）: [zhuanlan.zhihu.com/p/27331842](https://link.juejin.im/?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F27331842)
6. 手把手教你使用Systrace（二）——锁优化
   : [zhuanlan.zhihu.com/p/27535205](https://link.juejin.im/?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F27535205)

# 硬件相关

1. Flash Wear Management in Android Automotive : [source.android.com/devices/tec…](https://link.juejin.im/?target=https%3A%2F%2Fsource.android.com%2Fdevices%2Ftech%2Fperf%2Fflash-wear)
2. Cortex-A75 和 Cortex-A55 : [www.10tiao.com/html/431/20…](https://link.juejin.im/?target=http%3A%2F%2Fwww.10tiao.com%2Fhtml%2F431%2F201706%2F2650236929%2F1.html)
3. CPU Utilization is Wrong : [www.brendangregg.com/blog/2017-0…](https://link.juejin.im/?target=http%3A%2F%2Fwww.brendangregg.com%2Fblog%2F2017-05-09%2Fcpu-utilization-is-wrong.html)

# 编程语言

1. 探索 Java 隐藏的开销 : [academy.realm.io/cn/posts/36…](https://link.juejin.im/?target=https%3A%2F%2Facademy.realm.io%2Fcn%2Fposts%2F360andev-jake-wharton-java-hidden-costs-android%2F)

# Kernel

1. 内核探索：Regmap 框架：简化慢速 I/O 接口优化性能 [www.tinylab.cn/kernel-expl…](https://link.juejin.im/?target=http%3A%2F%2Fwww.tinylab.cn%2Fkernel-explore-regmap-framework%2F)
2. 嵌入式 Linux 启动时间优化 [www.tinylab.cn/elinux-org-…](https://link.juejin.im/?target=http%3A%2F%2Fwww.tinylab.cn%2Felinux-org-boot-time-optimization%2F)

# 我辈楷模

1. 我到底有多么努力 : [mp.weixin.qq.com/s?__biz=MzU…](https://link.juejin.im/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzU3ODAxNDcwNQ%3D%3D%26amp%3Bmid%3D2247484147%26amp%3Bidx%3D1%26amp%3Bsn%3Dcd5f8fead3bcaac2d22a3dd699d2e79f%26amp%3Bchksm%3Dfd7a9e6dca0d177b27095d3d12720e83ba1638028799a89a8879929c1ad442529a62a46c5fe3%26amp%3Bmpshare%3D1%26amp%3Bscene%3D1%26amp%3Bsrcid%3D1027hi7FsUIG3AirEiJg198C%23rd)
2. 工作以来的一些感悟 : [zhuanlan.zhihu.com/zmywly8866/…](https://link.juejin.im/?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fzmywly8866%2F20711335)
3. 如何自学Android？[zhuanlan.zhihu.com/p/20708611](https://link.juejin.im/?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F20708611)