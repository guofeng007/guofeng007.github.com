---
layout: post
title: Android 权限实践总结
categories: Blog
description:  Android权限实践总结
keywords:   Android , 权限实践总结

---

# Android 权限实践总结

##  前言

虽然Android M 23推出了动态权限申请，但是6.0之前的系统怎么办呢？特别是恶心的国产 ROM 各自为政，都搞了一套自己的权限。除此之外，当你的 targetSDKVersion < 23时，在 M 以上的手机是默认全部授予的，看起来没问题，但是用户居然可以手动取消。WTF........这一坨该怎么弄，本文综合对权限的理解与工程中的使用，给出如下指南，觉得有用的可以参考下。

## 相机权限



### 官方给的建议有两种：

1. ContextCompact.checkSelfPermission,最终是调用了 checkPermission(pid,uid,permission)，其实和 Context.checkSelfPermission 一样

2. PermissionChecker,结合 checkPermission(pid,uid,permission),

   同事使用AppOpsManagerCompat.permissionToOp(permission)和AppOpsManagerCompat.noteProxyOp(context, op, packageName, AppOpsManagerCompat.MODE_ALLOWED来检查，

   上述两个方案毫无例外，都跪了，在 targetsdkversion<23和6.0以下国产手机。

### 民间使用的方案：

   RxPermission,只是在外面套了一个 Fragment 来接受权限回调，这个看自己喜好了，本文忽略这个，专注于讲本质



### 结合PermissionChecker
```java

if(PermissionChecker.checkpermisssion() == GRANTED){

   try

   doSomething()j;

​    catch 

​    无权限提示

} else {

​    // 无权限

​    if build.sdk >=6.0

​          // 去申请，在回调中处理

​          requestpermisssion()

​    else 

​         // 尝试一下，激活国产权限

​          try

​               doSomething

​           catch

​               提示无权限，退出重新来

}

onRequestPermissionResult {

   // 有权限

​            doSomething

// 无权限，提示

}
```

但是 noteop 不准确啊，所以废弃

### 最稳妥的方案：官方原生
```java
if hasPermission() 
	tryit
else 
	requestpermisssion
```

```java

onRequestPermissionResult
	if true
		tryit
	else 
		showTips
```

```java
fuc hasPermission():
 if sdk<6.0 || sdk.target<23  
 	return true 
 else 
 	return checkSelfPermission()
```

# 结语

由于对权限判断都不准，所以尽量对未知情况返回 GRANT,让具体的 API 去 try 一下，国产手机可以触发权限弹框。如果 api 返回 Null，在弹框提示真的无权限或者被占用。

面对支离破碎的 android 机型，即碎片化，也只能这么『硬来了』。