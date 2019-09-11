---
layout: post
title: DeepLink AppLink-InstantApp 前奏详解
categories: Blog
description:  DeepLink AppLink-InstantApp 前奏详解
keywords:      DeepLink、 AppLink、InstantApp 前奏详解

---

# DeepLink AppLink-InstantApp 前奏详解

参见 [ DeepLink AppLink](https://segmentfault.com/a/1190000011169174),



# 1.为何引入Deep Links和App Links的概念

肯定有很多童鞋不理解，为什么我要在讲Instant App技术之前，要提及这个“看似不相关”的概念呢？实则真正在进行Instant App程序开发时，是必然涉及到这标题所述的两个概念的。应该先搞清这两个概念的区别，再来进行程序开发，就会变得易如反掌了。

Google为我们提供了一个非常好的操作体验：比如当我的好朋友给我发了一个定位的链接信息，这个信息我们看上去好像不那么直观得看出，我的好朋友在什么具体位置。但是如果我一旦点击了这个链接，如果我直接能在某个地图应用里看到具体图像位置的话，那么这个体验就会变得更好了。

对于一个链接，当点击后，Google为我们预留了两种的处理方式，这就是Deep Links和App Links。这两种方式，在体验上是截然不同的。所以想要采用哪种处理方式，这就取决于我们自己的决定。

注意，我这里只是在处理链接的角度上讲，如何处理是取决于用户自己。但是对于Instant App技术来讲，是要采用App Links的处理方式。所以这里同学们需要注意。

那么好，接下来我要着重讲解Deep Links和App Links这两个处理URL的方式。

# 2.Deep Links的作用

要想明白Instant App中的精髓，首先要了解Deep Links是什么东西，以及其本身的作用。

我们来回想一个场景。假如你的手机内安装了UC浏览器（UC浏览器并没有给我充值，先声明），并且还同时存在其他浏览器，当你在一个应用程序内点击一个链接，系统会弹出一个对话框，说明的是让你选择一个可以打开链接的应用或方式。你可以选择系统浏览器，也可以选择UC浏览器；如果你的手机内只有一个可以打开链接的程序（比如只有一个浏览器程序），那么系统就会用这个应用程序直接打开这个链接。

那么结果来了，Deep Links是可以来让用户，通过一段类似URL的链接，达到快速用某个应用打开的作用。当然这也需要用户确认，到底是用哪个程序打开。

![img](https://segmentfault.com/img/remote/1460000011169955)

好，接下来我们再看下官方对于Deep Links工作的阐述。当用户点击一个链接，或者有一个web URI的请求调用，安卓系统会尝试几个操作，直到操作请求成功：

> 1. Open the user's preferred app that can handle the URI, if one is designated.
> 2. Open the only available app that can handle the URI.
> 3. Allow the user to select an app from a dialog.

将上述的操作放入刚刚我们举得例子里，不难发现，系统内部执行了如下的判断过程：

- 首先系统判断出哪些程序开放了可以处理URL的程序配置；
- 其次来看下是不是只有这一个应用程序能处理这个URL；
- 再次来看下有哪些应用可以处理这个URL；

这个判断的过程，完全有系统来进行。所以要懂得它的判断过程，尤为重要。

## 2.1进行Deep Links的配置

我们来设定一个情景。当我的朋友给我发来一段链接，内容是这样的：

![img](https://segmentfault.com/img/remote/1460000011169956)

那么这里我们要区分出来，我们需要在VideoActivity上呈现。这段链接对于intent-filter过滤器来说，scheme部分为“http”，而host部分为“doncc.org”，考虑到我的程序可能应对这个链接下面很多其他的视频，所以我可以选定pathPrefix的过滤方式，这样能包括其他视频的链接一并过滤出来（当然你也可以要求更为严格的过滤）。

对于支持Deep Links的程序的配置，很是简单。或许我们来段manifest文件的过滤器配置更为直观：

```
<activity android:name=".VideoActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data
            android:host="doncc.org"
            android:pathPrefix="/video"
            android:scheme="http" />
    </intent-filter>
</activity>
```

好，当我们的配置配好之后，就可以将程序运行在模拟器上了。然后将程序退出。

## 2.2进行Deep Links的验证

验证部分需要打开terminal，并输入命令如下：

```
$ adb shell am start
        -W -a android.intent.action.VIEW
        -d <URI> <PACKAGE>
```

注意这里的`<URI>`，指的是你在程序内指定的地址。那么应对上述文中，`<URI>`指定的就是`http://doncc.org/video/1.mp4`（或者尾部为其他的视频资源），而`<PACKAGE>`指的是你的程序的package包名。所以当你输入正确的命令并执行后，你的模拟器就会直接看到执行后的页面。说明此时Deep Links已奏效。

![img](https://static.segmentfault.com/v-5a9fa408/global/img/squares.svg)

特别强调一点。这里我们是直接验证Deep Links是否奏效。然而设置支持此Deep Links的Activity，在第一次呈现时，会有一个选择对话框，因为此Activity添加了其他的过滤器，同样系统内也有跟它一致的过滤器，系统默认是不知道你要用哪个程序打开此链接页面的。所以在提示框内呈现提示用什么应用方式打开。

说了这么多，如果还是没太清楚，可以参考我录制的视频：[Android 如何使用 Deep Links 处理URL链接](http://v.youku.com/v_show/id_XMzAxMjU4NDIwOA==.html?spm=a2hzp.8253869.0.0)

# 3.App Links的作用

我们再来看一个应用场景。

或许我们有过这样的体验。当你的朋友用短信的形式，给你发了一篇知乎的文章链接给你，如果你点击了链接，假设你的手机内没有安装知乎程序，那么手机会自动通过浏览器，显示出来这个文章的页面信息；如果你的手机内安装了知乎的程序，那么会直接打开知乎程序，显示出这个文章的信息。

我们之前知道了Deep Links的作用，实则你可以理解成App Links是一个特殊的Deep Links。因为Deep Links在第一次使用时，是需要用户主动进行授权的，而App Links的作用是无需这第一次的主动授权，是足够信任这个链接地址的。当然这样开放，势必会带来安全的隐患。没有关系，Google大大已经为我们想到了这一点了，在Android 6.0以后，程序可以支持App Links来让程序处理默认指定的链接。

相比Deep Links而言，App Links具有一定的优势的：

- **更安全、更严格：**相比Deep Links我们不难发现，在制定具体你的域名地址时，如果指定的条件过于宽泛，势必会可能其他的链接地址后缀也可以访问到你应用程序。如果过于严格的约束，那么所制定的业务有太过于麻烦。所以App Links的出现，就为我们更好的解决这一问题；
- **更为无缝的用户体验：**当你点击一个链接的时候，可能会有一个URL链接到你的网站。如果你的手机内没有安装这个应用，那么链接可以很轻松的重定向到你的网站，不至于页面会弹出一个404 not found的错误，给用户带来非常不好的体验；
- **可支持Android Instant App瞬时加载技术：**可以支持瞬时加载技术。用户无需安装应用，凭一个URL链接，就可以体验到同App一样的用户体验。这一点也是至关重要的，我们也会在后续课程内详细介绍。
- **可从Google Search中搜到你的应用：**同上条一样，我们可以通过Google Search来找到你的应用。这一点也是对我们Instant App有所帮助的一点。

从诸多关于App Links的优点来看，我们可以总结出，想要掌握Instant App技术，首先必须掌握App Links是如何配置的，且App Links是基于Deep Links的。所以我们就知道了该如何去学习了。

![img](https://segmentfault.com/img/remote/1460000011169958)

好，接下来我们要深入学习关于App Links。

## 3.1App Links的配置

对于支持App Links的程序的配置，要比支持Deep Links的配置稍加复杂一步。总共可分为两个部分：

- 对manifest过滤器的编写
- 数字资产链接文件的生成

接下来我们详细说明每个部分。

### 3.1.1对manifest过滤器的编写

对于过滤器的编写，我们再来一段代码更为直接：

```
<activity android:name=".BlogActivity">
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />

         <category android:name="android.intent.category.DEFAULT" />
             <category android:name="android.intent.category.BROWSABLE" />

             <data
                android:scheme="http"
                android:host="doncc.github.io"
                android:pathPrefix="/2017/" />
    </intent-filter>
</activity>
```

事实上我们不难发现，这个过滤器的编写，要比之前Deep Links的编写内容，只多了一个属性。那就是：

```
<intent-filter android:autoVerify="true">
    ...
</intent-filter>
```

通过`android:autoVerify="true"`这个属性，就能让系统知道，这个程序是属于可以自动进行验证的。也就为后续进行数字资产链接文件的验证做好了铺垫。

### 3.1.2数字资产链接文件的生成

对于数字资产链接，其实是一个json文件，里面包括了一些应用程序的信息，而这个文件需要放置在你的网站的特定路径内。

那么数字资产链接文件该如何写呢？下面来看个例子：

```
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "org.doncc.learnapplinks",
    "sha256_cert_fingerprints":
    ["01:CD:30:56:A8:5E:EC:18:64:3D:8B:F2:AF:3F:B6:02:85:33:79:13:97:23:E3:FF:99:59:14:E5:03:FC:57:78"]
  }
}]
```

从上面的例子可以得出结论，数字资产链接文件配置的要求，有如下内容：

- 需知道你应用的包名（package_name）
- 需知道你发布程序的sha256指纹签名证书（sha256_cert_fingerprints）

对于包名，我们肯定知道该如何获得。而对于sha256指纹签名证书，我们可以通过命令来获取到。比如我发布的程序是用release.keystore来签名的，那么你在用命令行获得sha256签名时，就要指定你的key来获得：

```
$ keytool -list -v -keystore release.keystore
```

当生成完毕后，需要把这个文件重命名为`assetlinks.json`，并上传到你的网站的特定目录内：

```
https://domain.name/.well-known/assetlinks.json
```

其中`domain.name`部分为你的具体的网站地址。而需要在你的网站下面的`.well-known`目录下，新建`assetlinks.json`文件，也就是刚配置过的数字资产链接文件，来确定你的网址是可信的。

如果我们的网站会忽略隐藏文件，那么强烈建议你写一个取消忽略的配置文件。这个过程就不需要我讲了吧。。。

## 3.2App Links的验证

同样，对于App Links的验证，总体也可以分为两部分：

- 对数字资产链接文件配置合法性的验证
- 通过Shell命令验证

同样，我们详细说明每个部分。

### 3.2.1对数字资产链接文件配置合法性的验证

在进行App Links的验证过程要注意一点，根据官方说明，在程序安装完毕后，至少等待20秒后再进行链接的验证是最为好的。并且要保证你的网络足够好，因为在验证过程，系统识别一旦超过5秒，那么就会验证失败，以至于会出现和Deep Links同样的用户效果了。

> **Note:** Make sure you wait at least 20 seconds after installation of your app to allow for the system to complete the verification process.

- 通过网页的验证链接进行验证

我们可以首先来通过网页访问的形式，来确定我们自己的数字资产链接文件是否已经被网站关联上。这里我们给出官方的网页URL：

```
https://digitalassetlinks.googleapis.com/v1/statements:list?
   source.web.site=https://domain.name:optional_port&
   relation=delegate_permission/common.handle_all_urls
```

这里`source.web.site`是需要替换成我们自己的网站地址的。然后点击回车，如果出现类似下面页面，就说明当前数字资产链接文件已经被网站关联上了：

```
{
  "statements": [
    {
      "source": {
        "web": {
          "site": "https://doncc.github.io."
        }
      },
      "relation": "delegate_permission/common.handle_all_urls",
      "target": {
        "androidApp": {
          "packageName": "org.doncc.l02learnapplinks",
          "certificate": {
            "sha256Fingerprint": "01:CD:30:56:A8:5E:EC:18:64:3D:8B:F2:AF:3F:B6:02:85:33:79:13:97:23:E3:FF:99:59:14:E5:03:FC:57:78"
          }
        }
      }
    }
  ],
  "maxAge": "599.298570247s",
  "debugString": "********************* ERRORS *********************\nNone!\n********************* INFO MESSAGES *********************\n* Info: The following statements were considered when processing the request:\n\n---\nSource: Web asset with site https://doncc.github.io. (which is equivalent to 'https://doncc.github.io')\nRelation: delegate_permission/common.handle_all_urls\nTarget: Android app asset with package name org.doncc.l02learnapplinks and certificate fingerprint 01:CD:30:56:A8:5E:EC:18:64:3D:8B:F2:AF:3F:B6:02:85:33:79:13:97:23:E3:FF:99:59:14:E5:03:FC:57:78\nWhere this statement came from:\n  Origin of the statement: Web asset with site https://doncc.github.io. (which is equivalent to 'https://doncc.github.io')\n  Include directives followed (in order):\n    \u003cNone\u003e\nMatches source query: Yes\nMatches relation query: Yes\nMatches target query: Yes\n\n--- End of statement list. ---\n\n\n"
}
```

- 借助Google Digital Asset Links工具验证

还有一种验证方法，借助[Google Digital Asset Links](https://developers.google.com/digital-asset-links/tools/generator)这个工具，就能进行验证。

文中所填写的内容，正是你在配置前面`assetlinks.json`数字资产链接文件的信息。填写进去，进行验证。如果验证通过，说明你的网站已通过安全验证，可以愉快的使用App Links了。

- 借助App Links Assistant的测试工具验证

事实上还是有其他方法进行验证，借助App Links Assistant的测试工具进行验证。这部分我们会在后部分讲到。

### 3.2.2通过Shell命令验证

- adb命令构建URL Intent：

验证的过程和验证Deep Links类似。我们可以用构建URL Intent的方式来进行模拟链接的验证：

```
adb shell am start -a android.intent.action.VIEW \
    -c android.intent.category.BROWSABLE \
    -d "http://domain.name:optional_port"
```

注意，这里`-d`的内容需要替换成我们自己的网站地址。

如果成功，那么立即就会在模拟器上看到执行后的效果，直接的跳转到所在的Activity，而无需经过用户点击弹出框的确认过程。

![img](https://segmentfault.com/img/remote/1460000011169959)

这里我贴出关于App Links在开发时的视频：[如何使用 App Links 处理URL链接](http://v.youku.com/v_show/id_XMzAyMTM0NTI5Ng==.html)

# 4.通过App Links Assistant来协助配置

不难发现，实则Deep Links和App Links都有异曲同工之妙。我们从上面的内容来看，确实配置过程有点繁琐。幸好Google大大们已经为我们着想了这一点，在Android Studio 2.3版本及以上版本，内置了一个**App Links Assistant**的链接生成助手。根据这个助手的指引，我们就能很快的配置出属于我们自己程序的App Links，甚至是Deep Links。

那么这个助手该如何找到呢？在你的Tools > App Links Assistant，就可以找到了。

![img](https://segmentfault.com/img/remote/1460000011169960)

![img](https://segmentfault.com/img/remote/1460000011169961)

这里具体有一些如何使用App Links Assistant进行配置的文章描述，我们就不再赘述了，官方的文档已经很详尽的说明了这一点。如需参考请异步地址：[Add Android App Links](https://developer.android.com/studio/write/app-link-indexing.html)

# 5.Deep Links和App Links的对比

这里我们引自官方的对比表格更为直接。（来自：[the-difference](https://developer.android.com/training/app-links/verify-site-associations.html#the-difference)）

|                   | Deep Links                               | App Links                                |
| ----------------- | ---------------------------------------- | ---------------------------------------- |
| Intent URL scheme | `http`, `https`, or a custom scheme      | Requires `http` or `https`               |
| Intent action     | Any action                               | Requires `android.intent.action.VIEW`    |
| Intent category   | Any category                             | Requires `android.intent.category.BROWSABLE` and `android.intent.category.DEFAULT` |
| Link verification | None                                     | Requires a [Digital Asset Links](https://developers.google.com/digital-asset-links/v1/getting-started) file served on you website with HTTPS |
| User experience   | May show a disambiguation dialog for the user to select which app to open the link | No dialog; your app opens to handle your website links |
| Compatiblity      | All Android versions                     | Android 6.0 and higher                   |

我们不难看出，App Links的出现是弥补了Deep Links的一些不足。当然使用起来也应该酌情而定，毕竟Deep Links的超高自由度，也是后者望尘莫及的。说了这么多，再结合Instant App，我们就可以总结出以下几点：

- 在Instant App内使用App Links，具有更为安全的通信协议，也为协议的验证做好铺垫；
- 因为使用即时应用的前提，是点击一个链接直接跳转到界面，所以在Intent处理上，严格要求处理动作是`android.intent.action.VIEW`；
- 同上面描述的，Instant App也需要可以在网络上搜索到，并打开程序，比如Google搜索结果页面上打开。固然需要Intent的种类是`android.intent.category.BROWSABLE` 和 `android.intent.category.DEFAULT`；
- App Links是可以对URL链接进行安全验证的，而Deep Links是不具有验证功能的。其目的是为了能让Android程序足够信任其URL链接地址，减少让用户选择“xxx方式打开”这种动作歧义的对话框出现，在用户体验上能更让用户觉得是“无缝”的感觉；
- 对于链接点击处理功能的兼容，Deep Links能兼容所有Android版本，而App Links只兼容Android 6.0及更高的版本。

但是话有说回来，不难发现，在App Links与Deep Links的处理场景是很受局限的。大多数的程序是不经过系统处理的，而是直接内置方式处理。比如某聊天工具🙄。。。这就给其推广造成很大的阻碍，而能实现处理的地方有短信、内置浏览器内等一些原生App内。真是心塞。

好，这回我们就能彻底的了解了这两种URL链接处理方式了。

# 6.总结

对于Instant App程序开发，是依赖于App Links处理URL的方式，并且App Links又是一种特殊的Deep Links，也就是必须先知道Deep Links的作用，才能知道App Links的作用。

Deep Links的出现，是为了能让用户在不同应用见体验到App级的用户体验，而App Links的出现，是为了让这种体验更加“无缝衔接”；

对于App Links，需要一个数字资产链接文件的验证，这个文件是必须放在`网站根目录/.well-known/assetlinks.json`这个路径下，才能生效；

对于数字资产链接文件，其内部注意你的程序的签名，如果程序是released版本，那么你的签名指纹也需要换成对应的指纹；

对于App Links，需要在过滤器内添加`android:autoVerify="true"`属性，保证系统能识别到。

讲到这里，我们就可以初步了解了有关于制作Instant App的周边技术了。接下来我还会有更多的文章来持续讲解Instant App程序开发。