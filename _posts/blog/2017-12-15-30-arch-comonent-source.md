---
layout: post
title: Android Architecture Component 源码解析
categories: Blog
description: Android Architecture Component 源码解析
keywords: Android Architecture Component 、源码、解析
---



#  Android Architecture Component 源码解析

 **作者：国风**

​        Architecture Components 是在 2017 年 Google I/O 大会上，推出的一个构建 Android 应用架构的库。旨在为已经掌握开发 APP 基础的人员在构建鲁棒，高质量的 APP 方面提供最佳实践。 

## 1. 如何使用
support-v7（版本大于26.1.0），会自动导入 ArchCore 和 LifeCycle
如果需要 LiveData 则需要加入如下两行：
```java
// support v7 26.1.0 contains lifecycle runtime

// living data viewmodel
implementation "android.arch.lifecycle:extensions:1.0.0"
annotationProcessor "android.arch.lifecycle:compiler:1.0.0"

// room
implementation "android.arch.persistence.room:runtime:1.0.0"
annotationProcessor "android.arch.persistence.room:compiler:1.0.0"
```

 

## 2. Lifecycle源码解析
### 2.1 LifeCycle 使用：
Lifecycle 观察者：

```java
class MyLocationListener implements LifecycleObserver {
    private static final String TAG = MyLocationListener.class.getSimpleName();
    private boolean enabled = false;
    private Context mContext ;
    private Lifecycle mLifecycle;
    private LocationListener mLocationListener;
    public MyLocationListener(Context context, Lifecycle lifecycle, LocationListener callback) {
        mContext = context;
        mLifecycle = lifecycle;
        mLocationListener = callback;
    }

    // wrong check state first
    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    void start() {
        if (enabled) {
           // connect
        }
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    public void enable() {
        if (mLifecycle.getCurrentState().isAtLeast(STARTED)) {
            enabled = true;
            Log.d(TAG,"enable success");
            new Thread(new Runnable() {
                @Override
                public void run() {
                    long randSleep = (long)Math.random()*5*1000+3000;
                    try {
                        Thread.sleep(randSleep);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    Location location = new Location("baidu");
                    location.setLongitude(22);
                    location.setAltitude(23);
                    if(mLocationListener != null){
                        // 防止destroy清空 mLocationListener
                        mLocationListener.onLocationChanged(location);
                    }

                }
            }).start();

        }
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    void stop() {
        // disconnect if connected
        Log.d(TAG,"stop success");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    void release() {
        // release
        mLifecycle.removeObserver(this);
        mContext = null;
        mLifecycle = null;
        mLocationListener  =null;

    }
}
```

被观察者 Activity

```java
public class LifecycleActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_common_btn);

    }
    public void onClick(View view) {
        switch (view.getId()) {
            case R.id.button:
                getLifecycle().addObserver(new MyLocationListener(this, getLifecycle(), new LocationListener() {
                    @Override
                    public void onLocationChanged(final Location location) {
                        runOnUiThread(new Runnable() {
                            @Override
                            public void run() {
                                Toast.makeText(LifecycleActivity.this, location.getProvider() + ":\t longtitude:" + location.getLongitude()
                                        + "\t :altitude:" + location.getAltitude(),Toast.LENGTH_SHORT).show();
                            }
                        });
                    }

                    @Override
                    public void onStatusChanged(String provider, int status, Bundle extras) {

                    }

                    @Override
                    public void onProviderEnabled(String provider) {

                    }

                    @Override
                    public void onProviderDisabled(String provider) {

                    }
                }));
                break;
            default:
                break;
        }
    }
}

```

### 2.2 LifeCycle 源码解析
先来一张汇总图：
![final arch](/images/posts/GoogleArchitectureComponent/final-architecture.png)

首先看下我们的 LifeCycleActivity 继承关系：

![final arch](/images/posts/ArchComponentSource/LifeOwner.jpg)


从 support 26.1.0开始，Support 默认实现了 LifeOwner,所以我们的 Activity 都变成了可被观测生命周期的对象，那接下来我们看下是如何被观测的呢？
我们先看看 SupportActivity里面到底做了些什么，代码如下：

```java
@RestrictTo(LIBRARY_GROUP)
public class SupportActivity extends Activity implements LifecycleOwner {
    /**
     * Storage for {@link ExtraData} instances.
     *
     * <p>Note that these objects are not retained across configuration changes</p>
     */
    private SimpleArrayMap<Class<? extends ExtraData>, ExtraData> mExtraDataMap =
            new SimpleArrayMap<>();

    private LifecycleRegistry mLifecycleRegistry = new LifecycleRegistry(this);

    /**
     * Store an instance of {@link ExtraData} for later retrieval by class name
     * via {@link #getExtraData}.
     *
     * <p>Note that these objects are not retained across configuration changes</p>
     *
     * @see #getExtraData
     * @hide
     */
    @RestrictTo(LIBRARY_GROUP)
    public void putExtraData(ExtraData extraData) {
        mExtraDataMap.put(extraData.getClass(), extraData);
    }

    @Override
    @SuppressWarnings("RestrictedApi")
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ReportFragment.injectIfNeededIn(this);
    }

```
在 onCreate 中 ReportFragment.injectIfNeededIn(this)这一行我们继续跟踪，代码如下：

```java
@RestrictTo(RestrictTo.Scope.LIBRARY_GROUP)
public class ReportFragment extends Fragment {

private static final String REPORT_FRAGMENT_TAG = "android.arch.lifecycle"
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

void setProcessListener(ActivityInitializationListener processListener) {
    mProcessListener = processListener;
}

interface ActivityInitializationListener {
    void onCreate();

    void onStart();

    void onResume();
}
}
```
在 injectIfNeededIn()方法中，直接向当前 Activity中插入了一个 ReportFragment，然后我们看下这个 ReportFragment中定义了一个 ActivityInitializationListener,这个是用来监听 Activity 初始化的，我们可以先忽略，
在 ReportFragment每个声明周期中都调用了dispatch(Lifecycle.Event.ON_XXLifeCycle);这个刚好对应了Lifecycle 的各个生命周期![lifecycle](/images/GoogleArchitectureComponent/lifecycle-states.png)。
到这里也就明白了，原来监听生命周期的原理就是通过在 Activity 中插入一个不可见的 Fragment,通过 Fragment 把 Activity 的生命周期代理转发出去。

接下来我们继续看如何转发？

我们继续跟踪 dispatch()方法:
```java
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
```
获取当前 Activity，然后判断是否实现了声明周期拥有者接口，如果实现了则通过 getLifecycle 拿到 Lifecycle，然后直接调用 handleLifecycleEvent 将当前生命周期事件转发出去。
接下来我们继续看 SupportActivity 中的实现：
```java
@RestrictTo(LIBRARY_GROUP)
public class SupportActivity extends Activity implements LifecycleOwner {
    /**
     * Storage for {@link ExtraData} instances.
     *
     * <p>Note that these objects are not retained across configuration changes</p>
     */
    private SimpleArrayMap<Class<? extends ExtraData>, ExtraData> mExtraDataMap =
            new SimpleArrayMap<>();

    private LifecycleRegistry mLifecycleRegistry = new LifecycleRegistry(this);

    /**
     * Store an instance of {@link ExtraData} for later retrieval by class name
     * via {@link #getExtraData}.
     *
     * <p>Note that these objects are not retained across configuration changes</p>
     *
     * @see #getExtraData
     * @hide
     */
    @RestrictTo(LIBRARY_GROUP)
    public void putExtraData(ExtraData extraData) {
        mExtraDataMap.put(extraData.getClass(), extraData);
    }

    @Override
    @SuppressWarnings("RestrictedApi")
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ReportFragment.injectIfNeededIn(this);
    }

    @CallSuper
    @Override
    protected void onSaveInstanceState(Bundle outState) {
        mLifecycleRegistry.markState(Lifecycle.State.CREATED);
        super.onSaveInstanceState(outState);
    }

    /**
     * Retrieves a previously set {@link ExtraData} by class name.
     *
     * @see #putExtraData
     * @hide
     */
    @RestrictTo(LIBRARY_GROUP)
    public <T extends ExtraData> T getExtraData(Class<T> extraDataClass) {
        return (T) mExtraDataMap.get(extraDataClass);
    }

    @Override
    public Lifecycle getLifecycle() {
        return mLifecycleRegistry;
    }

    /**
     * @hide
     */
    @RestrictTo(LIBRARY_GROUP)
    public static class ExtraData {
    }
}
```
getLifecycle 返回的是 mLifecycleRegistry，我们看其定义的地方 LifecycleRegistry,继续跟踪
```java
public class LifecycleRegistry extends Lifecycle {

    private static final String LOG_TAG = "LifecycleRegistry";

    /**
     * Custom list that keeps observers and can handle removals / additions during traversal.
     *
     * Invariant: at any moment of time for observer1 & observer2:
     * if addition_order(observer1) < addition_order(observer2), then
     * state(observer1) >= state(observer2),
     */
    private FastSafeIterableMap<LifecycleObserver, ObserverWithState> mObserverMap =
            new FastSafeIterableMap<>();
    /**
     * Current state
     */
    private State mState;
    
    /**
     * Creates a new LifecycleRegistry for the given provider.
     * <p>
     * You should usually create this inside your LifecycleOwner class's constructor and hold
     * onto the same instance.
     *
     * @param provider The owner LifecycleOwner
     */
    public LifecycleRegistry(@NonNull LifecycleOwner provider) {
        mLifecycleOwner = new WeakReference<>(provider);
        mState = INITIALIZED;
    }

    
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

    private boolean isSynced() {
        if (mObserverMap.size() == 0) {
            return true;
        }
        State eldestObserverState = mObserverMap.eldest().getValue().mState;
        State newestObserverState = mObserverMap.newest().getValue().mState;
        return eldestObserverState == newestObserverState && mState == newestObserverState;
    }

   

    @Override
    public void addObserver(@NonNull LifecycleObserver observer) {
        State initialState = mState == DESTROYED ? DESTROYED : INITIALIZED;
        ObserverWithState statefulObserver = new ObserverWithState(observer, initialState);
        ObserverWithState previous = mObserverMap.putIfAbsent(observer, statefulObserver);

        if (previous != null) {
            return;
        }
        LifecycleOwner lifecycleOwner = mLifecycleOwner.get();
        if (lifecycleOwner == null) {
            // it is null we should be destroyed. Fallback quickly
            return;
        }

        boolean isReentrance = mAddingObserverCounter != 0 || mHandlingEvent;
        State targetState = calculateTargetState(observer);
        mAddingObserverCounter++;
        while ((statefulObserver.mState.compareTo(targetState) < 0
                && mObserverMap.contains(observer))) {
            pushParentState(statefulObserver.mState);
            statefulObserver.dispatchEvent(lifecycleOwner, upEvent(statefulObserver.mState));
            popParentState();
            // mState / subling may have been changed recalculate
            targetState = calculateTargetState(observer);
        }

        if (!isReentrance) {
            // we do sync only on the top level.
            sync();
        }
        mAddingObserverCounter--;
    }

   
    @Override
    public void removeObserver(@NonNull LifecycleObserver observer) {
        // we consciously decided not to send destruction events here in opposition to addObserver.
        // Our reasons for that:
        // 1. These events haven't yet happened at all. In contrast to events in addObservers, that
        // actually occurred but earlier.
        // 2. There are cases when removeObserver happens as a consequence of some kind of fatal
        // event. If removeObserver method sends destruction events, then a clean up routine becomes
        // more cumbersome. More specific example of that is: your LifecycleObserver listens for
        // a web connection, in the usual routine in OnStop method you report to a server that a
        // session has just ended and you close the connection. Now let's assume now that you
        // lost an internet and as a result you removed this observer. If you get destruction
        // events in removeObserver, you should have a special case in your onStop method that
        // checks if your web connection died and you shouldn't try to report anything to a server.
        mObserverMap.remove(observer);
    }

    /**
     * The number of observers.
     *
     * @return The number of observers.
     */
    @SuppressWarnings("WeakerAccess")
    public int getObserverCount() {
        return mObserverMap.size();
    }

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

    static State min(@NonNull State state1, @Nullable State state2) {
        return state2 != null && state2.compareTo(state1) < 0 ? state2 : state1;
    }

    static class ObserverWithState {
        State mState;
        GenericLifecycleObserver mLifecycleObserver;

        ObserverWithState(LifecycleObserver observer, State initialState) {
            mLifecycleObserver = Lifecycling.getCallback(observer);
            mState = initialState;
        }

        void dispatchEvent(LifecycleOwner owner, Event event) {
            State newState = getStateAfter(event);
            mState = min(mState, newState);
            mLifecycleObserver.onStateChanged(owner, event);
            mState = newState;
        }
    }
}
```

在这里看到了熟悉的 观察者模式 addObserver,removeObserver,同时也看到了分发的入口 handleLifecycleEvent,根据当前观察者列表中观察者的状态，分发到 onStateChanged()。
我们看下 lifecycle 中已经定义了若干常用的观察者，如下图：
![observer](/images/ArchcomponentSource/lifecycle-observer.jpg)
以FullLifecyleObserver 为例，如下，其 onStateChanged中根据不同状态回调 FullLifecycleObserver 的生命周期代理。
```java
/*
 * Copyright (C) 2017 The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package android.arch.lifecycle;

class FullLifecycleObserverAdapter implements GenericLifecycleObserver {

    private final FullLifecycleObserver mObserver;

    FullLifecycleObserverAdapter(FullLifecycleObserver observer) {
        mObserver = observer;
    }

    @Override
    public void onStateChanged(LifecycleOwner source, Lifecycle.Event event) {
        switch (event) {
            case ON_CREATE:
                mObserver.onCreate(source);
                break;
            case ON_START:
                mObserver.onStart(source);
                break;
            case ON_RESUME:
                mObserver.onResume(source);
                break;
            case ON_PAUSE:
                mObserver.onPause(source);
                break;
            case ON_STOP:
                mObserver.onStop(source);
                break;
            case ON_DESTROY:
                mObserver.onDestroy(source);
                break;
            case ON_ANY:
                throw new IllegalArgumentException("ON_ANY must not been send by anybody");
        }
    }
}

```
然后就回到了我们代码中对各个生命周期的监听回调中。注意我们的 demo 中是通过注解声明如何处理各个生命周期事件，其原理是通过 apt 动态生成,动态生成的代码如下：
```java
public class MyLocationListener_LifecycleAdapter implements GeneratedAdapter {
  final MyLocationListener mReceiver;

  MyLocationListener_LifecycleAdapter(MyLocationListener receiver) {
    this.mReceiver = receiver;
  }

  @Override
  public void callMethods(LifecycleOwner owner, Lifecycle.Event event, boolean onAny,
      MethodCallsLogger logger) {
    boolean hasLogger = logger != null;
    if (onAny) {
      return;
    }
    if (event == Lifecycle.Event.ON_START) {
      if (!hasLogger || logger.approveCall("start", 1)) {
        mReceiver.start();
      }
      return;
    }
    if (event == Lifecycle.Event.ON_RESUME) {
      if (!hasLogger || logger.approveCall("enable", 1)) {
        mReceiver.enable();
      }
      return;
    }
    if (event == Lifecycle.Event.ON_STOP) {
      if (!hasLogger || logger.approveCall("stop", 1)) {
        mReceiver.stop();
      }
      return;
    }
    if (event == Lifecycle.Event.ON_DESTROY) {
      if (!hasLogger || logger.approveCall("release", 1)) {
        mReceiver.release();
      }
      return;
    }
  }
}
```
然后加入到CompositeGeneratedAdaptersObserver,系统首先回调到 CompositeGeneratedAdaptersObserver.onStateChanged,然后在回调到 GeneratedAdapter 中的 callMethods,进而继续根据注解生成的分发方法进行下一步分发。
## 3 LiveData &ViewModel 源码解析


## 3.1 ViewModel & LiveData 使用：

```java

public class MainModel extends AndroidViewModel {

    private MutableLiveData<String> mMutableLiveData = new MutableLiveData<>();

    public MainModel(Application application) {
        super(application);
    }

    public MutableLiveData<String> getMutableLiveData() {
        return mMutableLiveData;
    }

    /**
     * 模仿获取数据.
     */
    public void requestGetData() {
        new Thread(){
            @Override
            public void run() {
                super.run();
                try {
                    Thread.sleep(1000);
                    getMutableLiveData().postValue("获取的数据");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }.start();

    }
@Override
protected void onCleared() {
    Log.d("MainModel", "清除资源");
}}

```

Activity 中使用的代码如下：

```java
public class ViewModelLivingActivity extends AppCompatActivity {

    private MainModel mMainModel;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_common_btn);
        mMainModel = ViewModelProviders.of(this).get(
                MainModel.class);

        mMainModel.getMutableLiveData().observe(this, new Observer<String>() {
            @Override
            public void onChanged(@Nullable final String s) {
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        Toast.makeText(ViewModelLivingActivity.this, s, Toast.LENGTH_SHORT).show();
                    }
                });

            }
        });


    }
    public void onClick(View view) {
        switch (view.getId()) {
            case R.id.button:
                mMainModel.requestGetData();
                break;
            default:
                break;
        }
    }
}

```
## 3.2 ViewModel & LiveData 源码解析
### 3.2.1 ViewModel 源码解析
ViewModel 保存当前应用页面的数据，不会因为 Activity 重启等丢失数据，ViewModel活的比Activity更长,这是怎么做到的呢？
首先我们看下 ViewModel 是如何生成的：
mMainModel = ViewModelProviders.of(this).get(MainModel.class);
首先我们要看下 ViewModelProviders.of 做了什么工作：
```java
@MainThread
public static ViewModelProvider of(@NonNull FragmentActivity activity) {

    //这句话是创建了sDefaultFactory对象
    initializeFactoryIfNeeded(checkApplication(activity));

    return new ViewModelProvider(ViewModelStores.of(activity), sDefaultFactory);
}

public ViewModelProvider(@NonNull ViewModelStore store, @NonNull Factory factory) {
    mFactory = factory;
    this.mViewModelStore = store;
}

/**
*这个factory只有一个通过反射创建对象（持有application引用）
*/@SuppressWarnings("WeakerAccess")public static class DefaultFactory extends ViewModelProvider.NewInstanceFactory {    private Application mApplication;

    /**
     * Creates a {@code DefaultFactory}
     *
     * @param application an application to pass in {@link AndroidViewModel}
     */
    public DefaultFactory(@NonNull Application application) {
        mApplication = application;
    }

    @NonNull
    @Override
    public <T extends ViewModel> T create(@NonNull Class<T> modelClass) {
        if (AndroidViewModel.class.isAssignableFrom(modelClass)) {
            //noinspection TryWithIdenticalCatches
            try {
                return modelClass.getConstructor(Application.class).newInstance(mApplication);
            } catch (NoSuchMethodException e) {
                throw new RuntimeException("Cannot create an instance of " + modelClass, e);
            } catch (IllegalAccessException e) {
                throw new RuntimeException("Cannot create an instance of " + modelClass, e);
            } catch (InstantiationException e) {
                throw new RuntimeException("Cannot create an instance of " + modelClass, e);
            } catch (InvocationTargetException e) {
                throw new RuntimeException("Cannot create an instance of " + modelClass, e);
            }
        }
        return super.create(modelClass);
    }
}

```
我们接下来看下 ViewModelProviders.of 做了什么工作：
```java
public class ViewModelStores {

    private ViewModelStores() {
    }

    /**
     * Returns the {@link ViewModelStore} of the given activity.
     *
     * @param activity an activity whose {@code ViewModelStore} is requested
     * @return a {@code ViewModelStore}
     */就是这句话
    @MainThread
    public static ViewModelStore of(@NonNull FragmentActivity activity) {
        return holderFragmentFor(activity).getViewModelStore();
    }

    /**
     * Returns the {@link ViewModelStore} of the given fragment.
     *
     * @param fragment a fragment whose {@code ViewModelStore} is requested
     * @return a {@code ViewModelStore}
     */
    @MainThread
    public static ViewModelStore of(@NonNull Fragment fragment) {
        return holderFragmentFor(fragment).getViewModelStore();
    }
    private Map<Activity, HolderFragment> mNotCommittedActivityHolders = new HashMap<>();

/**
 * @hide
 */
@RestrictTo(RestrictTo.Scope.LIBRARY_GROUP)
public static HolderFragment holderFragmentFor(FragmentActivity activity) {
    return sHolderFragmentManager.holderFragmentFor(activity);
}

/**
 * 主要代码
 */
HolderFragment holderFragmentFor(FragmentActivity activity) {
    FragmentManager fm = activity.getSupportFragmentManager();
    HolderFragment holder = findHolderFragment(fm);
    if (holder != null) {
        return holder;
    }
    holder = mNotCommittedActivityHolders.get(activity);
    if (holder != null) {
        return holder;
    }

    if (!mActivityCallbacksIsAdded) {
        mActivityCallbacksIsAdded = true;
        activity.getApplication().registerActivityLifecycleCallbacks(mActivityCallbacks);
    }
    holder = createHolderFragment(fm);
    mNotCommittedActivityHolders.put(activity, holder);
    return holder;
}

private static HolderFragment findHolderFragment(FragmentManager manager) {
    if (manager.isDestroyed()) {
        throw new IllegalStateException("Can't access ViewModels from onDestroy");
    }

    Fragment fragmentByTag = manager.findFragmentByTag(HOLDER_TAG);
    if (fragmentByTag != null && !(fragmentByTag instanceof HolderFragment)) {
        throw new IllegalStateException("Unexpected "
                + "fragment instance was returned by HOLDER_TAG");
    }
    return (HolderFragment) fragmentByTag;
}

private static HolderFragment createHolderFragment(FragmentManager fragmentManager) {
    HolderFragment holder = new HolderFragment();
    fragmentManager.beginTransaction().add(holder, HOLDER_TAG).commitAllowingStateLoss();
    return holder;
}


}

```
这里又出现了一个熟悉的 Fragment:holderFragmentFor(),其实现仍然是向 Activity 中插入了一个 Fragment,HolderFragment。
我们继续看 HolderFragment中做了什么
```java
@Override
    public void onDestroy() {
        super.onDestroy();
        mViewModelStore.clear();
    }

    public ViewModelStore getViewModelStore() {
        return mViewModelStore;
    }
```
核心代码如上：用 HolderFragment 的目的是为了监听 Activity 生命周期，并且在 onDestroy 中调用 mViewModelStore.clear()清理数据。
但是还有一个疑问，Activity 被销毁重建，为什么没有 clear 数据。其实这里用了一个非常绝妙的小技巧，那就是 Activity 在销毁重建的时候，会调用 onSaveInstanceState,做数据保存工作，Fragment 在这里被缓存，并没有销毁，也就算没有走 onDestroy,在新页面恢复的时候，取出来的仍然是之前的 Fragment.
所以 ViewModel能够跨越 Activity 销毁重建。
至于ViewModelProviders.of(this).get(MainModel.class)的get方法就简单了，直接调用默认或者自己集成的factory的create方法，同时又将这个viewModel put到mViewModelStore里，至于mViewModelStore是啥，干嘛用，继续来分析。

总结为：activity持有ViewModelStore，ViewModelStore中存放多个ViewModel，并且使用了一个fragment监听activity生命周期，在activity被销毁时调用所有存于 ViewModelStore中的ViewModel的clear方法。



### 3.2.2 LiveData 源码解析

我们先回顾下 LiveData的使用：
```java

public class MainModel extends AndroidViewModel {

    private MutableLiveData<String> mMutableLiveData = new MutableLiveData<>();

    public MainModel(Application application) {
        super(application);
    }

    public MutableLiveData<String> getMutableLiveData() {
        return mMutableLiveData;
    }

    /**
     * 模仿获取数据.
     */
    public void requestGetData() {
        new Thread(){
            @Override
            public void run() {
                super.run();
                try {
                    Thread.sleep(1000);
                    getMutableLiveData().postValue("获取的数据");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }.start();

    }
@Override
protected void onCleared() {
    Log.d("MainModel", "清除资源");
}}

```

核心在于使用了 MutableLiveData,外部注册监听：
```java
mMainModel.getMutableLiveData().observe(this, new Observer<String>() {
            @Override
            public void onChanged(@Nullable final String s) {
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        Toast.makeText(ViewModelLivingActivity.this, s, Toast.LENGTH_SHORT).show();
                    }
                });

            }
        });
```

并且在数据变化的时候调用了 MutableLiveData.postValue 更新数据。
为什么 LiveData能够感知生命周期，在合适 的时候通知呢？
带着疑问，我们继续跟踪observe
```java
    @MainThread
    public void observe(@NonNull LifecycleOwner owner, @NonNull Observer<T> observer) {
        if (owner.getLifecycle().getCurrentState() == DESTROYED) {
            // ignore
            return;
        }
        LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer);
        LifecycleBoundObserver existing = mObservers.putIfAbsent(observer, wrapper);
        if (existing != null && existing.owner != wrapper.owner) {
            throw new IllegalArgumentException("Cannot add the same observer"
                    + " with different lifecycles");
        }
        if (existing != null) {
            return;
        }
        owner.getLifecycle().addObserver(wrapper);
    }

```
原来如此，LiveData 的 Observe 是通过 已经实现了 ActivityOwner 的 SupportActivity的 LifeCycle 来监听生命周期，其观察者如下：
```java
class LifecycleBoundObserver implements GenericLifecycleObserver {
        public final LifecycleOwner owner;
        public final Observer<T> observer;
        public boolean active;
        public int lastVersion = START_VERSION;

        LifecycleBoundObserver(LifecycleOwner owner, Observer<T> observer) {
            this.owner = owner;
            this.observer = observer;
        }

        @Override
        public void onStateChanged(LifecycleOwner source, Lifecycle.Event event) {
            if (owner.getLifecycle().getCurrentState() == DESTROYED) {
                removeObserver(observer);
                return;
            }
            // immediately set active state, so we'd never dispatch anything to inactive
            // owner
            activeStateChanged(isActiveState(owner.getLifecycle().getCurrentState()));
        }

        void activeStateChanged(boolean newActive) {
            if (newActive == active) {
                return;
            }
            active = newActive;
            boolean wasInactive = LiveData.this.mActiveCount == 0;
            LiveData.this.mActiveCount += active ? 1 : -1;
            if (wasInactive && active) {
                onActive();
            }
            if (LiveData.this.mActiveCount == 0 && !active) {
                onInactive();
            }
            if (active) {
                dispatchingValue(this);
            }
        }
    }
```
在生命周期 activite 的情况下 dispatchValue，通知数据变化，在 onDestroy 的时候移除监听。
后续代码不在一一解释，到这里应该明白了 LiveData 是借助了 LiveCycle 来感知生命周期，并通知数据变化。


## 4.总结

通过对源码的分析可知，整个 ArchitectureComponent核心是通过 Fragment 实现对 Activity 生命周期监听，进而封装出 LifeCycle,ViewModel,LiveData 等组件，以供开发者搭配使用，减少犯错可能，最终优化开发。

([Github Demo 地址](https://github.com/guofeng007/ArchitectureComponentDemo)):
thanks to [[顺其自然55](https://juejin.im/post/5a03ed9251882560e356138c)]

