---
layout: post
title: ActivityResult 的一切
categories: Blog
description: ActivityResult 的一切
keywords:      ActivityResult 的一切

---

# ActivityResult 的一切

 Android 中 Activity 直接通信的方式一般是  
 1. A.startActivityForResult(B)
 2. B 在 finish()之前调用 setResult()存入结果,关闭 B 页面
 3. 回到 A 页面之后，在 onActivityResult 中接收结果

 这种典型的通信方式相信大家早就习以为常了，但是本文要说几个你可能没有注意到的点

## 1.这种方式能够稳定跨进程通信

是的，你没有听过，一般跨进程通信都是 AIDL，Service。但是在国内各种乱七八糟的 ROM 厂商系统中，很多都是把跨进程唤起 service,content provider,receiver 给禁止掉了。
 
为什么这么做？因为国内没有 GooglePlay 之类的唯一应用分发平台，导致市面上各种乱七八糟，各大 APP为了包活，经常
是你唤起我，我唤起你，这种唤起肯定是跨进程的。Rom 厂商肯定不愿意看到自己的系统被搞的乌烟瘴气，所以索性把各种非必要的冷唤起给禁用了。（PS:在华为手机上，冷唤起 Service 就能看到 log 里面的提示，之前小米手机的某个版本开始把唤起支付宝的服务给禁用调了，导致当时一大片用户无法支付，可怕，流氓），后来是小米下发了白名单，此事才作罢。

那支付宝以后会不会每天都胆战心惊呢？除了小米，国内 OV,一大堆国产收集2以后也会不会这么干！！！索性支付宝进行了一次升级（不要问我为什么对这个事情这么关注，我也是做某 BAT 支付的，我们的支付唤起也被禁用了）。支付宝升级的方式就是通过 startActivity 唤起支付（微信很机智，从一开始就是这么干的，不得不佩服微信支付团队）。

那么又有人要问了，会不会 Romc厂商以后把跨进程 startActivity 也禁用呢？  

讲了一大堆历史，也告诫我们，用最简单的方式去做，往往是最稳妥的。

## 2.什么时候调用 setResult

我们可能随手在某个地方写了 setResult，但是有没有想过，什么时候把 result 回调给上一个页面呢？我们调用了就会回调给上一个页面吗?二话不说，扔一坨代码如下：
```java
public final void setResult(int resultCode) {
        synchronized (this) {
            mResultCode = resultCode;
            mResultData = null;
        }
    }
```
我们的 setResult 只是把数据保存到 mResultCode,mResultData，所以你只要在 finish 之前，可以任意次调用 setResult()

那么问题来了，到底是什么时候回调给上一个页面呢？在扔一坨代码：
```java
private void finish(int finishTask) {
        if (mParent == null) {
            int resultCode;
            Intent resultData;
            synchronized (this) {
                resultCode = mResultCode;
                resultData = mResultData;
            }
            if (false) Log.v(TAG, "Finishing self: token=" + mToken);
            try {
                if (resultData != null) {
                    resultData.prepareToLeaveProcess(this);
                }
                if (ActivityManager.getService()
                        .finishActivity(mToken, resultCode, resultData, finishTask)) {
                    mFinished = true;
                }
            } catch (RemoteException e) {
                // Empty
            }
        } else {
            mParent.finishFromChild(this);
        }
    }

```

看到了吧，在调用 finish 的时候，会把 resultCode,resultData 通过 AMS回调给唤起页面。

## 3.onActivityResult 的封装
参考我另外一个[文章](https://guofeng007.github.io/2018/01/05/onActivityResult/)，封装成链式调用,感受下面这一坨代码[github](https://github.com/guofeng007/AvoidOnResult)：
```java
AvoidOnResult(this).startForResult(FetchDataActivity::class.java, object : AvoidOnResult.Callback {
                override fun onActivityResult(resultCode: Int, data: Intent?) =
                        if (resultCode == Activity.RESULT_OK) {
                            val text = data?.getStringExtra("text")
                            Toast.makeText(this@MainActivity, "callback -> " + text, Toast.LENGTH_SHORT).show()
                        } else {
                            Toast.makeText(this@MainActivity, "callback canceled", Toast.LENGTH_SHORT).show()
                        }

            })

```