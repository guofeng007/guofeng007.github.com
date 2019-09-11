---
layout: post
title: InstantApp 开发详解
categories: Blog
description:   InstantApp 详解
keywords:      DeepLink、 AppLink、InstantApp

---

# InstantApp 开发详解

参见 [Instant APP](https://segmentfault.com/a/1190000011648067),

# 1.Instant App程序的结构及概念

在之前的课程我们有介绍，要想进行Instant App的程序开发，必须基于Android Studio 3.0集成开发环境。故后面介绍的所有关于代码的书写，以及在集成开发环境内进行程序架构，本人均在Android Studio 3.0 preview版进行（因为写这文章的时候，最新版也就是preview版）。 -_-||

在Instant App内，有几个非常重要的概念。比如feature、feature modules、feature APK。在做之前，必须先搞懂这几个概念，才能做好你的Instant App程序开发。

瞬时加载程序应该起到的作用是，当你在需要一个功能需求时，从Google Play上可以下载这个程序的部分功能，从而体验到App级的用户体验，用户使用完这个应用的功能模块后，系统会丢掉这个功能模块的代码。不难发现，这个功能，肯定和完整App有着某种密切的联系。那么是什么联系呢？

好的，我们开始根据官方的图来进行程序架构分析。想要分析程序的结构，应从大到小、从外向内进行分析。我们会从下面两个方向进行分析，也就是从外到内。

- 从程序功能划分程序结构
- 从功能结构划分工程架构

## 1.1 从程序功能划分程序结构

如果你的应用程序是带有即时应用的话，那么你在构建你的项目之时，会带有一个或多个即时应用程序APK。这是根据你的程序划分程序功能情况而定，每个功能，可以生成对应功能的即时应用程序APK。

好，明白了这个功能APK后，我们再来看看它是怎么架构的。

我们先来从官方的一张图说起：

![img](https://segmentfault.com/img/remote/1460000011648070?w=470&h=334)

从图中我们可知，一个Instant App APK程序内，只有一个Base feature APK，而可以有多个feature APK来构成。换句话说，每个即时应用程序，都会有且只有一个基础功能APK。

换个角度想，如果你的即时应用程序只有一个功能，那么你只需有一个基础功能APK就够了；如果你的即时应用程序有多个功能，那么你需要一个基础功能APK，它包含其他功能所需要用到的共有数据即可，比如不同功能间，界面内会有一些公用的控件、基本信息等，那么这些共有数据就可以放在基础功能APK内，而其他功能APK，具备不同功能即可。

还有另外一种情况。用户已经体验了一个功能了，系统已为用户下载了基础功能APK以及功能APK，如果需要在这个功能内请求数据到另外一个功能内的情况下，系统只会为你下载另一项功能的代码，因为这是基础功能的代码已经存在本地了，无需再次下载了。

怎么样，Google大大的这个设计，是不是感觉简直逆天到爆？总结一句话就是，需要的就下载；不需要的，不下载。这为我们在一定程度上节省了很多宝贵的流量，也减少应用程序所占用的系统空间。（土豪请随意）🙄

## 1.2 从功能结构划分工程架构

从上面的小结我们可以知道，你的程序其实是按照功能的不同进行区分的，所有功能应有一个基础功能，在基础功能之上，将整个程序划分出不同的功能。那么不同的功能之间，该如何进行代码模块的划分架构呢？

我们再来看下官方发布的另外一张结构图：

![img](https://segmentfault.com/img/remote/1460000011648071?w=470&h=249)

从上图我们很清楚的知道，一个Instant App 程序内，只会包含一个基本的功能，而自定义的模块，会依赖于基本的功能块。这是一个很典型的瞬时加载程序的一个单一功能模块的程序架构。而`Instant app module`是瞬时加载程序的入口点，`App module`是功能程序的完整代码部分。

好的，我们了解了模块该如何划分后，就可以具体来看下，划分模块时需要配置的详细代码了。（不要跟我说看代码头疼，我们都是程序员👨‍💻‍）

# 2.构建单个功能模块的Instant App

想要构建单个功能模块，我们假如按照最简单的结构，可以分为如下：

- Base features module
- Android Instant Apps module
- App module (APK)

这三个方面足以说明一个简单的Instant App结构了。下面我们来逐个详细的了解。

## 2.1 按模块划分

### 2.1.1 Base features module

Base Features module的说明，我们可以从两方面谈起。

- manifest文件的修改

在AndroidManifest.xml文件中，你需要修改`application`标签的内容。像如下内容：

```
<application>
        <activity android:name=".MainActivity">
            <intent-filter android:order="1">
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.BROWSABLE" />
                <category android:name="android.intent.category.DEFAULT" />
                <data android:host="myfirstinstantapp.doncc.org"
                    android:path="/"
                    android:scheme="https" />
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
</application>
```

在这里，我们需要修改的东西会多一些。如果细心的同学会发现，其实修改的内容，和我们之前接触的App Links的配置是一致的。这就应了我们之前所述的，Instant App的实现是依赖于App Links的。固然在配置上也是一致的。

这到底是为什么呢？

别忘了，我们根据之前的架构可知，base features module是所有module的基础。也就是说，当系统加载程序时，首先加载的就是这个base features module，那么固然基础信息也就被系统读取到了。

所以你也就可以这么修改你的程序。当你需要一些基础信息，比如`<activity>`、`uses-permission`等基础信息，可以在base feature module的`manifest`文件中进行声明，作为基础需要的资源。

或许，你应该懂得，不是整个程序的基础信息，都必须加载进base feature module的`manifest`中，应是需要的就加载，不需要的就不加载。

- build.gradle的修改

  这里指的gradle文件，是base feature的gradle配置文件。你可以在里面找到`android`的相关配置。在内部，添加`baseFeature true`这样的配置即可。比如下面的代码所示：

```
apply plugin: 'com.android.feature'
    android {
          ...
        //add this line
        baseFeature true
          ...
        defaultConfig{
            //delete applicationId     
            ...
        }
    }
```

这样系统在读取gradle配置信息时，会知道这个模块是属于base feature，就会继续找它相应的子模块。你看Google大大已经封装得多么简洁了，真心爽啊。

### 2.1.2 Android Instant Apps module

在这Instant App模块内，情况有些特殊。这个模块内不包含任何的代码，只包含有构建信息的配置。比如下面的`build.gradle`文件配置：

```
apply plugin: 'com.android.instantapp'
...
dependencies {
    implementation project(':base')
}
```

在这里，我们需要把`apply plugin`这个配置改为`'com.android.instantapp'`，这是告知构建器本模块是Instant App模块。并且在`dependencies`模块内，需要指定Instant App模块是实现自谁，也就是从结构上它是谁的子结构。关于结构是怎么分，还需读懂上面的【图FeaturesSingle.png】为妙。

在这个模块内，你可以删除掉这个模块代码的`src`包，因为这个模块内，没有代码，所以即使添加`src`包也是没用的。故可以将其删除。

### 2.1.3 App module (APK)

在这个模块里，如果你需要构建你的Instant App的话，那么这个模块需要包含要所有功能的模块内容和产品的APK。在这个模块内也是比较特殊的 ， `manifest`文件内不包含除`<manifest>`以外的其他配置标签，因为有关于`application`、`activity`、`uses-permission`等配置信息，已经在base feature module中配置了，所以不用再进行配置。

这里我们也有个配置样例，可参考如下：

```
<manifest
    xmlns:android="http://schemas.android.com/apk/res/android"
    package="org.doncc.instantapp">
  <!--remove application-->
</manifest>
```

在上述代码中，`manifest`内的`package`值，要和你的base feature module所配置的包名保持一致。并且你需要移除掉生成的`<application>`标签。

而在build.gradle文件内，我们也需要进行一些修改：

```
apply plugin: 'com.android.application'
...
dependencies {
    implementation project(':base')
}
```

不难发现，如果我们需要构建一个完整的Instant App，我们需要在这个模块内添加结构是实现自谁。根据上面的【图FeaturesSingle.png】可知，我们这个模块是实现自base模块，所以我们需要在build.gradle内填写实现自base模块的语句配置。

## 2.2 总结

在本章内，我们知道了如何将一个已有的项目，变为Instant App程序架构。其最主要的思想，我们可以总结如下：

- 抽象父feature module：

  是将程序架构成拥有一个最顶级的feature module，这个feature module内集合你的Instant App或者其他子module在运行时需要的一些必要资源内容，包括Activity、Fragment等资源。这样能相对减少你的子module的资源大小，从而减少你宝贵的流量消耗。

- APK module及Instant App module，共同构成base feature module：

  同第一条总结的，当你抽象出base级的feature后，他们构成总体的结构，包括apk module及instantapp module。当然这不仅限于这两个！

- Instant App应使用App Links作为功能的接入口：

  因为App Links具有独特的验证方式，能与Google服务进行互通，且瞬时加载程序是依托于Chrome浏览器来进行交互的。所以需要使用App Links作为功能的接入口。如何验证App Links的重要性也就不言而喻了。如果需要回顾这块儿的课程，请翻阅之前我写过的文章，在那里我有详细介绍有关于App Links的使用。

# 3.多个feature构建你的Instant App

如果想了解多个feature是如何工作的，那么你首先、务必、一定要理解上个章节的内容，也就是单个功能模块是如何工作的，只有这样你才能很快的理解本章节内容。

## 3.1 按模块划分

多个feature工作起来其实并不复杂，原理跟单个feature其实是一样的。这里我依照贴心的放上google的官方图片来解释：

![img](https://segmentfault.com/img/remote/1460000011648072?w=470&h=453)

有人会说，为什么你只会引官方的图，而不自己做图呢。我想说，官方的图已经很简明扼要的阐述了内容，我们为什么还要重复造轮子呢？

好了废话不多说，来看图说话。看上去连线之间交叉复杂，但是这里面包含很清晰的逻辑关系。别急，跟着我的思路，一点一点理解上面的图。

第一，我们抛开浅绿色以上的图先不看，我们只看浅蓝色以下的部分。一个完整的Instant App，依旧有一个Base Feature，那么它可以引伸出两个子feature，分别是Feature 1、Feature 2。这是什么意思呢？这和我们之前讲过的内容正好相匹配上，一个父级的Feature，可能包含很多子级的Feature，而这些诸多的子Feature共同构成了一个完整的App功能。这也就是Instant App架构的精髓，把一个大的功能完全拆分成不同小部分的小功能，从而减少每个功能块的代码量的大小。

第二，我们再来看浅绿色的区域。我们会发现，Instant App module分别指向了Feature 1和Feature 2，并且App module也分别指向了Feature 1和Feature 2。这里有什么🐈腻么？

还记得App Links的特点么，在你要链接到的Activity，会在manifest文件处进行App Links的配置，链接到你想要访问的Activity。而Instant App恰恰就是运用App Links的机制。也就是说，从Instant App程序入口的操作，会在内部识别你请求的到底是哪个Activity，也就是哪个功能Feature。这样你才会看到，浅绿色的Instant App module分别指向了Feature 1和Feature 2。

**第三，我们要格外注意的是，浅绿色的Instant app module和app module需要同时实现自深蓝色的Base Feature，这需要在gradle配置文件内进行额外配置。图中并没有进行描述。这一点需要格外注意。**

## 3.2 总结

好了，这次我们就很容易的理解多个Feature是如何构架你的程序的。思路也不是很复杂，我们总结如下：

- 将你的程序的大功能模块，分为若干个不同的小feature module。**注意，我们之前提到过的不要忘记，每个feature module，尽量不要超过4MB大小。**当然这仅仅是建议，尽量去满足他。
- Instant App module、app module，要实现自每个feature，这样才能让App Links找到不同功能的Feature module。
- 你的instant app module和app module（假设你有这两个模块作为实现feature的子模块），那么你需要分别实现自base feature module。这样你的子feature才能生效。

本文着重对Instant App程序架构的阐述，从基本的架构概念，到单个功能模块的架构，再到复杂的多功能模块架构，很详细的说明了其结构上的关联性，能让你更轻松的理解Instant App程序开发。

不难看出，其实如果搞懂了Instant App程序架构，那么你在进行程序开发时就会变得易如反掌了。相信看完这篇文章后，后续的课程对于聪明的你来讲，简直可以轻松驾驭。

至此，关于Instant App程序开发，我们已经摸清了头绪。我们可以很轻松的驾驭它，并且应用到自己的项目中了。