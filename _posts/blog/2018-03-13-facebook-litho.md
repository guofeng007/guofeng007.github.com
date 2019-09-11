---
layout: post
title: Facebook UI 框架 Litho初探
categories: Blog
description:  Facebook UI 框架 Litho初探
keywords:     Facebook、 UI 框架、 Litho初探

---

# Facebook UI 框架 Litho初探

参见 [ Litho 官网](https://fblitho.com/docs/intro)
Demo 参见 [LithoDemo](https://github.com/guofeng007/FacebookLithoDemo)
# Litho是什么
Ligho 是 Facebook 出品的专门为构建Android UI界面的声明式框架，仅需要通过注解以及简单的 API 来编写高度优化的 Android UI 界面。其初衷是为了简化基于 RecycleView 的滑动式界面。

只需要定义一个简单的函数，接受输入属性，返回一个符合要求的界面。

```java
  @Prop String name) {

    return Text.create(c)
        .text("Hello, " + name)
        .textSizeRes(R.dimen.my_text_size)
        .textColor(Color.BLACK)
        .paddingDip(ALL, 10)
        .build();
  }
}
```

框架会自动在后台线程渲染（注意是非 UI 线程哟），自动增量渲染。
# 环境配置

## gradle 配置



```java
dependencies {
  // ...
  // Litho
  implementation 'com.facebook.litho:litho-core:0.12.0'
  implementation 'com.facebook.litho:litho-widget:0.12.0'
  compileOnly 'com.facebook.litho:litho-annotations:0.12.0'

  annotationProcessor 'com.facebook.litho:litho-processor:0.12.0'

  // SoLoader
  implementation 'com.facebook.soloader:soloader:0.2.0'

  // For integration with Fresco
  implementation 'com.facebook.litho:litho-fresco:0.12.0'

  // For testing
  testImplementation 'com.facebook.litho:litho-testing:0.12.0'
}

如果需要 section 库（列表库），额外添加
dependencies {

  // Sections
  implementation 'com.facebook.litho:litho-sections-core:0.12.0'
  implementation 'com.facebook.litho:litho-sections-widget:0.12.0'
  compileOnly 'com.facebook.litho:litho-sections-annotations:0.12.0'

  annotationProcessor 'com.facebook.litho:litho-sections-processor:0.12.0'
}

```

## 初始化



```java
[MyApplication.java]
public class MyApplication extends Application {

  @Override
  public void onCreate() {
    super.onCreate();

    SoLoader.init(this, false);
  }
}

```

### 使用


```java
[MyActivity.java]
import com.facebook.litho.ComponentContext;
import com.facebook.litho.LithoView;

public class MyActivity extends Activity {

  @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    final ComponentContext c = new ComponentContext(this);

    final LithoView lithoView = LithoView.create(
    	this /* context */,
    	Text.create(c)
            .text("Hello, World!")
            .textSizeDip(50)
            .build());

    setContentView(lithoView);
  }
}

```




