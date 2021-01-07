---
layout: post
title: Android逆向其他APP工具推荐【免Root】
categories: [Blog,Cat]
description: 与传统的反编译或者root后debug逆向相比，本方案更加轻量，不需要root，也不需要额外操作即可，方便好用
keywords: BoostMutiDex，MultiDex
---

# 一、背景

最近公司需要调研一个竞品的技术实现方案，需要一些逆向技术进行分析。

# 二、传统逆向工具与方法

传统的分析流程和工具一般都是

1. 想看源代码:先解压apk(apk就是zip压缩格式，改个后缀，用zip工具解压即可)，然后使用dex2jar.jar 反编译为jar进行查看
2. 想看manifest或者其他资源文件：直接拖拽到Android Studio（或者用tools->apk analyzer），可以看manifest声明以及资源文件
3. 配置抓包，看与后端的交互流程参数和返回值（现在Android 7.0以上，APP默认不会信任抓包的证书，此方法失效，所以需要在底版本系统上使用）
4. 想debug webview内容：没途径
5. 想抓websocket包，默认websocket不会走系统代理，很难抓到

# 三、 轻便方案


与传统的反编译或者root后debug逆向相比，本方案更加轻量，不需要root，也不需要额外操作即可，方便好用

1. 想看源代码、manifest、资源：  jadx 直接打开查看，还可以导出为一个gradle 工程，导入 stuido查看
2. 配置抓包：直接先安装virtualxposed，然后在virtualxposed内部克隆应用，打开后即可抓包
3. 想debug webview内容：在virtualxposed模块中下载WebviewDebug 插件，开启后，就可以链接数据线，在电脑上使用chrome浏览器，在地址栏输入 about://inspect就能看到可调试的网页
4. 想抓websocket包，安装HttpCanary，将其设置为系统vpn,然后就能抓到所有http,websocket包

# 四、针对加固的应用

可以尝试业界常用的破壳工具，或者在内存中dump dex这两种方案，目前暂未使用这个技巧





所需工具附件：链接: https://pan.baidu.com/s/1kOCSkYQGJnEWsX2Ggi9o7w 提取码: pgfw 