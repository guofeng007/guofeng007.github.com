---
layout: post
title: 滴滴回放平台DiDiPrism源码解析
categories: Blog
description: 用户行为回放是一种用于记录、重现和分析用户在应用程序或网站上的交互行为的技术。它通常用于改进用户体验、诊断问题、进行性能优化以及进行安全审计。以下是用户行为回放的关键概念和工作原理
keywords: 回放平台，DiDiPrism，源码解析
---


# 一、引言

## 什么是用户回放

用户行为回放是一种用于记录、重现和分析用户在应用程序或网站上的交互行为的技术。它通常用于改进用户体验、诊断问题、进行性能优化以及进行安全审计。以下是用户行为回放的关键概念和工作原理：

1. **录制用户行为**：在用户与应用程序或网站互动时，用户行为回放工具会记录下用户的各种操作，包括点击、滚动、输入、导航等。这些记录通常以事件流或日志的形式存储。
2. **重放用户行为**：一旦用户行为被记录，系统可以将其重新播放以还原用户的操作。这意味着应用程序或网站的界面将按照与用户原始交互相同的方式呈现，从而完全重现用户的体验。
3. **回放粒度**：用户行为回放通常允许以不同的粒度进行回放，从整个会话到特定的用户操作，用户可以选择回放的级别。
4. **用途**：用户行为回放的主要用途包括用户体验改进（识别用户界面问题和瓶颈）、性能优化（发现慢操作或资源消耗高的操作）、安全审计（检测潜在的安全漏洞和攻击）以及用户支持和培训。
5. **隐私和安全**：在使用用户行为回放技术时，必须谨慎处理用户数据的隐私和安全问题。通常，需要去除敏感信息，并采取措施确保用户数据的保密性。
6. **支持多平台**：用户行为回放可以在多个平台上进行，包括Web应用、移动应用（如Android和iOS）以及桌面应用。

用户行为回放工具的选择通常取决于应用程序或网站的需求和技术栈。这些工具可以自动化识别问题、优化性能，并提供有关用户行为的深入见解，从而有助于提高用户满意度和应用程序的质量。

## 为什么选择 DiDiPrism 作为Android回放平台

目前类似做回放的有支付宝的soloPie，但是比较复杂，录制操作需要手动操作，不太适合。

滴滴的DiDiPrism具备自动录制，自动回放，比较轻量，是目前开源项目中比较适合进行定制化开发。

# 二、安装和集成

<img src="https://p.ipic.vip/s58qz0.png" alt="image-20230918185833931" style="zoom:50%;" />

- prism-monitor模块负责采集操作行为

- prism-playback模块负责操作行为回放

- prism-behavior模块负责操作行为检测（目前暂时没想到场景以及为啥使用，可能滴滴有自己内部的诉求吧）

  线上一般是集成prism-monitor采集操作，回放prism-playback集成到debug包或者内部测试包（不建议这部分放在线上app，有一定风险）

## prism-monitor使用说明

``

```
// 在Application创建时期初始化
PrismMonitor.getInstance().init(application);
// 开始并设置监听事件
PrismMonitor.getInstance().start();
PrismMonitor.getInstance().addOnPrismMonitorListener(new PrismMonitor.OnPrismMonitorListener() {
    @Override
    public void onEvent(EventData eventData) {
        // 可以通过此方法获取事件信息，比如文字回放等，具体使用请查看demo。
        EventInfo eventInfo = PlaybackHelper.convertEventInfo(eventData);
    }
});
```

## prism-playback使用说明

```
// 在Application创建时期初始化
PrismPlayback.getInstance().init(this);

//通过PrismMonitor监听采集操作数据
List<EventData> mPlaybackEvents = ...;
//开启回放
PrismPlayback.getInstance().playback(mPlaybackEvents);
```

# 视频演示

<video src="https://p.ipic.vip/tkm178.mp4"></video>

# 三、原理介绍

行为采集主要采集用户的点击操作。

比较有挑战的是点击操作的采集，因为我们需要：

1. 获取被点击的view
2. 根据view生成唯一标识

## 1. 获取被点击的view

简单列一下目前一些主流方案：

- 遍历Activity视图中所有的view，给View设置AccessibilityDelegate
- 通过ASM给click等事件插入代码
- 通过继承baseActivity或baseDialog的方式，在dispatchTouchEvent ACTION_DOWN时，结合TouchTarget获取

这里不分析各主流方案的利弊，只介绍小桔棱镜的所采用的方式。

小桔棱镜采用的方案和最后一种方案类似，但不是通过baseActivity或baseDialog的方式，而是通过监听Window。Window.Callback可能大家不是特别熟悉，通过给Window setCallback可以拿到`dispatchTouchEvent`和`dispatchKeyEvent`两个回调事件。

- `dispatchTouchEvent` 在ACTION_DOWN时，结合TouchTarget可以获取到被点击的view。
- `dispatchKeyEvent` 可以获取到返回键等事件。

接下里的问题就在于 **如何监听Window？**

关于Window，可以与`WindowManagerImpl`联系起来，因为它管理着App所有的Window实例，Window实例的实体其实就是View类型。可以看下`WindowManagerImpl`的源码：

```
public final class WindowManagerImpl implements WindowManager {
    ...
    private final WindowManagerGlobal mGlobal = WindowManagerGlobal.getInstance();
    ...
    @Override
    public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
        applyDefaultToken(params);
        mGlobal.addView(view, params, mContext.getDisplay(), mParentWindow);
    }
    ...
}
```



通过源码发现，Window实例的实体view被添加进`mGlobal`，继续看`WindowManagerGlobal`的源码：

```
public final class WindowManagerGlobal {
	...
	private final ArrayList<View> mViews = new ArrayList<View>();
	...
	public void addView(View view, ViewGroup.LayoutParams params, Display display, Window parentWindow) {
		...
		mViews.add(view);
		...
    }
    ...
}
```



发现实体view最终被添加进`mViews`，接下来，我们可以反射这个`mViews`，将其改造为可监听的对象，比如：

```
public class WindowObserver extends ArrayList<View> {
    @Override
    public boolean add(View view) {
        // ...
        return super.add(view);
    }
    
    @Override
    public View remove(int index) {
        // ...
        return super.remove(index);
    }
}
```



这样一来，App每个Window展示的时候，我们就能第一时间获取到该Window实例的实体view。 接下来，我们再通过实体view反向拿到Window实例。通过断点，会发现这个view当为Activity或Dialog的window实例时，它就是`DecorView`。对于它应该相当的熟悉！看下它的源码：

```
public class DecorView extends FrameLayout implements RootViewSurfaceTaker, WindowCallbacks {
    ...
    private PhoneWindow mWindow;
    ...
}
```



看到没，它直接持有window实例！我们反射直接去拿！

如此这般操作后，我们拿到了window实例，就可以setCallback，获取到所有用户操作的dispatchTouchEvent和dispatchKeyEvent回调事件，进而获取被点击的view，顺便也获取到了返回键等事件。

## 2. 根据view生成唯一标识

目前一些主流方案会将一些关键信息按约定的格式组合而成，作为唯一标识。这些关键信息有：Activity类名、View Id或Resource Id名称、View Class名称、View Path等。不同方案有不同的考量，优化的方式也不同。 下面就从三个维度，介绍下小桔棱镜的做法：

**2.1 被点击view所在窗口信息** 我们并没有直接使用Activity类名，因为被点击的view除了发生在Activity中，还有发生在Dialog，所以我们加了一层窗口的逻辑，也就是窗口的类型，使用`w`表示窗口信息，格式如下： `w_&_{窗口名称}_&_{窗口类型}`，窗口名称其实也就是Activity类名，窗口类型有0或1，表示Activity或Dialog，由`_&_`连接，比如 `w_&_com.prism.MainActivity_&_0`。

**2.2 被点击view在ViewTree上的路径信息** 我们不记录每个view层级上的View Class名称或者index，只会记录关键层级，使用`vp`表示View Path信息。举几个例子：

- 当层级上的view能获取到view id时，比如： `vp_&_titlebar_item_left_back/thanos_title_bar/content[01]/`，其中`content[01]`表示系统自带的那个`android.R.id.content`，通过`[01]`区别。
- 当层级上的view类型有`AbsListView`或`RecyclerView`时，比如： `vp_&_*/listView/navigation_drawer/drawer_layout/content[01]/_^_vl_&_l:4,10`，其中`*`表示被点击的那个ListView item层级，另外使用`vl`来描述可复用容器item的信息，`l:4,10`表示AbsListView容器中第4位，在数据源中第10位。

**2.3 被点击view自身可提取的信息**

- 当view存在id时，使用`vi`表示id信息，比如：`vi_&_titlebar_item_left_back`
- 当view存在drawable等资源时，使用`vr`表示资源信息，比如：`vr_&_selector_btn_confirm`
- 当view存在可提取文本信息时，使用`vc`表示文本信息，比如：`vc_&_确定`

最后，将以上三个维度提取的信息通过`_^_`连接起来，作为被点击view唯一标识，比如`w_&_com.prism.MainActivity_&_0_^_vp_&_titlebar_item_left_back/thanos_title_bar/content[01]/_^_vi_&_titlebar_item_left_back_^_vc_&_确定`。

## 3 回放

回放是一个方向操作，上面我们记录了用户行为操作记录之后，可以序列化为json，上传到服务端。

回放端根据用户信息，时间筛选出一定时间范围的行为记录，然后在本地一条一条执行。

### 3.1 执行动作序列

滴滴的行为采集是有时间点的，但是回放的时候，是定时1s回放一个操作，这点其实不能很好还原用户操作的心理活动，可能用户很犹豫，找了很久才知道点击哪里。

``

```
case 1:
    mCurrentIndex++;
    postDelayed(new Runnable() {
        @Override
        public void run() {
            next(0);
        }
    }, 1000);
    break;
```



### 3.2 执行点击操作

view组件的查找和在生成时候的规则一样，只不过是反过来，找到控件之后，先构造MotionEvent，然后手动触发view.onTouchEvent(event)即可。

```java
public static void simulateClick(View view) {
    int[] outLocation = new int[2];
    view.getLocationOnScreen(outLocation);
    float x = outLocation[0] + view.getWidth() / 2;
    float y = outLocation[1] + view.getHeight() / 2;
    long downTime = SystemClock.uptimeMillis();
    final MotionEvent downEvent = MotionEvent.obtain(downTime, downTime, MotionEvent.ACTION_DOWN, x, y, 0);
    final MotionEvent upEvent = MotionEvent.obtain(downTime + 100, downTime + 100, MotionEvent.ACTION_UP, x, y, 0);
    view.onTouchEvent(downEvent);
    view.onTouchEvent(upEvent);
    downEvent.recycle();
    upEvent.recycle();
}
```

# 结论

滴滴棱镜最大的亮点是通过hook WindowManagerGlobal，自动监听窗口事件，自动采集数据。但是在回放端，没有跟进真实时间回放，整体方案还是有一定创新的。

类比H5的rrweb方案，不够轻量，因为滴滴棱镜需要在手机上运行APP来回放，成本比较高，不能直接在浏览器回放。业界也有公司在模拟器上回放，然后模拟器推流，在浏览器观看回放，但是模拟器的成本任然很好。

我想到还有一种方案，是把android view也用json描述，首次全量view快照，后续增量快照，记录每次页面view的diff，通过json上报给后端。同时记录用户操作轨迹（回放用于观察用户行为），其实rrweb也是这么做的，1个小时的用户数据也不大，不到1MB,没有任何环境依赖，直接回放。

rrweb因为已经有专业的团队做了开源，所以H5集成成本很低。android或者ios目前没有类似的方案，从开头成本巨大，也希望有人能试一试这个方案。

# 参考文献

[DiDiPrism](https://github.com/didi/DiDiPrism)
