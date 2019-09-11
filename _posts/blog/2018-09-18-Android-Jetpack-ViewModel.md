---
layout: post
title: Android Jetpack 之 ViewModel
categories: Blog
description: Android Jetpack 之 ViewModel
keywords:      Android Jetpack 之 ViewModel

---

# Android Jetpack 之 ViewModel
[原文](https://juejin.im/post/5beccf0a5188251e1f50c574?utm_source=gold_browser_extension)

`ViewModel` 是 Android 架构组件之一，用于分离 UI 逻辑与 UI 数据。在发生 *Configuration Changes* 时，它不会被销毁。在界面重建后，方便开发者呈现界面销毁前的 UI 状态。

本文主要分析 `ViewModel` 的以下3个方面：

- 获取和创建过程。
- *Configuration Changes* 存活原理。
- 销毁过程。

## 1. 依赖库

```
implementation "androidx.fragment:fragment:1.0.0"
implementation "androidx.lifecycle:lifecycle-viewmodel:2.0.0"
implementation "androidx.lifecycle:lifecycle-extensions:2.0.0"
复制代码
```

## 2. 主要类与接口

```
import androidx.fragment.app.Fragment;
import androidx.fragment.app.FragmentActivity;
import androidx.lifecycle.ViewModel;
import androidx.lifecycle.AndroidViewModel;
import androidx.lifecycle.ViewModelProvider;
import androidx.lifecycle.ViewModelProvider.Factory;
import androidx.lifecycle.ViewModelProviders;
import androidx.lifecycle.ViewModelStore;
import androidx.lifecycle.ViewModelStoreOwner;
复制代码
```

## 3. ViewModel

`ViewModel` 是一个抽象类，类中只定义了一个空实现的 `onCleared()` 方法。

```
public abstract class ViewModel {
    /**
     * This method will be called when this ViewModel is no longer used and will be destroyed.
     * <p>
     * It is useful when ViewModel observes some data and you need to clear this subscription to
     * prevent a leak of this ViewModel.
     */
    @SuppressWarnings("WeakerAccess")
    protected void onCleared() {
    }
}
复制代码
```

### 3.1 AndroidViewModel

`AndroidViewModel` 类扩展了 `ViewModel` 类，增加了 `Application` 字段，在构造方法初始化，并提供了 `getApplication()` 方法。

```
public class AndroidViewModel extends ViewModel {
    private Application mApplication;

    public AndroidViewModel(@NonNull Application application) {
        mApplication = application;
    }

    /**
     * Return the application.
     */
    @NonNull
    public <T extends Application> T getApplication() {
        return (T) mApplication;
    }
}
复制代码
```

## 4. 获取和创建过程分析

获取 `ViewModel` 对象代码如下：

```
ViewModelProviders.of(activityOrFragment).get(ViewModel::class.java)
复制代码
```

### 4.1 ViewModelProviders

`ViewModelProviders` 类提供了4个静态工厂方法 `of()` 创建新的 `ViewModelProvider` 对象。

```
ViewModelProviders.of(Fragment)
ViewModelProviders.of(FragmentActivity)
ViewModelProviders.of(Fragment, Factory)
ViewModelProviders.of(FragmentActivity, Factory)
复制代码
```

### 4.2 ViewModelProvider

`ViewModelProvider` 负责提供 `ViewModel` 对象，类中定义了以下两个字段：

```
private final Factory mFactory;
private final ViewModelStore mViewModelStore;
复制代码
```

先说说这两个类的功能。

### 4.3 ViewModelProvider.Factory

`Factory` 接口定义了一个创建 `ViewModel` 的接口 `create()`，`ViewModelProvider` 在需要时调用该方法新建 `ViewModel` 对象。

```
public interface Factory {
    <T extends ViewModel> T create(@NonNull Class<T> modelClass);
}
复制代码
```

Android 已经内置了2个 `Factory` 实现类，分别是：

- `AndroidViewModelFactory` 实现类，可以创建 `ViewModel` 和 `AndroidViewModel` 子类对象。
- `NewInstanceFactory` 类，只可以创建 `ViewModel` 子类对象。

它们的实现都是通过反射机制调用 `ViewModel` 子类的构造方法创建对象。

```
public static class NewInstanceFactory implements Factory {
    @Override
    public <T extends ViewModel> T create(Class<T> modelClass) {
        try {
            return modelClass.newInstance();
        } catch (InstantiationException e) {
            throw new RuntimeException("Cannot create an instance of " + modelClass, e);
        } catch (IllegalAccessException e) {
            throw new RuntimeException("Cannot create an instance of " + modelClass, e);
        }
    }
}
复制代码
```

`AndroidViewModelFactory` 继承 `NewInstanceFactory` 类，是个单例，支持创建 `AndroidViewModel` 子类对象。

```
public static class AndroidViewModelFactory extends ViewModelProvider.NewInstanceFactory {

    private static AndroidViewModelFactory sInstance;

    public static AndroidViewModelFactory getInstance(Application application) {
        if (sInstance == null) {
            sInstance = new AndroidViewModelFactory(application);
        }
        return sInstance;
    }

    private Application mApplication;

    public AndroidViewModelFactory(Application application) {
        mApplication = application;
    }

    @Override
    public <T extends ViewModel> T create(Class<T> modelClass) {
        if (AndroidViewModel.class.isAssignableFrom(modelClass)) {
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
复制代码
```

### 4.4 ViewModelStore

`ViewModelStore` 类中维护一个 `Map<String, ViewModel>` 对象存储已创建的 `ViewModel` 对象，并提供 `put()` 和 `get()` 方法。

```
public class ViewModelStore {
    private final HashMap<String, ViewModel> mMap = new HashMap<>();
    
    final void put(String key, ViewModel viewModel) {
        ViewModel oldViewModel = mMap.put(key, viewModel);
    }

    final ViewModel get(String key) {
        return mMap.get(key);
    }
}
复制代码
```

### 4.5 ViewModelStoreOwner

`ViewModelStore` 是来自于 `FragmentActivity` 和 `Fragment`，它们实现了 `ViewModelStoreOwner` 接口，返回当前 UI 作用域里的 `ViewModelStore` 对象。

```
public interface ViewModelStoreOwner {
    ViewModelStore getViewModelStore();
}
复制代码
```

在 `Fragment` 类中的实现如下：

```
public ViewModelStore getViewModelStore() {
    if (getContext() == null) {
        throw new IllegalStateException("Can't access ViewModels from detached fragment");
    }
    if (mViewModelStore == null) {
        mViewModelStore = new ViewModelStore();
    }
    return mViewModelStore;
}
复制代码
```

在 `FragmentActivity` 类中的实现如下：

```
public ViewModelStore getViewModelStore() {
    if (getApplication() == null) {
        throw new IllegalStateException("Your activity is not yet attached to the "
                + "Application instance. You can't request ViewModel before onCreate call.");
    }
    if (mViewModelStore == null) {
        mViewModelStore = new ViewModelStore();
    }
    return mViewModelStore;
}
复制代码
```

### 4.6 创建 ViewModelProvider

回到 `of()` 方法的实现。

```
public static ViewModelProvider of(FragmentActivity activity, Factory factory) {
    Application application = checkApplication(activity);
    if (factory == null) {
        factory = ViewModelProvider.AndroidViewModelFactory.getInstance(application);
    }
    return new ViewModelProvider(activity.getViewModelStore(), factory);
}
复制代码
```

在创建 `ViewModelProvider` 对象时需要传入 `ViewModelStore` 和 `Factory` 对象。若 `factory` 为 `null`，将使用 `AndroidViewModelFactory` 单例对象。

### 4.7 获取 `ViewModel` 对象

调用 `ViewModelProvider` 对象的 `get()` 方法获取 `ViewModel` 对象，如果在 `ViewModelStore` 里不存在，则使用 `Factory` 创建一个新的对象并存放到 `ViewModelStore`里。

```
public <T extends ViewModel> T get(String key, Class<T> modelClass) {
    ViewModel viewModel = mViewModelStore.get(key);

    if (modelClass.isInstance(viewModel)) {
        return (T) viewModel;
    }

    viewModel = mFactory.create(modelClass);
    mViewModelStore.put(key, viewModel);

    return (T) viewModel;
}
复制代码
```

## 5. *Configuration Changes* 存活原理

当 `Activity` 或 `Fragment` 被系统重建时，`ViewModel` 对象不会被销毁，新的 `Activity` 或 `Fragment` 对象拿到的是同一个 `ViewModel` 对象。

在 `FragmentActivity#onRetainNonConfigurationInstance()` 方法中，会将 `ViewModelStore`对象保留起来。

```
public final Object onRetainNonConfigurationInstance() {
    Object custom = onRetainCustomNonConfigurationInstance();

    FragmentManagerNonConfig fragments = mFragments.retainNestedNonConfig();

    if (fragments == null && mViewModelStore == null && custom == null) {
        return null;
    }

    NonConfigurationInstances nci = new NonConfigurationInstances();
    nci.custom = custom;
    nci.viewModelStore = mViewModelStore;
    nci.fragments = fragments;
    return nci;
}
复制代码
```

然后在 `onCreate()` 方法能获取之前保留起来的 `ViewModelStore` 对象。

```
protected void onCreate(Bundle savedInstanceState) {
    mFragments.attachHost(null /*parent*/);
    super.onCreate(savedInstanceState);

    NonConfigurationInstances nc = (NonConfigurationInstances) getLastNonConfigurationInstance();
    if (nc != null) {
        mViewModelStore = nc.viewModelStore;
    }
    // ...
}
复制代码
```

那 `Fragment` 作用域里是如何实现的呢？在 `FragmentActivity` 的 `onRetainNonConfigurationInstance()` 方法中里有这样一句代码：

```
FragmentManagerNonConfig fragments = mFragments.retainNestedNonConfig();
复制代码
```

实现保留的机制是一样的，只不过放在 `FragmentManagerNonConfig` 对象中。是在 `FragmentManager#saveNonConfig()` 方法中将 `ViewModelStore` 对象保存到 `FragmentManagerNonConfig` 里的。

```
void saveNonConfig() {
    ArrayList<Fragment> fragments = null;
    ArrayList<FragmentManagerNonConfig> childFragments = null;
    ArrayList<ViewModelStore> viewModelStores = null;
    if (mActive != null) {
        for (int i=0; i<mActive.size(); i++) {
            Fragment f = mActive.valueAt(i);
            if (f != null) {
                if (f.mRetainInstance) {
                    if (fragments == null) {
                        fragments = new ArrayList<Fragment>();
                    }
                    fragments.add(f);
                    f.mTargetIndex = f.mTarget != null ? f.mTarget.mIndex : -1;
                    if (DEBUG) Log.v(TAG, "retainNonConfig: keeping retained " + f);
                }
                FragmentManagerNonConfig child;
                if (f.mChildFragmentManager != null) {
                    f.mChildFragmentManager.saveNonConfig();
                    child = f.mChildFragmentManager.mSavedNonConfig;
                } else {
                    // f.mChildNonConfig may be not null, when the parent fragment is
                    // in the backstack.
                    child = f.mChildNonConfig;
                }

                if (childFragments == null && child != null) {
                    childFragments = new ArrayList<>(mActive.size());
                    for (int j = 0; j < i; j++) {
                        childFragments.add(null);
                    }
                }

                if (childFragments != null) {
                    childFragments.add(child);
                }
                if (viewModelStores == null && f.mViewModelStore != null) {
                    viewModelStores = new ArrayList<>(mActive.size());
                    for (int j = 0; j < i; j++) {
                        viewModelStores.add(null);
                    }
                }

                if (viewModelStores != null) {
                    viewModelStores.add(f.mViewModelStore);
                }
            }
        }
    }
    if (fragments == null && childFragments == null && viewModelStores == null) {
        mSavedNonConfig = null;
    } else {
        mSavedNonConfig = new FragmentManagerNonConfig(fragments, childFragments,
        viewModelStores);
    }
}
复制代码
```

该方法的调用顺序是：`FragmentActivity#onSaveInstanceState()` -> `FragmentManager#saveAllState()` -> `FragmentManager#saveNonConfig()`。

## 6. 销毁过程

在 `FragmentActivity` 类的 `onDestory()` 方法中。

```
@Override
protected void onDestroy() {
    super.onDestroy();
    if (mViewModelStore != null && !isChangingConfigurations()) {
        mViewModelStore.clear();
    }
    mFragments.dispatchDestroy();
}
复制代码
```

在 `Fragment` 类的 `onDestory()` 方法中。

```
public void onDestroy() {
    mCalled = true;
    FragmentActivity activity = getActivity();
    boolean isChangingConfigurations = activity != null && activity.isChangingConfigurations();
    if (mViewModelStore != null && !isChangingConfigurations) {
        mViewModelStore.clear();
    }
}
复制代码
```

先判断是否有发生 *Configuration Changes*，如果没有则会调用 `ViewModelStore` 的 `clear()`方法，再一一调用每一个 `ViewModel` 的 `onCleared()` 方法。

```
public final void clear() {
    for (ViewModel vm : mMap.values()) {
        vm.onCleared();
    }
    mMap.clear();
}
复制代码
```

## 7. 总结

以上便是 `ViewModel` 3个主要过程的剖析，这里做一下总结。

- 通过 `ViewModelProviders` 创建 `ViewModelProvider` 对象，调用该对象的 `get()` 方法获取 `ViewModel` 对象。 当 `ViewModelStore` 里不存在想要的对象，`ViewModelProvider` 会使用 `Factory` 新建一个对象并存放到 `ViewModelStore` 里。
- 当发生 发生 *Configuration Changes* 时，`FragmentActivity` 利用 `getLastNonConfigurationInstance()`、`onRetainNonConfigurationInstance()` 方法实现 `ViewModelStore` 的保留与恢复，进而实现 `ViewModel` 对象的保活。
- 当 `FragmentActivity` 和 `Fragment` 被销毁时，会根据是否发生 *Configuration Changes*来决定是否销毁 `ViewModel`。