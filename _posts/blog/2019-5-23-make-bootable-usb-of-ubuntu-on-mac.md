---
layout: post
title: MAC制作 Ubuntu 启动 USB、安装 Ubuntu 学习 AI、Android 源码
categories: Blog
description: MAC制作 Ubuntu 启动 USB、安装 Ubuntu 学习 AI、Android 源码
keywords: MAC、制作、Ubuntu、启动USB、安装 Ubuntu 学习 AI、Android 源码

---




# 步骤如下

## 1.ISO转换位 DMG
   
```Java
  hdiutil convert -format UDRW -o ./ubuntu-18.04-desktop-amd64 ./ubuntu-18.04-desktop-amd64.iso    
```

## 2.得到 U 盘设备号
   
```java
diskutil list 
dev/diskx
```

## 3.卸载 U 盘设备
   
```java
 diskutil unmountDisk /dev/disk2    
```

## 4.创建可启动的 USB 驱动盘

```java
 sudo dd if=ubuntu-18.04-desktop-amd64.dmg of=/dev/disk2 bs=1m    
```

## 5.弹出磁盘
   
 ```java
 diskutil eject /dev/disk2    
 ```