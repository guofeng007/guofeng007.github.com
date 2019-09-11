---
layout: post
title: Android热修复原理概览
categories: Blog
description: Android 热修复原理概述
keywords: Android、热修复、原理、概述

---



# Android热修复原理概览

 

**作者：国风**


# 背景
传统 APP 出现严重 bug 时，只能通过重新发布修复版本的安装包，其升级流程为：APP紧急Bug修复上线周期7天,依次修复、测试、发布、用户更新下载，整个周期可能持续一两个月，才能达到90%的更新率，而且更新的时候是下载整个安装包，在移动网络流量有限的环境下，很多用户是拒绝更新的。因此需要一种对用户而言，低成本的更新方案，热修复便应运而生。

热修复2015年爆发、其主要表现形式为动态下发补丁代码，高效及时的解决 APP 存在的 bug，本文接下来将对插件化这两年的成果进行阐释，各位看官请继续向下看。

# 热修复详解

##  热修复能够解决的问题

1. dex 代码修复
2. so 原生库修复
3. resource 资源修复

## 热修复流程

1. 输出补丁包(开发透明、单独编写)
2. 下载补丁包
3. 校验补丁包(加密、验签)
4. 加载补丁包

## 热修复主流方案

### Native Hook

#### 方案一

![NativeHook-plan1](/images/posts/hotfix/NativeHook-plan1.png)

#### 方案二

![NativeHook-plan2](/images/posts/hotfix/NativeHook-plan2.png)

#### 典型代表：阿里 AndFix

![NativeHook-ex1-andfix](/images/posts/hotfix/NativeHook-ex1-andfix.png)


### MultiDex ClassLoader

#### ClassLoader 结构

![MultiDex-1-classloader](/images/posts/hotfix/MultiDex-1-classloader.png)

ClassLoader 加载过程：

![Multidex-2-classloader-load](/images/posts/hotfix/Multidex-2-classloader-load.png)

ClassLoader 加载顺序:

![MultiDex-2-classloader-seq](/images/posts/hotfix/MultiDex-2-classloader-seq.png)

#### ClassLoader 存在的问题

![Multidex-barry](/images/posts/hotfix/Multidex-barry.png)

#### MultiDex 方案典型代表：QZone, Tinker, QFix
Class preverified问题解决方案

1. 插桩（突破条件2）：QZone
2. 全量替换（突破条件3）：Tinker
3. Native调用dvmResolveClass（突破条件1）：QFix

##### QZone

![Multidex-qzone](/images/posts/hotfix/Multidex-qzone.png)

1. 性能问题
2. ART模式内联函数内存地址错乱问题
  把补丁类所有相关类统一打入补丁包
##### Tinker

![MultiDex-Tinker](/images/posts/hotfix/MultiDex-Tinker.png)


### InstantRun

#### 典型代表:美团 Robust

#### 原理

![InstantRun-solution-2](/images/posts/hotfix/InstantRun-solution-2.png)

![InstantRun-solution](/images/posts/hotfix/InstantRun-solution.png)

#### 代码实例

![InstantRun-ex1](/images/posts/hotfix/InstantRun-ex1.png)


# 热修复方案对比

| 指标    | AndFix | Qzone | Tinker | InstantRun |
| ----- | ------ | ----- | ------ | ---------- |
| 开发透明  | no     | yes   | yes    | no         |
| 性能损耗  | 较小     | 较大    | 较小     | 较小         |
| 兼容性   | 一般     | 较高    | 较高     | 最高         |
| 补丁包大小 | 一般     | 较大    | 较小     | 一般         |
| 即使生效  | yes    | no    | no     | yes        |

# 热修复区别与插件

热修复集中在快速修复、体积小、及时。

插件化重在业务模块化，动态发布。

二者在一定条件下可以混用，但是最好按照其意图来使用。

# 注意事项

热修复不是请客吃饭（慎用），不要当作解bug的工具，热修复的存在时为了解决线上突发紧急问题。平时还是需要加强代码质量，不要以为有热修复就可以高枕无忧了，热修复不是万金油，整个平台建设仍然是重中之重。后续仍然不可避免的遇到兼容性问题，这些都是后续需要考虑的问题。

# 总结

 从2015年开始，热修复，插件化一直是安卓开发的超级热点，随着2017年的结束，热修复、插件化也不断趋于稳定。天朝各大厂商依次开源了自己的热修复：美团，微信等，插件框架，包括滴滴，360等。作为普通开发者的我们，应该好好珍惜这个节点，利用开源方案不断学习巩固自己的安卓系统知识，为高级开发者做准备。

# 附录：

### 字节码操作工具库



- ASM

  ​	- 字节码操作框架

  ​	- 基于虚拟机指令

  ​	- 动态生成字节码、增强现有字节码

  ​	- 很多工具都是基于ASM框架：dex2jar

- JavaAssist

  ​	- 基于Java API

  ​	- 动态生成字节码、增强现有字节码

   - 主要用于对性能要求不高的场景

### 开源插件化框架

### 开源热修复


  ​