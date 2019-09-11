---
layout: post
title: 《Android 开发艺术探索》示例代码修复
categories: Blog
description: 《Android 开发艺术探索》示例代码修复
keywords:      《Android 开发艺术探索》示例代码修复

---

# android-art-res-new

相信很多人都看了《Android 开发艺术探索》这本书，不知道有多少人会在运行代码学习一遍，当我浏览 git 代码时，发现项目都是 eclipse 时代的，为了方便他人学习，我统一进行了转换，并进行了部分修复工作，如下：

1. 更新所有项目为 AndroidStudio gradle 形式
2. nineold 动画库现在已经用不到了，因为基本所有 app 最低版本4.0，不在需要这个兼容库了，但是为了纪念历史，仍然引入了
3. notification 的示例有问题，已经更新

我在公司17年的插件化项目中的很多思路也是从这里学习到的，致敬《Android 开发艺术探索》,原地址https://github.com/singwhatiwanna/android-art-res，修复之后的代码地址 https://github.com/guofeng007/android-art-res-new

