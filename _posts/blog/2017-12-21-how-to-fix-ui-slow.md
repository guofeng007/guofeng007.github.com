---
layout: post
title: Android UI 性能优化
categories: Blog
description: Android UI 性能优化
keywords: Android、性能、优化

---



# Android UI 性能优化

 

**转载**
[原文](https://juejin.im/entry/5a3b53ce518825772a4b2846?utm_source=gold_browser_extension)
[译文](https://developer.android.com/topic/performance/vitals/render.html)

本文来源于Google IO 2017上的视频  [Android Performance: UI](https://developer.android.com/topic/performance/vitals/render.html)  ，翻译自Android官网；个人觉得非常有价值，比如指出 **对象分配**、**垃圾回收(GC)**、**线程调度**以及**Binder调用** 是Android系统中常见的卡顿原因，更重要的是给出了定位和解决这些问题的方案；而非简单地告诉你避免对象分配，减少布局层级，减少过度绘制等苍白无力的内容。另外，Google开发团队在各个不同场合不厌其烦地提到了 **Systrace**用以解决App中不同维度的问题，这是一个远被低估的强大的工具。希望对大家有帮助 ^_^
水平有限，翻译不妥之处请多多指教。

-----------------正文的分割线

UI渲染是指从App生成帧并显示在屏幕上的行为。为了保证App用户体验的流畅性，App需要在16ms内渲染完一帧以达到60fps的帧率（[为什么是60fps?](https://www.youtube.com/watch?v=CaMTIgxCSqU)。如果你的App UI渲染缓慢，那么系统会强制跳过某些帧，用户就会感知到app的“口吃”，也就是卡顿。
（下面三段可以认为是Google的广告，与性能优化无关）为了帮助开发者提高App的质量，Android自动监控了App的卡顿并且把信息展示在Android Vitals dashboard上。如果对这些信息是如何收集的感兴趣，参考 [Play Console docs**](https://support.google.com/googleplay/android-developer/answer/7385505)。
如果你的app有卡顿的情况，Android Vitals dashboard这个页面提供了诊断和解决问题的指引。

> PS：Android Vitals dashboard 和Android系统记录了使用UI Toolkit（App中从Canvas和View继承树绘制出来的对用户可见的部分）的渲染时间统计。如果你的App没有使用UI Toolkit，比如有的app使用`Vulkan`, `Unity`, `Unreal`或者 `OpenGL`，那么在Android Vitals dashboard中是无法看到渲染时间统计的，可以通过`adb shell dumpsys gfxinfo <package name>`来确定设备是否对某个app记录了这些信息。

**定位卡顿**

精准地定位App中引起卡顿问题的代码是非常困难的，本小结介绍一些定位卡顿问题的方法：

- 直观分析
- Systrace
- 定制性能监控工具

**直观分析**可以让你在短时间内查看整个App的卡顿情况，但是它不像Systrace可以提供更多卡顿的细节。**Systrace**可以提供足够的信息，但如果对App的所有使用场景运用Systrace分析，你会被大量的数据淹没以至于难以分析问题。直观分析和Systrace都可以在你的本地设备上检测卡顿问题，但如果没办法在本地设备上重现卡顿问题，你可以构建自定义的性能监控工具，以便在线上运行的设备上测量App特定部分的性能。

**直观分析**

直观分析可以帮助你定位App使用场景中产生卡顿的地方。你可以简单地打开App然后使用它的各个功能来查看界面是否卡顿。以下是做直观分析时候的一些建议：

- 使用release版本（至少是非debuggable）的App。ART运行时为了支持debug的某些特性在debug情况下去掉了好几个非常重要的性能优化点；因此要确保你分析的App是和用户使用接近的版本。
- 打开 [Profile GPU Rendering**](https://developer.android.com/studio/profile/inspect-gpu-rendering.html#profile_rendering)  。 **Profile GPU Rendering**在屏幕上显示了各种图表，可以帮助你直观地看到绘制UI窗口的每一帧相对16ms每帧的标准花费了多长时间。每个显示栏有各个不同颜色的组件，每个组件都被映射到渲染pipeline的某个阶段，因此你可以看到哪一部分花费了最长的时间。例如，如果某一帧在处理输入的时候花费了较长时间，那你就应该查看一下你的代码里面处理用户输入的部分。
- 某些特定的组件，比如 [RecyclerView**](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html)，它们是常见的卡顿根源，如果你的App使用了这些组件，最好分析使用了这些组件的部分。
- 尽量使用较慢的设备来恶化卡顿问题以便分析。

一旦发现了产生卡顿的场景，或许你已经知道了造成卡顿的原因，但如果你需要更详细的信息来分析问题，可以借助Systrace。

**使用Systrace**

虽然Systrace是展示整个设备在干什么的工具，它对定位卡顿问题也非常有帮助。Systrace有着非常小的运行时开销，因此你在分析问题的时候可以体验到真实的卡顿。

使用Systrace来记录App卡顿场景下的trace（可以通过 [Systrace WalkThrough**](https://developer.android.com/studio/command-line/systrace.html) 来查看如何做）。systrace的图表被进程和线程分为若干个部分，你的app的trace结果大致长这样：

![img](https://user-gold-cdn.xitu.io/2017/12/21/16077bf786241587?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

上图所示的systrace包含着可以定位卡顿的如下信息：

1. Systrace显示了每一帧绘制的时间段，并且给每一帧都有不同颜色，可以突出较慢的渲染帧时间。与直观分析相比，这可以帮助你更精确地找到单独的卡顿的某一帧。更详细的内容可以参考 [Inspecting Frames**](https://developer.android.com/studio/command-line/systrace.html#frames)。
2. Systrace会监测你App中的问题并会在单独帧和警告栏里面展示警告提示信息；跟着这些提示的指引来分析问题是最好的选择。
3. 某些 Android 框架和库，比如 [RecyclerView](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html)有自定义的trace标记，因此systrace的timeline会展示这些方法在何时执行以及执行了多长时间。

在查看了systrace的输出结果之后，你可能会发现某些可疑的造成卡顿的方法。比如：如果timeline显示某一帧过慢是由RecyclerView引起的，你可以给相关代码 [添加Trace标记**](https://developer.android.com/studio/command-line/systrace.html#app-trace)，然后重新运行systrace来获取更多的信息。新版的systrace的timeline会展示你代码里面这些方法的调用的时机以及耗费的时间。

如果没有从systrace中找到为什么UI线程执行较长时间的细节，那么你可能需要使用   [Android CPU Profiler](https://developer.android.com/studio/profile/cpu-profiler.html#method_traces)   来记录采样或者插桩的method trace。不过通常情况下，method trace不适合用来定位卡顿问题，因为它们运行时的开销太高可能会造成误报，并且它无法看到线程是在运行还是处于阻塞状态。但是，method trace可以帮助你定位代码中耗时长的方法；在定位到耗时方法之后，可以 [添加Trace标记](https://developer.android.com/studio/command-line/systrace.html#app-trace) 然后重新运行systrace来查看是否是因为这些方法引起的卡顿。

> PS：当纪录systrace的时候，每一个trace标记（一个开始和结束标记对）会带来10纳秒的开销，为了避卡顿误报，不要在一帧内被调用很多次的方法里面添加trace标记，也不要在调用耗时200纳秒以下的方法里面添加标记。

要了解更详细的信息，可以参阅 [Understanding Systrace**](https://source.android.com/devices/tech/debug/systrace)。

**定制性能监控工具**

如果你无法在本地设备上重现卡顿问题，可以在App内构建自定义的性能监控工具，通过线上设备来帮助定位卡顿问题。

要定制性能监控工具，可以通过   [FrameMetricsAggregator](https://developer.android.com/reference/android/support/v4/app/FrameMetricsAggregator.html)   来收集App某个特定部分的帧渲染时间，然后通过   [Firebase Preformance Monitoring**](http://link.zhihu.com/?target=https%3A//firebase.google.com/docs/perf-mon/)   来记录和分析数据。

更详细的内容参阅 [Use Firebase Performance Monitoring with Android Vitals**](https://firebase.google.com/docs/perf-mon/get-started-android#pdc)。

## **修复卡顿**

要修复卡顿问题，首先查看那些没有在16.7ms内完成的帧，然后查看造成这个的原因在哪。是因为View#draw反常地花费了较长时间，又或者是布局过程耗时？详细介绍见下文的**常见卡顿原因**。

要避免卡顿问题，耗时较长的任务应该在UI线程之外异步完成；因此需要时刻注意你的代码运行在哪个线程，并且在post不重要的任务到主线程的时候保持谨慎。

如果你的App有一个复杂而重要的UI，可以考虑   [writing instrumentation tests](https://developer.android.com/training/testing/performance.html#automate)   来自动监测较慢的渲染时间，然后定期运行测试case来避免问题复发。更多内容见   [Automated Performance Testing Codelab**](https://codelabs.developers.google.com/codelabs/android-perf-testing/index.html)。

## **常见的卡顿原因**

下面的小结将介绍一些App中常见卡顿的原因，并提供一些定位它们的最佳实践。

## **滚动列表**

[ListView**](https://developer.android.com/reference/android/widget/ListView.html) ，特别是   [RecyclerView](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html) 被广泛用于复杂的滚动列表里面，它们是最容易导致卡顿的部分。这两个控件内部都添加了Systrace标记，因此你可以借助systrace来分析它们是否造成了app的卡顿。在获取RecyclerView以及你自己添加的systrace标记的时候，必须要给systrace传递 `-a your-package-name `，不然就不会输出这些标记的信息。在systrace里面，你可以点击RecyclerView的相应标记来看RecyclerView当时在干什么。

**RecyclerView:notifyDataSetChanged**

如果你观察到在某一帧内RecyclerView中的每个item都被重新绑定了（并且因此重新布局和重新绘制），请确保你没有对RecyclerView执行局部更新的时候调用 `notifyDataSetChanged()`, `setAdaper(Adapter)`或者 `swapAdaper(Adaper, boolean)`。这些方法表示**整个列表**内容改变了，并且会在systrace里面显示为 **RV FullInvaludate**。在内容改变或者添加内容的时候应该使用 [SortedList](https://developer.android.com/reference/android/support/v7/util/SortedList.html)   或者   [DiffUtil](https://developer.android.com/reference/android/support/v7/util/DiffUtil.html) 生成更小的更新操作。

例如，如果app从服务端收到了新的新闻列表消息，当你把信息post到Adapter的时候，可以像下面这样使用`notifyDataSetChanged()`:

```
void onNewDataArrived(List<News> news) {
    myAdapter.setNews(news);
    myAdapter.notifyDataSetChanged();
}

```

但是这么做有个严重的缺陷——如果这是个微不足道的列表更新（也许是在顶部加一条），RecyclerView并不知道这个信息——RecyclerView被告知丢掉它所有item缓存的状态，并且需要重新绑定所有东西。

更可取的是使用 [DiffUtil**](https://developer.android.com/reference/android/support/v7/util/DiffUtil.html)，它可以帮你计算和分发细小的更新操作：

```
void onNewDataArrived(List<News> news) {
    List<News> oldNews = myAdapter.getItems();
    DiffResult result = DiffUtil.calculateDiff(new MyCallback(oldNews, news));
    myAdapter.setNews(news);
    result.dispatchUpdatesTo(myAdapter);

```

只需要自定义一个 [DiffUtil.Callback](https://developer.android.com/reference/android/support/v7/util/DiffUtil.Callback.html) 实现类告诉DiffUtil如何分析你的item，DiffUtil就能自动帮你完成其他的所有事情。

**RecyclerView:嵌套RecyclerViews**

嵌套RecyclerView是非常常见的事，特别是一个垂直列表里面有一个水平滚动列表的时候（比如Google Play store的主页）。如果你第一次往下滚动页面的时候，发现有很多内部的item执行inflate操作，那可能就需要检查你是否在内部（水平）RecyclerView之间共享了 [RecyclerView.RecyclerViewPoo](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.RecycledViewPool.html) 了。默认情况下，每个RecyclerView有自己堵路的item池。在屏幕上有十几个itemViews的情况下，如果所有的行都显示相似的View类型，而itemViews不能被不同的水平列表共享，那就是有问题的。

```
class OuterAdapter extends RecyclerView.Adapter<OuterAdapter.ViewHolder> {
    RecyclerView.RecycledViewPool mSharedPool = new RecyclerView.RecycledViewPool();

    ...

    @Override
    public void onCreateViewHolder(ViewGroup parent, int viewType) {
        // inflate inner item, find innerRecyclerView by ID…
        LinearLayoutManager innerLLM = new LinearLayoutManager(parent.getContext(),
                LinearLayoutManager.HORIZONTAL);
        innerRv.setLayoutManager(innerLLM);
        innerRv.setRecycledViewPool(mSharedPool);
        return new OuterAdapter.ViewHolder(innerRv);

    }
    ...

```

如果你想进行进一步的优化，可以对内部RecyclerView的LinearLayout调用 [setInitialPrefetchItemCount(int)**](https://developer.android.com/reference/android/support/v7/widget/LinearLayoutManager.html#setInitialPrefetchItemCount(int))。比如如果你在每一行都是展示三个半item，可以调用 `innerLLM.setInitialItemsPrefetchCount(4);` 这样当水平列表将要展示在屏幕上的时候，如果UI线程有空闲时间，RecyclerView会尝试在内部预先把这几个item取出来。

**RecyclerView:Too much inflation/Create taking too long**

通过在UI线程空闲的时候提前完成任务，RecyclerView的prefetch可以帮助解决大多数情况下inflate的耗时问题。如果你在某一帧内看到inflate过程（并且不在**RV Prefectch**标记内），请确保你是在最近的设备上（prefect特性现在只支持Android 5.0，API 21

以上的设备）进行测试的，并且使用了较新版本的 [Support Library**](https://developer.android.com/topic/libraries/support-library/index.html)。

如果你在item显示在屏幕上的时候频繁观察到inflate造成卡顿，需要验证一下你是否使用了额外的你不需要的View类型。RecyclerView内容的View类型越少，在新item显示的时候需要的inflation越少。在可能的情况下，可以合并合理的View类型——如果不同类型之间仅仅有图表，颜色和少许文字不同，你可以在bind的时候动态改变它们，来避免inflate过程。（同时也可以减少内存占用）

如果view的类型是合理的，那么就尝试减少inflation耗费的时间。减少不必要的容器类ViewGroup或者用来View结构——可以考虑使用 [ConstrainLayout**](https://developer.android.com/training/constraint-layout/index.html)，它可以有效减少View结构。如果还需要优化性能，并且你item的view继承树比较简单而且不需要复杂的theme和style，可以考虑自己调用构造函数（不使用xml）——虽然通常失去XML的简单和特性是不值的。

**RecyclerView:Bind taking too long**

绑定过程(也就是 [onBindViewHolder(VH, int)**](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.Adapter.html#onBindViewHolder)    应该是非常简单的，除了及其复杂的item，其他所有的item的绑定过程耗时应该远小于1毫秒。onBinderViewHolder应该简单地从adapter里取出POJO对象，然后对ViewHolder里面的View调用setter方法。如果 **RV OnBindView**耗费了较长时间，请验证一下是否在绑定的代码里面做了别的工作。

如果你在adapter里使用简单的POJO对象，那你完全可以借助 [Data Binding**](https://developer.android.com/topic/libraries/data-binding/index.html) 库来避免在onBindViewHolder里面写绑定代码。

**RecyclerView or ListView:layout/draw taking too long**

对于draw和layout造成的问题，查看下文的 **布局性能 **和 **渲染性能**。

**ListView:Inflation**

ListView中的View复用机制很容易被偶然破坏，如果你看到ListView的每个Item出现在屏幕上的时候都触发了inflate过程，必须要检查你的[Adapter.getView()](https://link.juejin.im/?target=http%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%253A%2F%2Fdeveloper.android.com%2Freference%2Fandroid%2Fwidget%2FAdapter.html%2523getView)是否使用、重新绑定并且返回了`convertView`这个参数。如果你的`getView()`实现每次都inflate，那就没法享受ListView的View复用机制。`getView()`方法的结构应该永远是下面这个样子：

```
view getView(int position, View convertView, ViewGroup parent) {

    if (convertView == null) {
        // only inflate if no convertView passed
        convertView = mLayoutInflater.inflate(R.layout.my_layout, parent, false)
    }
    // … bind content from position to convertView …
    return convertView;
}

```

## 布局性能

如果Systrace显示Layout段的 **Choreographer#doFrame **做了大量的工作，或者执行得太频繁，那么你可能遇到了布局性能问题。App的布局性能取决于View继承树的哪一部分改变了布局参数或者输入。

**布局性能：耗时**

如果布局的每一段都要花费数毫秒，那么可能是嵌套 [RelativeLayout](https://developer.android.com/guide/topics/ui/layout/relative.html)   或者 [带weight的LinearLayout**](https://developer.android.com/guide/topics/ui/layout/linear.html#Weight) 造成的。这些类型的布局都可能触发子View的多次测量/布局过程，导致嵌套这些布局可能会造成布局时间的时间复杂度为O(2^n)（n为嵌套深度）。因此，需要避免使用RelativeLayout或者带weight的LinearLayout，除非它们是View树的叶子节点。有几个方式可以做到这一点：

- 重新组织布局结构
- 自定义布局逻辑，详情可见 [优化布局**](https://developer.android.com/training/improving-layouts/optimizing-layout.html)。
- 尝试将布局转换为 [ConstraintLayout**](https://developer.android.com/training/constraint-layout/index.html)，它可以提供类似的特性，但是没有性能问题。

**布局性能：频率**

布局过程通常在新内容出现在屏幕上的时候出现，比如RecyclerView中的某个Item滚动到屏幕可见区域上。如果某个重要的布局在每一帧上都执行了layout过程，那可能是你在移动整个布局，而这通常会引发掉帧。通常情况下，动画应该操作View的绘制属性（比如setTranslationX/Y/Z, setRotation, setAlpha)，这些操作比改变View的布局属性（padding，或者margin）要廉价得多。通过invalidate()进而在下一帧触发 draw(Canvas) 会在View被invalidated的时候重新记录绘制操作，这个过程通常也比layout廉价得多。

## **渲染性能**

Android UI 绘制工作分为两个阶段：运行在在UI线程的 `View#draw`，以及在RenderThread里执行的`DrawFrame`。第一个阶段会执行被标记为invalidated的View的 `draw(Canvas)` 方法，这个方法通常会调用很多你的App代码里面的自定义View的相关方法；第二个阶段发生在native线程RenderThread里面，它会基于第一阶段View#draw的调用来执行相应的操作。

**渲染性能：UI线程**

如果 `View#draw` 调用花费了较长时间，常见的一种情况是在UI线程在绘制一个Bitmap。绘制Bitmap会使用CPU渲染，因此需要尽量避免。你可以通过 [Android CPU Profiler**](https://developer.android.com/studio/profile/cpu-profiler.html#method_traces) 用method tracing来确定是否是这个原因。

通常情况下绘制Bitmap是因为我们想给Bitmap加一个装饰效果，比如圆角：

```
Canvas bitmapCanvas = new Canvas(roundedOutputBitmap);
Paint paint = new Paint();
paint.setAntiAlias(true);
// draw a round rect to define shape:
bitmapCanvas.drawRoundRect(0, 0,
        roundedOutputBitmap.getWidth(), roundedOutputBitmap.getHeight(), 20, 20, paint);
paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.MULTIPLY));
// multiply content on top, to make it rounded
bitmapCanvas.drawBitmap(sourceBitmap, 0, 0, paint);
bitmapCanvas.setBitmap(null);
// now roundedOutputBitmap has sourceBitmap inside, but as a circle

```

如果你的UI线程做的是这种工作，你可以在一个后台线程里面完成解码然后在UI线程绘制。在某些情况下（比如本例），甚至可以直接在draw的时候完成，比如如果你的代码长这样：

```
void setBitmap(Bitmap bitmap) {
    mBitmap = bitmap;
    invalidate();
}

void onDraw(Canvas canvas) {
    canvas.drawBitmap(mBitmap, null);
}

```

可以用如下的代码来代替：

```
void setBitmap(Bitmap bitmap) {
    mShaderPaint.setShader(
            new BitmapShader(bitmap, TileMode.CLAMP, TileMode.CLAMP));
    invalidate();
}

void onDraw(Canvas canvas) {
    canvas.drawRoundRect(0, 0, mWidth, mHeight, 20, 20, mShaderPaint);
}

```

要注意的是，上述操作也适用于 background protection（在Bitmap上绘制一个渐变）和 image filtering （用 [ColorMatrixColorFilter**](https://developer.android.com/reference/android/graphics/ColorMatrixColorFilter.html) )这两个对Bitmap的常见操作。

如果你是因为别的原因而绘制Bitmap，或许你可以使用缓存，尝试在支持硬件加速的Canvas上直接绘制，或必要的时候调用 [setLayerType](https://developer.android.com/reference/android/view/View.html#setLayerType)   设置Canvas 为 [LAYER_TYPE_HARDWARE**](https://developer.android.com/reference/android/view/View.html#LAYER_TYPE_HARDWARE)  来缓存复杂的渲染输出，这样也可以享受GPU渲染的优势。

**渲染性能：RenderThread**

某些Canvas操作在UI线程是非常廉价的，但却会在RenderThead触发大量昂贵的计算操作。通常Systrace会给这些调用给出警告提示。

**Canvas.saveLayer()**

要尽量避免 [Cavas.saveLayer()](https://link.juejin.im/?target=http%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%253A%2F%2Fdeveloper.android.com%2Freference%2Fandroid%2Fgraphics%2FCanvas.html%2523saveLayer) 调用，这个方法会在每一帧触发昂贵、未被缓存的离屏渲染。虽然在Android 6.0上优化了这个操作的性能（避免了GPU上的渲染目标切换），仍然需要尽可能地避免调用这个方法；如果实在需要调用它，确保给它传递[CLIP_TO_LAYER_SAVE_FLAG**](https://developer.android.com/reference/android/graphics/Canvas.html#CLIP_TO_LAYER_SAVE_FLAG)。

**Animating large Paths**

如果在一个支持硬件加速的Canvas上调用 [Canvas.drawPath()](https://link.juejin.im/?target=http%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%253A%2F%2Fdeveloper.android.com%2Freference%2Fandroid%2Fgraphics%2FCanvas.html%2523drawPath), 系统会首先在CPU上绘制这些path，然后把它传递给GPU。如果你的path对象很大，那最好避免在每一帧之间修改它，这样path对象就可以被系统缓存起来，使得绘制更加高效。`drawPoints()`, `drawLines()`, `drawRect/Circle/Oval/RoundRect()` 比 `drawPath` 更加高效——因此最好使用它们替代相应的`drawPath`操作，虽然可能用的代码量更多。

**Canvas.clipPath**

[clipPath(Path)**](https://developer.android.com/reference/android/graphics/Canvas.html#clipPath(android.graphics.Path)) 会触发昂贵的裁剪操作，因此也需要尽量避免。在可能的情况下，应该尽量直接绘制出需要的形状，而不是裁剪成相应的图形；这样性能更高，并且支持反锯齿；例如下面这个`clipPath` 操作：

```
canvas.save();
canvas.clipPath(mCirclePath);
canvas.drawBitmap(mBitmap);
canvas.restore();

```

可以用如下代替：

```
// one time init:
mPaint.setShader(new BitmapShader(mBitmap, TileMode.CLAMP, TileMode.CLAMP));
// at draw time:
canvas.drawPath(mCirclePath, mPaint);

```

**Bitmap uploads**

Android的显示系统使用OpenGL，bitmap在底层表现为OpenGL的纹理，因此在bitmap第一次被展示的时候，bitmap会被首先上传的GPU上。Systrace上标记为 **Upload width x height Texture**就是这种情况。这个过程可能会花费数毫秒（如下图），但是这是GPU显示图像的必要过程。

![img](https://user-gold-cdn.xitu.io/2017/12/21/16077bf789d6a532?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> App在上传一个180万像素的bitmap时花费了10ms，要么减少图片的大小，那么使用prepareToDraw提前完成这个操作

如果这个过程花费了较长时间，首先检查在trace里面显示的图片的宽和高，确保图片的大小不比它显示出来的区域大太多，因为这样会浪费上传时间和内存。常见的图片加载库都会提供一个方便的方式来获取和请求一个合适大小的Bitmap。

在Android 7.0上，图片加载代码（通常是图片加载库）可以调用 [prepareToDraw**](https://developer.android.com/reference/android/graphics/Bitmap.html#prepareToDraw()) 在需要的时候提前触发Bitmap上传动作；这种方式可以使Bitmpa在RenderThread空闲的时候提前完成。可以在图片解码之后或者在Bitmap绑定到View上的时候完成这个操作——理想情况下，图片加载库会帮助你完成这些；如果你想要自己掌控图片加载，或者需要确保不在绘制的时候触发Bitmap上传，可以直接在代码里面调用 `prepareToDraw`。

## **线程调度延迟**

线程调度器是Android操作系统的一部分，操作系统用它来决定系统中的线程如何执行、何时执行以及执行多长时间。某些情况下，App卡顿是因为UI线程被阻塞或者没有运行。Systrace用不同的颜色（如下图）来标记某个线程是 Sleep（灰色），Runnable（蓝色：可运行状态，但是调度器没有选择让它运行），Actively running（绿色），或者 Uninterruptible sleep(红色或者橘黄色），这对解决由于线程调度引起的卡顿非常有帮助。

> 老版本的Android系统频繁出现线程调度问题并不是App自己的锅，Android开发团队对这一块进行了持续的改进，因此在debug线程调度的问题的时候，最好使用新版本的Android系统，以确定线程问题确实是App的锅而非系统问题。

![img](https://user-gold-cdn.xitu.io/2017/12/21/16077bf77f033c22?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> 要注意的是，在一帧的某些部分，UI线程或者RenderThread是不期望被运行的。比如，当RenderThread 的`syncFrameState` 执行以及Bitmap上传的时候UI线程处于阻塞状态——这样RenderThead可以安全地从UI线程copy数据。再举个例子：当RenderThread使用IPC（内部进程通信）的时候它自己也可能处于阻塞状态：比如在一帧开始的时候获取buffer，从buffer查询信息，或者通过`eglSwapBuffers` 把buffer回传给合成器。

App执行中的长时间停顿通常情况下是由**Binder调用**（Android中的内部进程通信机制）引起的。在最近的一些Android版本上，Binder调用是UI线程暂停执行最常见的原因之一。一般的解决方案是，避免调用IPC函数，缓存调用值，或者把工作放到后台线程执行。随着代码库越来越大，开发人员很容易就不小心地在某个低层次的方法里面添加了Binder调用的函数——不过我们可以通过tracing很容易滴发现和修复这个问题。

如果app中有binder通信，可以用如下的adb命令来查看调用栈：

```
$ adb shell am trace-ipc start
… use the app - scroll/animate ...
$ adb shell am trace-ipc stop --dump-file /data/local/tmp/ipc-trace.txt
$ adb pull /data/local/tmp/ipc-trace.txt

```

有时候某些看起来无害的的方法（比如 [getRefreshRate()**](https://developer.android.com/reference/android/view/Display.html#getRefreshRate())会触发Binder通信，然后如果它们频繁地被调用就会引发严重的性能问题。周期性地对App进行tracing可以帮助你在问题出现的时候快速地定位和解决它们。

![img](https://user-gold-cdn.xitu.io/2017/12/21/16077bf77c577533?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> 由于RecylerView的fling触发的Binder调用引起UI线程sleeping。保持你的IPC调用方法简单，然后使用trace-ipc来移除不必要的调用

如果你没有发现Binder调用，但是UI线程依然处于没在运行的状态，那可能是因为UI线程在等待其他线程某个操作的**锁**。一贯情况下，UI线程不应该等待其他线程的执行结果——别的线程应该在拿到结果之后post给UI线程。

## **对象分配和垃圾回收**

自从Android引入 ART 并且在Android 5.0上成为默认的运行时之后，对象分配和垃圾回收（GC）造成的卡顿已经显著降低了，但是由于对象分配和GC有额外的开销，它依然又可能使线程负载过重。 在一个调用不频繁的地方（比如按钮点击）分配对象是没有问题的，但如果在在一个被频繁调用的紧密的循环里，就需要避免对象分配来降低GC的压力。

可以通过Systrace来确定是否发生了频繁的GC，然后用 [Android Memory Profier**](https://developer.android.com/studio/profile/memory-profiler.html) 分析分配的对象都是些什么。如果你尽可能地做到了避免分配对象（特别是在紧密的循环里），那就几乎不会遇到这种问题。

![img](https://user-gold-cdn.xitu.io/2017/12/21/16077bf7a7acd977?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> 发生在HeapTaskThread线程上一个耗时94ms的GC

在最近的Android版本上，GC通常运行在一个叫做HeapTaskDaemon的后台线程里面。如上图所示，过多的对象分配意味着CPU将在GC上耗费更多的资源。