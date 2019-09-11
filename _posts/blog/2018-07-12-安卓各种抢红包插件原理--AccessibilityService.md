---
layout: post
title: 安卓各种抢红包插件原理--AccessibilityService
categories: Blog
description:  安卓各种抢红包插件原理--AccessibilityService
keywords:      安卓各种抢红包插件原理--AccessibilityService

---


[转载原文](https://xudeveloper.github.io/2018/06/18/Android%20AccessibilityService%E6%9C%BA%E5%88%B6%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/)
[github 代码地址](https://github.com/guofeng007/AccessibilityServiceDemo)


#### 一、本文需要解决的问题

之前本人做了一个项目，需要用到AccessibilityService这个系统提供的拓展服务。这个服务本意是作为Android系统的一个辅助功能，去帮助残疾人更好地使用手机。但是由于它的一些特性，给很多项目的实现提供了一个新的思路，例如之前大名鼎鼎的微信抢红包插件，本质上就是使用了这个服务。我研究AccessibilityService的目的是解决以下几个我在使用过程中所思考的问题：

1. AccessibilityService这个Service跟一般的Service有什么区别？
2. AccessibilityService是如何做到监控并捕捉用户行为的？
3. AccessibilityService是如何做到查找控件，执行点击等操作的？

#### 二、初步分析

本文基于Android 7.1的源码对AccessibilityService进行分析。
为了更好地理解和分析代码，我写了一个demo，如果想学习具体的使用方法，可以参考Google官方文档[AccessibilityService](https://developer.android.com/reference/android/accessibilityservice/AccessibilityService.html)。本文不做AccessibilityService的具体使用教程。

##### 创建AccessibilityService

```
public class MyAccessibilityService extends AccessibilityService {

    private static final String TAG = "MyAccessibilityService";

    @Override
    public void onCreate() {
        super.onCreate();
        Log.i(TAG, "onCreate");
    }

    @Override
    public void onAccessibilityEvent(AccessibilityEvent event) {
        int eventType = event.getEventType();
        switch (eventType) {
            case AccessibilityEvent.TYPE_VIEW_CLICKED:
                // 捕获到点击事件
                Log.i(TAG, "capture click event!");
                AccessibilityNodeInfo nodeInfo = getRootInActiveWindow();
                if (nodeInfo != null) {
                    // 查找text为Test!的控件
                    List<AccessibilityNodeInfo> button = nodeInfo.findAccessibilityNodeInfosByText("Test!");
                    nodeInfo.recycle();
                    for (AccessibilityNodeInfo item : button) {
                        Log.i(TAG, "long-click button!");
                        // 执行长按操作
                        item.performAction(AccessibilityNodeInfo.ACTION_LONG_CLICK);
                    }
                }
                break;
            default:
                break;
        }
    }

    @Override
    public void onInterrupt() {
        Log.i(TAG, "onInterrupt");
    }
}
```

##### AccessibilityService配置

res/xml/accessibility_service_config.xml

```
<?xml version="1.0" encoding="utf-8"?>
<accessibility-service xmlns:android="http://schemas.android.com/apk/res/android"
    android:accessibilityEventTypes="typeAllMask"
    android:accessibilityFeedbackType="feedbackSpoken"
    android:accessibilityFlags="flagRetrieveInteractiveWindows|flagRequestFilterKeyEvents"
    android:canRequestFilterKeyEvents="true"
    android:canRetrieveWindowContent="true"
    android:description="@string/app_name"
    android:notificationTimeout="100"
    android:packageNames="com.xu.accessibilitydemo" />
```

##### 在manifest中进行注册

```
<service
    android:name=".MyAccessibilityService"
    android:enabled="true"
    android:exported="true"
    android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE">
    <intent-filter>
        <action android:name="android.accessibilityservice.AccessibilityService"/>
    </intent-filter>

     <meta-data
         android:name="android.accessibilityservice"
         android:resource="@xml/accessibility_service_config"/>
</service>
```

##### 创建一个text为Test!的button控件，设置监听方法

```
public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";

    private Button button;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        button = findViewById(R.id.button);

        button.setOnLongClickListener(new View.OnLongClickListener() {
            @Override
            public boolean onLongClick(View v) {
                Log.i(TAG, "onLongClick");
                return false;
            }
        });

    }
}
```

##### 开启AccessibilityService

开启AccessibilityService有两种方法，一种是在代码中开启，另一种是手动开启，具体开启位置为**设置–无障碍**中开启。

##### 运行应用，点击text为Test!的按钮

会出现以下的日志：
[![log.png](https://upload-images.jianshu.io/upload_images/1963233-23d8f1dfce6d7009.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)](https://upload-images.jianshu.io/upload_images/1963233-23d8f1dfce6d7009.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

**具体解释：**
点击按钮即产生TYPE_VIEW_CLICKED事件 –> 被AcceesibilityService捕获 –> 捕获后执行长按按钮操作 –> 执行长按回调方法。

为什么AcceesibilityService能捕获并执行其他操作呢，接下来我将对源码进行解析~

#### 三、源码解析

###### AccessibilityService内部逻辑

###### AccessibilityService.java

```
public abstract class AccessibilityService extends Service {
      // 省略代码
      public abstract void onAccessibilityEvent(AccessibilityEvent event);
      
      public abstract void onInterrupt();

      @Override
      public final IBinder onBind(Intent intent) {
          return new IAccessibilityServiceClientWrapper(this, getMainLooper(), new Callbacks() {
              @Override
              public void onServiceConnected() {
                  AccessibilityService.this.dispatchServiceConnected();
              }

              @Override
              public void onInterrupt() {
                  AccessibilityService.this.onInterrupt();
              }

              @Override
              public void onAccessibilityEvent(AccessibilityEvent event) {
                  AccessibilityService.this.onAccessibilityEvent(event);
              }

              @Override
              public void init(int connectionId, IBinder windowToken) {
                  mConnectionId = connectionId;
                  mWindowToken = windowToken;

                  // The client may have already obtained the window manager, so
                  // update the default token on whatever manager we gave them.
                  final WindowManagerImpl wm = (WindowManagerImpl) getSystemService(WINDOW_SERVICE);
                  wm.setDefaultToken(windowToken);
              }

              @Override
              public boolean onGesture(int gestureId) {
                  return AccessibilityService.this.onGesture(gestureId);
              }

              @Override
              public boolean onKeyEvent(KeyEvent event) {
                  return AccessibilityService.this.onKeyEvent(event);
              }

              @Override
              public void onMagnificationChanged(@NonNull Region region,
                      float scale, float centerX, float centerY) {
                  AccessibilityService.this.onMagnificationChanged(region, scale, centerX, centerY);
              }

              @Override
              public void onSoftKeyboardShowModeChanged(int showMode) {
                  AccessibilityService.this.onSoftKeyboardShowModeChanged(showMode);
              }

              @Override
              public void onPerformGestureResult(int sequence, boolean completedSuccessfully) {
                  AccessibilityService.this.onPerformGestureResult(sequence, completedSuccessfully);
              }
          });
      }
}
```

分析：

1. AccessibilityService是一个抽象类，继承于Service，提供两个抽象方法 onAccessibilityEvent() 和 onInterrupt()；
2. 虽然是抽象类，但是实现了最重要的 onBind() 方法，在其中创建了一个IAccessibilityServiceClientWrapper对象，实现Callbacks接口中的抽象方法。

###### IAccessibilityServiceClientWrapper

```
// 以分析onAccessibilityEvent为例，省略部分代码
public static class IAccessibilityServiceClientWrapper extends IAccessibilityServiceClient.Stub
            implements HandlerCaller.Callback {
    private final HandlerCaller mCaller;
    private final Callbacks mCallback;
    private int mConnectionId;
    
    public IAccessibilityServiceClientWrapper(Context context, Looper looper,
                Callbacks callback) {
        mCallback = callback;
        mCaller = new HandlerCaller(context, looper, this, true /*asyncHandler*/);
    }
    
    public void init(IAccessibilityServiceConnection connection, int connectionId,
                IBinder windowToken) {
        Message message = mCaller.obtainMessageIOO(DO_INIT, connectionId,
                    connection, windowToken);
        mCaller.sendMessage(message);
    }
    
    // 省略部分代码 

    public void onAccessibilityEvent(AccessibilityEvent event) {
        Message message = mCaller.obtainMessageO(DO_ON_ACCESSIBILITY_EVENT, event);
        mCaller.sendMessage(message);
    }

    @Override
    public void executeMessage(Message message) {
        switch (message.what) {
            case DO_ON_ACCESSIBILITY_EVENT: {
                AccessibilityEvent event = (AccessibilityEvent) message.obj;
                if (event != null) {
                    AccessibilityInteractionClient.getInstance().onAccessibilityEvent(event);
                    mCallback.onAccessibilityEvent(event);
                    // Make sure the event is recycled.
                    try {
                        event.recycle();
                    } catch (IllegalStateException ise) {
                        /* ignore - best effort */
                    }
                }
            } return;
            // ...         
        }
     }
}
```

分析：

1. IAccessibilityServiceClientWrapper继承于IAccessibilityServiceClient类，它是一个aidl接口，同时注意到它是继承于IAccessibilityServiceClient.Stub类，可以大概猜测到，AccessibilityService为一个远程Service，使用到跨进程通信技术，后面我还会继续分析这个；

2. IAccessibilityServiceClientWrapper的类构造方法中，有两个比较重要的参数，一个是looper，另一个是Callbacks callback。Looper不用说，而Callbacks接口定义了很多方法，代码如下：

   ```
   public interface Callbacks {
       public void onAccessibilityEvent(AccessibilityEvent event);
       public void onInterrupt();
       public void onServiceConnected();
       public void init(int connectionId, IBinder windowToken);
       public boolean onGesture(int gestureId);
       public boolean onKeyEvent(KeyEvent event);
       public void onMagnificationChanged(@NonNull Region region,
                   float scale, float centerX, float centerY);
       public void onSoftKeyboardShowModeChanged(int showMode);
       public void onPerformGestureResult(int sequence, boolean completedSuccessfully);
   }
   ```

3. IAccessibilityServiceClientWrapper同时也实现了HandlerCaller.Callback接口，HandlerCaller类通过命名也可以知道，它内部含有一个Handler实例，所以可以把它当做一个Handler，而处理信息的方法就是HandlerCaller.Callback#executeMessage(msg)方法

4. 代码有点绕，故简单总结一下流程：
   AccessibilityEvent产生
     -> Binder驱动
      -> IAccessibilityServiceClientWrapper#onAccessibilityEvent(AccessibilityEvent)
   ​    -> HandlerCaller#sendMessage(message); // message中包括AccessibilityEvent
   ​     -> IAccessibilityServiceClientWrapper#executeMessage();
   ​      -> Callbacks#onAccessibilityEvent(event);
   ​       -> AccessibilityService.this.onAccessibilityEvent(event);

到这里解决了我们的第一个问题：**AccessibilityService同样继承于Service类，它属于远程服务类，是Android系统提供的一种服务，可以绑定此服务，用于捕捉界面的一些特定事件。**

###### AccessibilityService外部逻辑

前面分析了接收到AccessibilityEvent之后的代码逻辑，那么，这些AccessibilityEvent是怎样产生的呢，而且，在回调执行之后是怎么做到点击等操作的（如demo所示）？我们接下来继续分析相关的源码~

我们从demo作为例子开始入手，首先我们知道，一个点击事件的产生，实际代码逻辑是在View#onTouchEvent() -> View#performClick()中：

```
public boolean performClick() {
    final boolean result;
    final ListenerInfo li = mListenerInfo;
    if (li != null && li.mOnClickListener != null) {
        playSoundEffect(SoundEffectConstants.CLICK);
        li.mOnClickListener.onClick(this);
        result = true;
    } else {
        result = false;
    }
    // ！！！
    sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);
    return result;
}
```

这里找到一个重点方法sendAccessibilityEvent()，继续跟进去，最后走到View#sendAccessibilityEventUncheckedInternal()方法：

```
public void sendAccessibilityEventUncheckedInternal(AccessibilityEvent event) {
    if (!isShown()) {
        return;
    }
    onInitializeAccessibilityEvent(event);
    // Only a subset of accessibility events populates text content.
    if ((event.getEventType() & POPULATING_ACCESSIBILITY_EVENT_TYPES) != 0) {
        dispatchPopulateAccessibilityEvent(event);
    }
    // In the beginning we called #isShown(), so we know that getParent() is not null.
    getParent().requestSendAccessibilityEvent(this, event);
}
```

这里的getParent()会返回一个实现ViewParent接口的对象。
我们可以简单理解为，它会让View的父类执行requestSendAccessibilityEvent()方法，而View的父类一般为ViewGroup，我们查看ViewGroup#requestSendAccessibilityEvent()方法

```
@Override
public boolean requestSendAccessibilityEvent(View child, AccessibilityEvent event) {
    ViewParent parent = mParent;
    if (parent == null) {
        return false;
    }
    final boolean propagate = onRequestSendAccessibilityEvent(child, event);
    if (!propagate) {
        return false;
    }
    return parent.requestSendAccessibilityEvent(this, event);
}
```

这里涉及到一个变量mParent，我们要找到这个mParent变量是在哪里被赋值的。
首先我们在View类中找到一个相关的方法View#assignParent()：

```
void assignParent(ViewParent parent) {
    if (mParent == null) {
        mParent = parent;
    } else if (parent == null) {
        mParent = null;
    } else {
        throw new RuntimeException("view " + this + " being added, but" + " it already has a parent");
    }
}
```

但是View类中并没有调用此方法，猜测是View的父类进行调用。
通过对源码进行搜索，发现最后是在ViewRootImpl#setView()中进行调用，赋值的是this即ViewRootImpl本身。
直接跳到ViewRootImpl#requestSendAccessibilityEvent()方法：

```
@Override
public boolean requestSendAccessibilityEvent(View child, AccessibilityEvent event) {
    if (mView == null || mStopped || mPausedForTransition) {
        return false;
    }
    // Intercept accessibility focus events fired by virtual nodes to keep
    // track of accessibility focus position in such nodes.
    final int eventType = event.getEventType();
    switch (eventType) {
        case AccessibilityEvent.TYPE_VIEW_ACCESSIBILITY_FOCUSED:
            {
                final long sourceNodeId = event.getSourceNodeId();
                final int accessibilityViewId = AccessibilityNodeInfo.getAccessibilityViewId(sourceNodeId);
                View source = mView.findViewByAccessibilityId(accessibilityViewId);
                if (source != null) {
                    AccessibilityNodeProvider provider = source.getAccessibilityNodeProvider();
                    if (provider != null) {
                        final int virtualNodeId = AccessibilityNodeInfo.getVirtualDescendantId(sourceNodeId);
                        final AccessibilityNodeInfo node;
                        if (virtualNodeId == AccessibilityNodeInfo.UNDEFINED_ITEM_ID) {
                            node = provider.createAccessibilityNodeInfo(AccessibilityNodeProvider.HOST_VIEW_ID);
                        } else {
                            node = provider.createAccessibilityNodeInfo(virtualNodeId);
                        }
                        setAccessibilityFocus(source, node);
                    }
                }
            }
            break;
            // 省略部分代码
    }
    // ！！！
    mAccessibilityManager.sendAccessibilityEvent(event);
    return true;
}
```

重点：AccessibilityManager#sendAccessibilityEvent(event)

```
public void sendAccessibilityEvent(AccessibilityEvent event) {
    final IAccessibilityManager service;
    final int userId;
    synchronized(mLock) {
        service = getServiceLocked();
        if (service == null) {
            return;
        }
        if (!mIsEnabled) {
            Looper myLooper = Looper.myLooper();
            if (myLooper == Looper.getMainLooper()) {
                throw new IllegalStateException("Accessibility off. Did you forget to check that?");
            } else {
                // If we're not running on the thread with the main looper, it's possible for
                // the state of accessibility to change between checking isEnabled and
                // calling this method. So just log the error rather than throwing the
                // exception.
                Log.e(LOG_TAG, "AccessibilityEvent sent with accessibility disabled");
                return;
            }
        }
        userId = mUserId;
    }
    boolean doRecycle = false;
    try {
        event.setEventTime(SystemClock.uptimeMillis());
        // it is possible that this manager is in the same process as the service but
        // client using it is called through Binder from another process. Example: MMS
        // app adds a SMS notification and the NotificationManagerService calls this method
        long identityToken = Binder.clearCallingIdentity();
        // ！！！
        doRecycle = service.sendAccessibilityEvent(event, userId);
        Binder.restoreCallingIdentity(identityToken);
        if (DEBUG) {
            Log.i(LOG_TAG, event + " sent");
        }
    } catch (RemoteException re) {
        Log.e(LOG_TAG, "Error during sending " + event + " ", re);
    } finally {
        if (doRecycle) {
            event.recycle();
        }
    }
}

private IAccessibilityManager getServiceLocked() {
    if (mService == null) {
        tryConnectToServiceLocked(null);
    }
    return mService;
}

private void tryConnectToServiceLocked(IAccessibilityManager service) {
    if (service == null) {
        IBinder iBinder = ServiceManager.getService(Context.ACCESSIBILITY_SERVICE);
        if (iBinder == null) {
            return;
        }
        service = IAccessibilityManager.Stub.asInterface(iBinder);
    }
    try {
        final int stateFlags = service.addClient(mClient, mUserId);
        setStateLocked(stateFlags);
        mService = service;
    } catch (RemoteException re) {
        Log.e(LOG_TAG, "AccessibilityManagerService is dead", re);
    }
}
```

这里有使用到Android Binder机制，重点为IAccessibilityManager#sendAccessibilityEvent()方法，这里调用的是代理方法，实际代码逻辑在AccessibilityManagerService#sendAccessibilityEvent()：

```
@Override
public boolean sendAccessibilityEvent(AccessibilityEvent event, int userId) {
    synchronized(mLock) {
        // We treat calls from a profile as if made by its parent as profiles
        // share the accessibility state of the parent. The call below
        // performs the current profile parent resolution..
        final int resolvedUserId = mSecurityPolicy.resolveCallingUserIdEnforcingPermissionsLocked(userId);
        // This method does nothing for a background user.
        if (resolvedUserId != mCurrentUserId) {
            return true; // yes, recycle the event
        }
        if (mSecurityPolicy.canDispatchAccessibilityEventLocked(event)) {
            mSecurityPolicy.updateActiveAndAccessibilityFocusedWindowLocked(event.getWindowId(), event.getSourceNodeId(), event.getEventType(), event.getAction());
            mSecurityPolicy.updateEventSourceLocked(event);
            // ！！！
            notifyAccessibilityServicesDelayedLocked(event, false);
            notifyAccessibilityServicesDelayedLocked(event, true);
        }
        if (mHasInputFilter && mInputFilter != null) {
            mMainHandler.obtainMessage(MainHandler.MSG_SEND_ACCESSIBILITY_EVENT_TO_INPUT_FILTER, AccessibilityEvent.obtain(event)).sendToTarget();
        }
        event.recycle();
    }
    return (OWN_PROCESS_ID != Binder.getCallingPid());
}

private void notifyAccessibilityServicesDelayedLocked(AccessibilityEvent event, boolean isDefault) {
    try {
        UserState state = getCurrentUserStateLocked();
        for (int i = 0, count = state.mBoundServices.size(); i < count; i++) {
            Service service = state.mBoundServices.get(i);
            if (service.mIsDefault == isDefault) {
                if (canDispatchEventToServiceLocked(service, event)) {
                    service.notifyAccessibilityEvent(event);
                }
            }
        }
    } catch (IndexOutOfBoundsException oobe) {
        // An out of bounds exception can happen if services are going away
        // as the for loop is running. If that happens, just bail because
        // there are no more services to notify.
    }
}
```

1. 在方法中，最后会调用notifyAccessibilityServicesDelayedLocked()方法，然后将event进行回收；
2. 在notifyAccessibilityServicesDelayedLocked()方法中，会获得所有Bound即绑定的Service，执行notifyAccessibilityEvent()方法，通过跟踪代码逻辑，最后会调用绑定Service的onAccessibilityEvent()方法。绑定的Service是指我们自己实现的继承于AccessibilityService的Service类，当你在设置-无障碍中开启服务之后即将服务绑定到AccessibilityManagerService中。

这样我们解决了第二个问题：
**AccessibilityService是如何做到监控捕捉用户行为的：（以点击事件为例）**
**AccessibilityEvent产生：**
View#performClick()
  -> View#sendAccessibilityEventUncheckedInternal()
   -> ViewGroup#requestSendAccessibilityEvent()
​    -> ViewRootImpl#requestSendAccessibilityEvent()
​     -> AccessibilityManager#sendAccessibilityEvent(event)
​      -> AccessibilityManagerService#sendAccessibilityEvent()
​       -> AccessibilityManagerService#notifyAccessibilityServicesDelayedLocked()
​        -> Service#notifyAccessibilityEvent(event)

**AccessibilityEvent处理：**
AccessibilityEvent
  -> Binder驱动
   -> IAccessibilityServiceClientWrapper#onAccessibilityEvent(AccessibilityEvent)
​    -> HandlerCaller#sendMessage(message); // message中包括AccessibilityEvent
​     -> IAccessibilityServiceClientWrapper#executeMessage();
​      -> Callbacks#onAccessibilityEvent(event);
​       -> AccessibilityService.this.onAccessibilityEvent(event);

###### AccessibilityService交互之查找控件

在demo中，我们在MyAccessibilityService中调用了getRootInActiveWindow()方法获取被监控的View的所有结点，这些结点都封装成一个AccessibilityNodeInfo对象中。同时也调用AccessibilityNodeInfo#findAccessibilityNodeInfosByText()方法查找相应的控件。
这些方法的本质是调用了AccessibilityInteractionClient类的对应方法。
以AccessibilityInteractionClient#findAccessibilityNodeInfosByText()为例：

```
public List<AccessibilityNodeInfo> findAccessibilityNodeInfosByText(int connectionId, int accessibilityWindowId, long accessibilityNodeId, String text) {
    try {
        IAccessibilityServiceConnection connection = getConnection(connectionId);
        if (connection != null) {
            final int interactionId = mInteractionIdCounter.getAndIncrement();
            final long identityToken = Binder.clearCallingIdentity();
            final boolean success = connection.findAccessibilityNodeInfosByText(accessibilityWindowId, accessibilityNodeId, text, interactionId, this, Thread.currentThread().getId());
            Binder.restoreCallingIdentity(identityToken);
            if (success) {
                List<AccessibilityNodeInfo> infos = getFindAccessibilityNodeInfosResultAndClear(interactionId);
                if (infos != null) {
                    finalizeAndCacheAccessibilityNodeInfos(infos, connectionId);
                    return infos;
                }
            }
        } else {
            if (DEBUG) {
                Log.w(LOG_TAG, "No connection for connection id: " + connectionId);
            }
        }
    } catch (RemoteException re) {
        Log.w(LOG_TAG, "Error while calling remote" + " findAccessibilityNodeInfosByViewText", re);
    }
    return Collections.emptyList();
}
```

代码逻辑比较简单，就是直接调用IAccessibilityServiceConnection#findAccessibilityNodeInfosByText()方法。
IAccessibilityServiceConnection是一个aidl接口，从注释看，它是AccessibilitySerivce和AccessibilityManagerService之间沟通的桥梁。
猜想代码真正的实现在AccessibilityManagerService中。
AccessibilityManagerService.Service#findAccessibilityNodeInfosByText()：

```
@Override
public boolean findAccessibilityNodeInfosByText(int accessibilityWindowId, long accessibilityNodeId, String text, int interactionId, IAccessibilityInteractionConnectionCallback callback, long interrogatingTid) throws RemoteException {
    final int resolvedWindowId;
    IAccessibilityInteractionConnection connection = null;
    Region partialInteractiveRegion = Region.obtain();
    synchronized(mLock) {
        if (!isCalledForCurrentUserLocked()) {
            return false;
        }
        resolvedWindowId = resolveAccessibilityWindowIdLocked(accessibilityWindowId);
        final boolean permissionGranted = mSecurityPolicy.canGetAccessibilityNodeInfoLocked(this, resolvedWindowId);
        if (!permissionGranted) {
            return false;
        } else {
            connection = getConnectionLocked(resolvedWindowId);
            if (connection == null) {
                return false;
            }
        }
        if (!mSecurityPolicy.computePartialInteractiveRegionForWindowLocked(resolvedWindowId, partialInteractiveRegion)) {
            partialInteractiveRegion.recycle();
            partialInteractiveRegion = null;
        }
    }
    final int interrogatingPid = Binder.getCallingPid();
    final long identityToken = Binder.clearCallingIdentity();
    MagnificationSpec spec = getCompatibleMagnificationSpecLocked(resolvedWindowId);
    try {
        connection.findAccessibilityNodeInfosByText(accessibilityNodeId, text, partialInteractiveRegion, interactionId, callback, mFetchFlags, interrogatingPid, interrogatingTid, spec);
        return true;
    } catch (RemoteException re) {
        if (DEBUG) {
            Slog.e(LOG_TAG, "Error calling findAccessibilityNodeInfosByText()");
        }
    } finally {
        Binder.restoreCallingIdentity(identityToken);
        // Recycle if passed to another process.
        if (partialInteractiveRegion != null && Binder.isProxy(connection)) {
            partialInteractiveRegion.recycle();
        }
    }
    return false;
}
```

1. 此方法在AccessibilityManagerService的内部类Service中实现，这个Service继承于IAccessibilityServiceConnection.Stub，验证了我上面的猜想是正确的；

2. 代码重点是调用connection.findAccessibilityNodeInfosByText()，这里的connection实例与上面不同，它隶属于IAccessibilityInteractionConnection类。这个类同样是一个aidl接口，从注释上看，它又是AccessibilityManagerService与指定窗口的ViewRoot之间沟通的桥梁。
   再次猜想，真正的代码逻辑在ViewRootImpl中。
   查看ViewRootImpl.AccessibilityInteractionConnection#findAccessibilityNodeInfosByText()：

   ```
   @Override
   public void findAccessibilityNodeInfosByText(long accessibilityNodeId, String text, Region interactiveRegion, int interactionId, IAccessibilityInteractionConnectionCallback callback, int flags, int interrogatingPid, long interrogatingTid, MagnificationSpec spec) {
       ViewRootImpl viewRootImpl = mViewRootImpl.get();
       if (viewRootImpl != null && viewRootImpl.mView != null) {
           viewRootImpl.getAccessibilityInteractionController().findAccessibilityNodeInfosByTextClientThread(accessibilityNodeId, text, interactiveRegion, interactionId, callback, flags, interrogatingPid, interrogatingTid, spec);
       } else {
           // We cannot make the call and notify the caller so it does not wait.
           try {
               callback.setFindAccessibilityNodeInfosResult(null, interactionId);
           } catch (RemoteException re) {
               /* best effort - ignore */
           }
       }
   }
   ```

3. 同样的，此方法在ViewRootImpl的内部类AccessibilityInteractionConnection中实现，这个内部类继承于IAccessibilityServiceConnection.Stub，验证了我的猜想；

4. 查找控件等操作，ViewRootImpl并不是直接处理，而是交给AccessibilityInteractionController类去查找，查找到的结果会保存到一个callback中，这个callback为IAccessibilityInteractionConnectionCallback类型，它也是一个aidl接口，而AccessibilityInteractionClient类继承了IAccessibilityInteractionConnectionCallback.Stub，即最后查询后的结果会回调到AccessibilityInteractionClient类中，如上面AccessibilityInteractionClient#findAccessibilityNodeInfosByText()方法，最后会调用getFindAccessibilityNodeInfosResultAndClear()方法获取结果。具体如何寻找指定控件则不再分析代码。

###### AccessibilityService交互之执行控件操作

类似的，与上面的流程基本相同，只是回调的时候，返回的是执行操作的返回值（True or False）。

到这里，我们解决了最后一个问题：
**AccessibilityService是如何做到查找控件，执行点击等操作的？**
总结：
寻找指定控件/执行操作
  -> 交给AccessibilityInteractionClient类处理
​    -> Binder
​      -> AccessibilityManagerService类进行查找/执行操作
​        -> Binder
​          -> 指定窗口的ViewRoot（ViewRootImpl）进行查找/执行操作
​        <- Binder
​    <- 结果回调到AccessibilityInteractionClient类

#### 四、有用代码记录

1. HandlerCaller类：结合Handler类和自定义的接口类（Caller.java），利用Handler的消息循环机制来分发消息，将最终的处理函数交给Caller#executeMessage()：

   ```
   // HandlerCaller.java
   public class HandlerCaller {
       final Looper mMainLooper;
       final Handler mH;

       final Callback mCallback;

       class MyHandler extends Handler {
           MyHandler(Looper looper, boolean async) {
               super(looper, null, async);
           }

           @Override
           public void handleMessage(Message msg) {
               mCallback.executeMessage(msg);
           }
       }

       public interface Callback {
           public void executeMessage(Message msg);
       }

       public HandlerCaller(Context context, Looper looper, Callback callback,
               boolean asyncHandler) {
           mMainLooper = looper != null ? looper : context.getMainLooper();
           mH = new MyHandler(mMainLooper, asyncHandler);
           mCallback = callback;
       }

       public Handler getHandler() {
           return mH;
       }

       public void executeOrSendMessage(Message msg) {
           // If we are calling this from the main thread, then we can call
           // right through.  Otherwise, we need to send the message to the
           // main thread.
           if (Looper.myLooper() == mMainLooper) {
               mCallback.executeMessage(msg);
               msg.recycle();
               return;
           }
           
           mH.sendMessage(msg);
       }

       public void sendMessageDelayed(Message msg, long delayMillis) {
           mH.sendMessageDelayed(msg, delayMillis);
       }

       public boolean hasMessages(int what) {
           return mH.hasMessages(what);
       }
       
       public void removeMessages(int what) {
           mH.removeMessages(what);
       }
       
       public void removeMessages(int what, Object obj) {
           mH.removeMessages(what, obj);
       }
       
       public void sendMessage(Message msg) {
           mH.sendMessage(msg);
       }

       public SomeArgs sendMessageAndWait(Message msg) {
           if (Looper.myLooper() == mH.getLooper()) {
               throw new IllegalStateException("Can't wait on same thread as looper");
           }
           SomeArgs args = (SomeArgs)msg.obj;
           args.mWaitState = SomeArgs.WAIT_WAITING;
           mH.sendMessage(msg);
           synchronized (args) {
               while (args.mWaitState == SomeArgs.WAIT_WAITING) {
                   try {
                       args.wait();
                   } catch (InterruptedException e) {
                       return null;
                   }
               }
           }
           args.mWaitState = SomeArgs.WAIT_NONE;
           return args;
       }

       public Message obtainMessage(int what) {
           return mH.obtainMessage(what);
       }
        
       // 省略部分代码
   }
   ```

2. HandlerCaller#sendMessageAndWait()：

   ```
   public SomeArgs sendMessageAndWait(Message msg) {
       if (Looper.myLooper() == mH.getLooper()) {
           throw new IllegalStateException("Can't wait on same thread as looper");
       }
       SomeArgs args = (SomeArgs) msg.obj;
       args.mWaitState = SomeArgs.WAIT_WAITING;
       mH.sendMessage(msg);
       synchronized(args) {
           while (args.mWaitState == SomeArgs.WAIT_WAITING) {
               try {
                   args.wait();
               } catch (InterruptedException e) {
                   return null;
               }
           }
       }
       args.mWaitState = SomeArgs.WAIT_NONE;
       return args;
   }
   ```