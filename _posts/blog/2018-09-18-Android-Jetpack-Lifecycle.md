---
layout: post
title: Android 架构组件 LifeCycle 源码解析
categories: Blog
description: Android 架构组件 LifeCycle 源码解析
keywords:      Android 架构组件 LifeCycle 源码解析

---

# Android 架构组件 LifeCycle 源码解析



​    随着 Android系统的发展，碎片化也越来越严重，APP开发考虑的东西越来越多，如后台任务，用户操作，电量消耗，性能等等，给APP 的开发带来沉重的负担。不同的公司开发 APP 的技术水平不一样，导致整个 APP 应用市场上的产品质量参差不齐。为了保障 APP 的质量，Google推出了 Jetpack 组件，该组件从基础库、架构、行为、UI 四个大的方面入手，旨在统一 APP 开发框架，提升稳定性。整个组件库比较庞大，本文首先对 Jetpack 整体进行简介，然后着重分析架构模块中 LifeCycle 的实现原理，感兴趣的读者可以用类似的方法去分析其他组件，借以抛砖引玉。

# 1. Android Jetpack 简介

​    Jetpack是一系列库、工具、架构组成的，帮助开发人员快速方便的构建高效、稳定的Anroid App，使用该组件库可以：

\1.  轻松管理应用程序的生命周期

\2.  构建可观察的数据对象，以便在基础数据库更改时通知视图

\3.  存储在应用程序轮换中未销毁的UI相关数据，在界面重建后恢复数据

\4.  轻松的实现SQLite数据库

\5.  系统自动调度后台任务的执行，优化使用性能

​    其构成如下：

![/C:/0885f05e82cae96eaa692a8069d3d162](![bridge](/images/posts/lifecycle/clip_image001.png)

## 1.1 基础库Foundation

​    基础库提供向下兼容、测试、Multidex 分包以及 Kotlin 扩展支持

1. [AppCompact](https://developer.android.com/topic/libraries/support-library/packages.html#v7-appcompat)      : 提供各个版本的兼容 API，以及基础库，包含      v4，v7，v13等，随着版本的不断提升还会有更多的兼容库。
2. [Android KTX:](https://developer.android.com/kotlin/ktx.html) 提供对 Kotlin 语言相关的支持库，方便 Kotlin 开发
3. [Multidex](https://developer.android.com/studio/build/multidex.html):      通过 Dex 拆分，解决 App 包过大的问题
4. [Test](https://developer.android.com/topic/libraries/testing-support-library/index.html):       Android 单元测试和 UI 测试的库

## 1.2 架构组件 Architecture

​    架构组件的目的是让 APP 更加健壮、可测试、易维护，主要包含：

1. [Data      Binding:](https://developer.android.com/topic/libraries/data-binding/) 声明式数据绑定，即 MVVM 在 Android 平台的实现
2. [Lifecycles](https://developer.android.com/topic/libraries/architecture/lifecycle)：管理应用生命周期
3. [LiveData](https://developer.android.com/topic/libraries/architecture/livedata):      动态观察数据变化，感知界面生命周期，自动更新数据
4. [Navigation](https://developer.android.com/topic/libraries/architecture/navigation.html):      App 内部导航框架
5. [Paging](https://developer.android.com/topic/libraries/architecture/paging/):      优雅地按需加载数据
6. [Room](https://developer.android.com/topic/libraries/architecture/room):      SQLite ORM 对象关系映射
7. [ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel):      在可感知界面状态下，管理 UI 数据
8. [WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager):      统一管理后台任务

## 1.3 行为 Behavior

1. [Download      Manager](https://developer.android.com/reference/android/app/DownloadManager): 统一管理（定时、周期）下载
2. [Media      & playback](https://developer.android.com/guide/topics/media-apps/media-apps-overview.html) :提供兼容 API 处理多媒体播放、投射
3. [Notifications](https://developer.android.com/guide/topics/ui/notifiers/notifications.html):      提供兼容 API 处理通知，同时能够兼容手表和车载设备
4. [Permissions](https://developer.android.com/guide/topics/permissions/index.html):      权限检查和申请
5. [Sharing](https://developer.android.com/training/sharing/shareaction):      提供标题栏通用分享
6. [Slices](https://developer.android.com/guide/slices): 创建灵活的 UI 组件模块，在 APP 外展示数据（如 Google 搜索）
7. [Preferences](https://developer.android.com/guide/topics/ui/settings):      提供统一的偏好设置

## 1.4 界面 UI

1. [Animation &      transitions](https://developer.android.com/training/animation/): 在不同场景切换时，播放组件动画
2. [Auto](https://developer.android.com/auto): 开发车载工具箱
3. [Emoji](https://developer.android.com/guide/topics/ui/look-and-feel/emoji-compat):      为老系统提供最新的 Emoji 颜文字
4. [Fragment](https://developer.android.com/guide/components/fragments):      界面组件单元
5. [Layout](https://developer.android.com/guide/topics/ui/declaring-layout):      使用不同的算法进行界面布局
6. [Palette](https://developer.android.com/training/material/palette-colors):      从调色板中提取有用的信息
7. [TV](https://developer.android.com/tv) : TV开发工具箱
8. [Wear OS by Google](https://developer.android.com/wear): 手表开发工具箱

​    Android Jetpack 所包含的内容及其丰富，其源码是没办法在一篇文章中解释清除。因此在接下来的章节以 Lifecycles 组件为突破口，基于2.0.0版本进行分析，其他的组件可以顺藤摸瓜，依次研究。

# 2. 架构组件LifeCycles 原理解析

## 2.1 架构组件概览

​    第一部分对 Jetpack 进行整体介绍时，在1.2节中提到架构组件的构成，其使用方式如下图。

​    Lifecycle 已经默认包含在 support-compact / fragment.2.0.0版本就具备了 Lifecycle 的能力库中的 AppCompactActivity 、Fragment中，也就是下图中顶部的 Activity/Fragment。

## ![/C:/74c6f50adc7de0b3776f58d091a7925a](![bridge](/images/posts/lifecycle/clip_image002.png)

## 2.2 Lifecycles 使用方式

Lifecycle是一个生命周期感知组件，它通过类似@OnLifecycleEvent(Lifecycle.Event.ON_RESUME)的注解标注方法，使之可以监听UI组件生命周期的变化。

以往在生命周期中执行相应的方法需要设置接口，然后在声明周期中回调接口，造成了代码的耦合。

假如我们要在界面可见时监听 GPS 获取定位，在界面不可见时停止定位。

**在没有使用LifeCycle 时，我们监听Lifecycle的方式是这样的：**

```
class MyLocationListener {
    public MyLocationListener(Context context, Callback callback) {
        // ...
    }
 
    void start() {
        // connect to system location service
    }
 
    void stop() {
        // disconnect from system location service
    }
}
 
class MyActivity extends AppCompatActivity {
    private MyLocationListener myLocationListener;
 
    @Override
    public void onCreate(...) {
        myLocationListener = new MyLocationListener(this, (location) -> {
            // update UI
        });
    }
 
    @Override
    public void onStart() {
        super.onStart();
        myLocationListener.start();
        // manage other components that need to respond
        // to the activity lifecycle
    }
 
    @Override
    public void onStop() {
        super.onStop();
        myLocationListener.stop();
        // manage other components that need to respond
        // to the activity lifecycle
    }
}
```

这种方式有明显的缺陷，我们要在 Activity 的各种生命周期做不同的处理。如果我们还有其他的类似需求，这里将不断膨胀，难以维护，牵一发而动全身。

**Lifecycle****框架的使用方式**

```
public class MyObserver implements LifecycleObserver {
    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    public void connectListener() {
        ...
    }
 
    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    public void disconnectListener() {
        ...
    }
}
 
myLifecycleOwner.getLifecycle().addObserver(new MyObserver());
```

## 2.3 Lifecycle 原理

 Lifecycle 统一监听 UI 组件声明周期，并且允许其他组件监听自己，来实现生命周期感知。

其定义了 UI 组件的**状态**和对应的**事件**，类似状态机，通过UI 组件不同生命周期，触发Lifecycle的事件并更新状态，其状态变化如下：

![/C:/4a4ea796bc8071d76a5377bc601b47bc](![bridge](/images/posts/lifecycle/clip_image003.png)

其本质仍然是观察者模式，观察UI界面生命周期变化。

## 2.4 源码解析

我们先看一下 Lifecycle 的使用步骤：

1. 实现LifecycleObserver观察者
2. 编写执行方法
3. 在方法加上对应的事件注解
4. 获取 LifeOwner并添加观察者

按照使用步骤来分析实现原理，讲真个源码分为：

1. 状态和事件定义
2. 事件注解
3. 生命周期感知
4. 通知观察者

### 2.4.1 事件和状态定义

如下是对2.3节状态的定义，不同的状态和事件，对应着界面的变化，比较简单，直接看如下代码中状态的定义：

```
public abstract class Lifecycle {
  
    @MainThread
    public abstract void addObserver(@NonNull LifecycleObserver observer);
   
    @MainThread
    public abstract void removeObserver(@NonNull LifecycleObserver observer);
 
    
    @MainThread
    @NonNull
    public abstract State getCurrentState();
 
    @SuppressWarnings("WeakerAccess")
    public enum Event {        
        ON_CREATE,        
        ON_START,
        ON_RESUME,
        ON_PAUSE,
        ON_STOP,
        ON_DESTROY,
        ON_ANY
    }    
    @SuppressWarnings("WeakerAccess")
    public enum State {       
        DESTROYED,        
        INITIALIZED,
        CREATED,
        STARTED,
        RESUMED;
        public boolean isAtLeast(@NonNull State state) {
            return compareTo(state) >= 0;
        }
    }
}
```

### 2.4.2 注解的实现

```
@SuppressWarnings("unused")
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface OnLifecycleEvent {
    Lifecycle.Event value();
}
```

​    从代码@Retention(RetentionPolicy.RUNTIME)可以看出使用的是运行时注解（运行时注解反射是很耗时的，所以在CallbackInfo 中有对反射进行缓存），注解取值为事件枚举Lifecycle.Event。

注解是如何调用的？我们需要看一下观察者的继承结构

# ![/C:/7de455a52467a048929bdc1450fc5b57](![bridge](/images/posts/lifecycle/clip_image004.png)

基于注解的观察值使用了 CallbackInfo 来封装注解事件和对应的处理方法，其代码如下次：

```
private CallbackInfo createInfo(Class klass, @Nullable Method[] declaredMethods) {
    Class superclass = klass.getSuperclass();
    Map<MethodReference, Lifecycle.Event> handlerToEvent = new HashMap<>();
    if (superclass != null) {
        CallbackInfo superInfo = getInfo(superclass);
        if (superInfo != null) {
            handlerToEvent.putAll(superInfo.mHandlerToEvent);
        }
    }
 
    Class[] interfaces = klass.getInterfaces();
    for (Class intrfc : interfaces) {
        for (Map.Entry<MethodReference, Lifecycle.Event> entry : getInfo(
                intrfc).mHandlerToEvent.entrySet()) {
            verifyAndPutHandler(handlerToEvent, entry.getKey(), entry.getValue(), klass);
        }
    }
 
    Method[] methods = declaredMethods != null ? declaredMethods : getDeclaredMethods(klass);
    boolean hasLifecycleMethods = false;
    for (Method method : methods) {
        OnLifecycleEvent annotation = method.getAnnotation(OnLifecycleEvent.class);
        if (annotation == null) {
            continue;
        }
        hasLifecycleMethods = true;
        Class<?>[] params = method.getParameterTypes();
        int callType = CALL_TYPE_NO_ARG;
        if (params.length > 0) {
            callType = CALL_TYPE_PROVIDER;
            if (!params[0].isAssignableFrom(LifecycleOwner.class)) {
                throw new IllegalArgumentException(
                        "invalid parameter type. Must be one and instanceof LifecycleOwner");
            }
        }
        Lifecycle.Event event = annotation.value();
 
        if (params.length > 1) {
            callType = CALL_TYPE_PROVIDER_WITH_EVENT;
            if (!params[1].isAssignableFrom(Lifecycle.Event.class)) {
                throw new IllegalArgumentException(
                        "invalid parameter type. second arg must be an event");
            }
            if (event != Lifecycle.Event.ON_ANY) {
                throw new IllegalArgumentException(
                        "Second arg is supported only for ON_ANY value");
            }
        }
        if (params.length > 2) {
            throw new IllegalArgumentException("cannot have more than 2 params");
        }
        MethodReference methodReference = new MethodReference(callType, method);
        verifyAndPutHandler(handlerToEvent, methodReference, event, klass);
    }
    CallbackInfo info = new CallbackInfo(handlerToEvent);
    mCallbackMap.put(klass, info);
    mHasLifecycleMethods.put(klass, hasLifecycleMethods);
    return info;
}
```

createInfo 会依次解析 Class 并且将解析结果放入缓存：

private final Map<Class, CallbackInfo> mCallbackMap 缓存了所有观察者的信息
 private final Map<Class, Boolean> mHasLifecycleMethods 缓存了观察者是否有生命周期方法

final Map<Lifecycle.Event, List<MethodReference>> mEventToHandlers; 记录了对应事件的处理方法
 final Map<MethodReference, Lifecycle.Event> mHandlerToEvent; 记录了方法对应的处理事件

在注册观察者时，基于注解的会走到最后的分支，保存ReflectiveGenericLifecycleObserver，并通过 CallbackInfo记录上述注解信息，如下代码所示：

```
@NonNull
static GenericLifecycleObserver getCallback(Object object) {
    if (object instanceof FullLifecycleObserver) {
        return new FullLifecycleObserverAdapter((FullLifecycleObserver) object);
    }
 
    if (object instanceof GenericLifecycleObserver) {
        return (GenericLifecycleObserver) object;
    }
 
    final Class<?> klass = object.getClass();
    int type = getObserverConstructorType(klass);
    if (type == GENERATED_CALLBACK) {
        List<Constructor<? extends GeneratedAdapter>> constructors =
                sClassToAdapters.get(klass);
        if (constructors.size() == 1) {
            GeneratedAdapter generatedAdapter = createGeneratedAdapter(
                    constructors.get(0), object);
            return new SingleGeneratedAdapterObserver(generatedAdapter);
        }
        GeneratedAdapter[] adapters = new GeneratedAdapter[constructors.size()];
        for (int i = 0; i < constructors.size(); i++) {
            adapters[i] = createGeneratedAdapter(constructors.get(i), object);
        }
        return new CompositeGeneratedAdaptersObserver(adapters);
    }
    return new ReflectiveGenericLifecycleObserver(object);
}
```

 

### 2.4.3 生命周期感知

我们示例采用的是基于注解反射的观察者，从这里展开分析，首先看是如何注册的。注册需要获取 LifeOwner，然后获取 LifeCycle(只有唯一实现者LifecycleRegistry)，而这一切都应该在 AppCompactActivity 中，更具体一点是在父类ComponentActivity，代码如下：

```
@RestrictTo(LIBRARY_GROUP)
public class ComponentActivity extends Activity
        implements LifecycleOwner, KeyEventDispatcher.Component {
private LifecycleRegistry mLifecycleRegistry = new LifecycleRegistry(this);
 
@Override
@SuppressWarnings("RestrictedApi")
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    ReportFragment.injectIfNeededIn(this);
}
 
 
@Override
public Lifecycle getLifecycle() {
    return mLifecycleRegistry;
}
```

​    注意在 onCreate 中有一个ReportFragment.injectIfNeededIn(this)，注入了一个 Fragment

```
public class ReportFragment extends Fragment {
    private static final String REPORT_FRAGMENT_TAG = "androidx.lifecycle"
            + ".LifecycleDispatcher.report_fragment_tag";
 
    public static void injectIfNeededIn(Activity activity) {
        // ProcessLifecycleOwner should always correctly work and some activities may not extend
        // FragmentActivity from support lib, so we use framework fragments for activities
        android.app.FragmentManager manager = activity.getFragmentManager();
        if (manager.findFragmentByTag(REPORT_FRAGMENT_TAG) == null) {
            manager.beginTransaction().add(new ReportFragment(), REPORT_FRAGMENT_TAG).commit();
            // Hopefully, we are the first to make a transaction.
            manager.executePendingTransactions();
        }
    }
 
    static ReportFragment get(Activity activity) {
        return (ReportFragment) activity.getFragmentManager().findFragmentByTag(
                REPORT_FRAGMENT_TAG);
    }
 
    private ActivityInitializationListener mProcessListener;
 
    private void dispatchCreate(ActivityInitializationListener listener) {
        if (listener != null) {
            listener.onCreate();
        }
    }
 
    private void dispatchStart(ActivityInitializationListener listener) {
        if (listener != null) {
            listener.onStart();
        }
    }
 
    private void dispatchResume(ActivityInitializationListener listener) {
        if (listener != null) {
            listener.onResume();
        }
    }
 
    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        dispatchCreate(mProcessListener);
        dispatch(Lifecycle.Event.ON_CREATE);
    }
 
    @Override
    public void onStart() {
        super.onStart();
        dispatchStart(mProcessListener);
        dispatch(Lifecycle.Event.ON_START);
    }
 
    @Override
    public void onResume() {
        super.onResume();
        dispatchResume(mProcessListener);
        dispatch(Lifecycle.Event.ON_RESUME);
    }
 
    @Override
    public void onPause() {
        super.onPause();
        dispatch(Lifecycle.Event.ON_PAUSE);
    }
 
    @Override
    public void onStop() {
        super.onStop();
        dispatch(Lifecycle.Event.ON_STOP);
    }
 
    @Override
    public void onDestroy() {
        super.onDestroy();
        dispatch(Lifecycle.Event.ON_DESTROY);
        // just want to be sure that we won't leak reference to an activity
        mProcessListener = null;
    }
 
    private void dispatch(Lifecycle.Event event) {
        Activity activity = getActivity();
        if (activity instanceof LifecycleRegistryOwner) {
            ((LifecycleRegistryOwner) activity).getLifecycle().handleLifecycleEvent(event);
            return;
        }
 
        if (activity instanceof LifecycleOwner) {
            Lifecycle lifecycle = ((LifecycleOwner) activity).getLifecycle();
            if (lifecycle instanceof LifecycleRegistry) {
                ((LifecycleRegistry) lifecycle).handleLifecycleEvent(event);
            }
        }
    } 
}
```

原来是通过插入一个 Fragment,在各个生命周期回调中转发到 注册中心 LifecycleRegistry。

### 2.4.4 通知观察者

上一节提到 ReportFragment 会将各个生命周期转发给 LifecycleRegistry.handleLifecycleEvent,我们继续分析代码

```
public void handleLifecycleEvent(@NonNull Lifecycle.Event event) {
    State next = getStateAfter(event);
    moveToState(next);
}
 
private void moveToState(State next) {
    if (mState == next) {
        return;
    }
    mState = next;
    if (mHandlingEvent || mAddingObserverCounter != 0) {
        mNewEventOccurred = true;
        // we will figure out what to do on upper level.
        return;
    }
    mHandlingEvent = true;
    sync();
    mHandlingEvent = false;
}
@NonNull
@Override
public State getCurrentState() {
    return mState;
}
 
static State getStateAfter(Event event) {
    switch (event) {
        case ON_CREATE:
        case ON_STOP:
            return CREATED;
        case ON_START:
        case ON_PAUSE:
            return STARTED;
        case ON_RESUME:
            return RESUMED;
        case ON_DESTROY:
            return DESTROYED;
        case ON_ANY:
            break;
    }
    throw new IllegalArgumentException("Unexpected event value " + event);
}
 
private static Event downEvent(State state) {
    switch (state) {
        case INITIALIZED:
            throw new IllegalArgumentException();
        case CREATED:
            return ON_DESTROY;
        case STARTED:
            return ON_STOP;
        case RESUMED:
            return ON_PAUSE;
        case DESTROYED:
            throw new IllegalArgumentException();
    }
    throw new IllegalArgumentException("Unexpected state value " + state);
}
 
private static Event upEvent(State state) {
    switch (state) {
        case INITIALIZED:
        case DESTROYED:
            return ON_CREATE;
        case CREATED:
            return ON_START;
        case STARTED:
            return ON_RESUME;
        case RESUMED:
            throw new IllegalArgumentException();
    }
    throw new IllegalArgumentException("Unexpected state value " + state);
}
 
private void forwardPass(LifecycleOwner lifecycleOwner) {
    Iterator<Entry<LifecycleObserver, ObserverWithState>> ascendingIterator =
            mObserverMap.iteratorWithAdditions();
    while (ascendingIterator.hasNext() && !mNewEventOccurred) {
        Entry<LifecycleObserver, ObserverWithState> entry = ascendingIterator.next();
        ObserverWithState observer = entry.getValue();
        while ((observer.mState.compareTo(mState) < 0 && !mNewEventOccurred
                && mObserverMap.contains(entry.getKey()))) {
            pushParentState(observer.mState);
            observer.dispatchEvent(lifecycleOwner, upEvent(observer.mState));
            popParentState();
        }
    }
}
 
private void backwardPass(LifecycleOwner lifecycleOwner) {
    Iterator<Entry<LifecycleObserver, ObserverWithState>> descendingIterator =
            mObserverMap.descendingIterator();
    while (descendingIterator.hasNext() && !mNewEventOccurred) {
        Entry<LifecycleObserver, ObserverWithState> entry = descendingIterator.next();
        ObserverWithState observer = entry.getValue();
        while ((observer.mState.compareTo(mState) > 0 && !mNewEventOccurred
                && mObserverMap.contains(entry.getKey()))) {
            Event event = downEvent(observer.mState);
            pushParentState(getStateAfter(event));
            observer.dispatchEvent(lifecycleOwner, event);
            popParentState();
        }
    }
}
 
// happens only on the top of stack (never in reentrance),
// so it doesn't have to take in account parents
private void sync() {
    LifecycleOwner lifecycleOwner = mLifecycleOwner.get();
    if (lifecycleOwner == null) {
        Log.w(LOG_TAG, "LifecycleOwner is garbage collected, you shouldn't try dispatch "
                + "new events from it.");
        return;
    }
    while (!isSynced()) {
        mNewEventOccurred = false;
        // no need to check eldest for nullability, because isSynced does it for us.
        if (mState.compareTo(mObserverMap.eldest().getValue().mState) < 0) {
            backwardPass(lifecycleOwner);
        }
        Entry<LifecycleObserver, ObserverWithState> newest = mObserverMap.newest();
        if (!mNewEventOccurred && newest != null
                && mState.compareTo(newest.getValue().mState) > 0) {
            forwardPass(lifecycleOwner);
        }
    }
    mNewEventOccurred = false;
}
```

​    我们梳理一下顺序：

```
1.->handleLifecycleEvent()
2.->moveToState()
3.->sync()
4.->backwardPass()/forwardPass()
5.->observer.dispatchEvent(lifecycleOwner, event);
```

 

​     总结一下，就是按照状态切换流程，如下图所示，切换状态并通知生命周期。

![/C:/5c3d20ff2429c4edadb198fa6d194d69](![bridge](/images/posts/lifecycle/clip_image005.png)

# 2.5 结尾

​    APP 开发最佳实践并不是一蹴而就的，而是随着平台、业务的不断发展，一同向前演进。没有最好的架构，只有最适合的架构。Google 推出的 Jetpack 开发套件 内容及其丰富，本文只是以其中的 Lifecycle 组件为核心，分析介绍其实现原理，管中窥豹。对其他模块感兴趣的读者也可以参照本文的分析方式进行庖丁解牛。这些组件现在正在融入我们当前的项目中，哪些部分适合，那些水土不服，仍需要实践的检验。