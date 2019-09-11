---
layout: post
title: Android Clean Arch
categories: Blog
description: Android Clean Architecture
keywords: Google, Andoroid , App, Clean Component
---



# Google Android Architecture Component 

 

**作者：**[crazysunj](http://crazysunj.com/)

​        本来准备总结写一篇的，看该作者写的简洁有力，就不造轮子浪费时间了，内容如下：



# 带你领略Clean架构的魅力

## 前言

当项目需求不断扩张的时候，当开发团队人员不断增加，当新技术不断涌现，当软件质量不断提高，我还是不能和你分手，不能和你分手。我对唱出声的同学不发表任何意见。如果你真的碰到上述问题而没有演进你的架构，可能你碰到的问题都是属于灵异事件。那这里的核心点是架构，那它又是个什么玩意？它能带来什么好处？

## 架构

到底什么是架构？我现在的水平还不能告诉你，没资格。我能告诉你自己在学习过程中的领悟，我想架构就是一种约定，怎样的约定？为了更好的研发效率、扩展性和稳定性，更精准的风险防控能力。这样说可能有点抽象。

![img](http://crazysunj.com/img/lizi.jpeg)

作为一名资深宅男，床到门就是世界上最遥远的距离，上一次厕所好像经历了九九八十一难，但我们从不寂寞，你说是吧，Mac。额，好像扯远了，我用完东西可能一直放在我喜欢的地方，只是在其他人眼中显得有点乱，在我眼里是很有规律的。因为我一直遵循着这个规定，每本书都有我给它定义的位置。但是我老妈不知道啊，虽然整理的很干净，但是书放错了位置，老妈可能是无意识的，就算下次我问她，她也记不清了。为了找到这本书，我也只能遍历一遍了。如果我们双方都遵循我们俩一起定义的一个规定，一个约定，达成一种共识。那么就很简单了，书永远在那，就算是添新书还是移除，双方都有章可循。

由此可见，架构的好处就是除了引进新的技术，自己或者说产品本身都有更好的体验外，还由于做了统一规范，让结构更加清晰，健壮，团队协作更加方便。

![img](http://crazysunj.com/img/meizizi.jpeg)

以上并不是今天的主题，架构的理解也仅限于我的理解，相对于Android，我们知道的架构更多的可能叫MVC、MVP和MVVM等。关于这个。。。对方不想和你说话，并向你扔了一个[链接](http://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html)。一篇好的文章是由文章本身内容和优秀的评论组成的，所以，大家慢慢欣赏，算了，买一送一，喜欢研究架构的同学戳[这里](https://github.com/googlesamples/android-architecture)，谷歌爸爸的。这里，我直接引出今天的主题Clean架构。

## Clean架

大多数情况下，碰到问题，我们要从3个方向去思考，what、how、why。引用某大佬的一句话：未经思考的人生是不值得去过的人生。所以，列位看官不要仅仅满足于快餐文化，经常多动脑子，看问题千万不要流于表面！额，又扯远了。。。我先说说what。

### What

概念一直都是枯燥乏味的，如果不喜欢的朋友，可以直接跳到How或者本章节的总结部分，但是概念也是最基础的，最基础是不是最重要我不发表意见。

秉承我们No picture,say a J8的优雅传统，先上个毫无意义的结构图。

![img](http://crazysunj.com/img/cleanArchitecture.jpg)

这张图可能对于你们小白看来觉得很不可思议，但对于我们专业来说，也是一脸懵逼（手动滑稽），我简单的解释下：

- Enterprise Business Rules：业务对象
- Application Business Rules：用于处理我们的业务对象，业务逻辑所在，也称为Interactor
- Interface Adapters： 接口转换，拿到我们需要的数据，表现层（Presenters）和控制层（Controllers）就在这一层
- Frameworks and Drivers: 这里是所有具体的实现了：比如：UI，工具类，基础框架等等。

关于上面的图，米娜可以戳[这里](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html)。

我解释后估计你还是一头雾水，我们再来看一个图：

![img](http://crazysunj.com/img/clean_model_all.jpg)

好像相对于上面那张图更好理解，知道为什么吗？因为字少了好多。哈哈。接下来的内容以及我的开源项目中都是以此为基础来写的。分别来解释下。

#### 表现层 (PRESENTATION LAYER)

我们这里的表现层以MVP为基础，个人觉得Clean本身也是MVP的基础上更加抽象，更加独立。熟悉的MVP的同学非常清楚这一层是干嘛用的。老规矩，先上张图。

![img](http://crazysunj.com/img/clean_model_p.jpg)

是不是很眼熟？P层使得V层（Fragment和Activity）内部除UI逻辑再无其它逻辑。而我的开源项目中的Presenter由多个Interactor组成。底下会介绍。

#### 领域层 (DOMAIN LAYER)

![img](http://crazysunj.com/img/clean_model_domain.jpg)

图上很明显了，这里主要是interactor的实现类和业务对象。讲道理这里应该只属于java模块，但是有时候我们的业务对象，可能要实现第三方库中的实体类接口，不得不改为Android模块，暂时没想到很好的办法，有知道的大佬可以指教一下。

#### 数据层 (DATA LAYER)

![img](http://crazysunj.com/img/clean_model_data.jpg)

这是一种Repository模式，具体的可以看[这里](https://msdn.microsoft.com/en-us/library/ff649690.aspx)。以我现在的见解，只能说只要项目复杂而需要分层，那么就应该用这个模式，它让clean架构的clean更加亮眼。

这个本来就是概念，我相信大家也不愿意看，所以就简单介绍。如果想详细了解，可以戳[这里](https://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/)。

#### 总结

现在谈谈自己的看法，后者是相对前者较为具体的一种符合Android的结构。在这插一个clean架构的依赖性规则：内层不能依赖外层。三者也都分别解释了是干什么用的，那么为什么有分为这三者，它们又有什么联系？我是个俗人，那就应该用俗话来讲，从数据层利用Repository模式让领域层感觉不到数据访问层的存在，即原始数据是独立的，业务规则不绑定具体哪一种数据，通俗点讲就是你要什么数据？我给你取，但你不需要知道我从哪里取的；因此领域层对数据层怎么实现的是一无所知，而领域层主要工作就是你给了我数据，那我就要用，怎么用？都是我来决定；用完之后再回调给表现层渲染UI。因此大多数的业务逻辑都在领域层，可以说是一个APP的核心。我认为这里透露着一个很重要的设计理念就是数据驱动UI，我都想给自己点个赞，哈哈。其实，到这里，你心里已经有点13数的话，可以跳到Why，因为怎么用已经是具体的东西，而架构本身就是一种共识，是抽象的，从Java角度讲你可以多个类去实现这个接口。下面的使用只是我对Clean架构理解的一点代码体现。

### How

下面的例子是从我开源库[CrazyDaily](https://github.com/crazysunj/CrazyDaily)中选取的，以知乎日报为例。

#### 数据层 (DATA LAYER)

数据层就是从我们的仓库（Repository）中取数据，可以从云端、磁盘或者内存中取。

```
public interface ZhihuService {
    String HOST = "http://news-at.zhihu.com/api/4/";

    @GET("news/latest")
    Flowable<ZhihuNewsEntity> getZhihuNewsList();
    
    @GET("news/{id}")
    Flowable<ZhihuNewsDetailEntity> getZhihuNewsDetail(@Path("id") long id);
}

public class ZhihuDataRepository implements ZhihuRepository {
    ...
    @Inject
    public ZhihuDataRepository(HttpHelper httpHelper) {
        mZhihuService = httpHelper.getZhihuService();
    }
    @Override
    public Flowable<ZhihuNewsEntity> getZhihuNewsList() {
        return mZhihuService.getZhihuNewsList()
        ...
    }
    ...
}
```

这里比较尴尬的是只提供了云端的数据，采用的是retrofit+okhttp的框架获取。比较正确的方式应该是给ZhihuDataRepository提供一个Factory而不是HttpHelper，Factory根据不同的条件获取相应的数据。比如像这样：

```
@Inject
 UserDataRepository(UserDataStoreFactory dataStoreFactory,
     UserEntityDataMapper userEntityDataMapper) {
   this.userDataStoreFactory = dataStoreFactory;
   this.userEntityDataMapper = userEntityDataMapper;
 }
```

UserDataStoreFactory是从不同地方获取数据的一个工厂类，UserEntityDataMapper是我们的数据包装类，不知道还记得上面的Interface Adapters吗？细心的朋友可以关注到ZhihuDataRepository实现了ZhihuRepository，但是ZhihuRepository并非数据层的东西，而是领域层的东西，很显然，以接口进行关联，但内容独立，没错，这就是传说中的依赖倒置原则。

![img](http://crazysunj.com/img/666.jpeg)

#### 领域层 (DOMAIN LAYER)

```
public interface ZhihuRepository {
    
    Flowable<ZhihuNewsEntity> getZhihuNewsList();

    Flowable<ZhihuNewsDetailEntity> getZhihuNewsDetail(long id);
}

public abstract class UseCase<T, Params> {

    ...

    public UseCase() {
    	...
    }

    protected abstract Flowable<T> buildUseCaseObservable(Params params);

    public void execute(Params params, DisposableSubscriber<T> subscriber) {
        ...
    }
	...
}

public class ZhihuNewsListUseCase extends UseCase<ZhihuNewsEntity, Void> {

    private final ZhihuRepository mZhihuRepository;

    @Inject
    public ZhihuNewsListUseCase(ZhihuRepository zhihuRepository) {
        mZhihuRepository = zhihuRepository;
    }

    @Override
    protected Flowable<ZhihuNewsEntity> buildUseCaseObservable(Void aVoid) {
        return mZhihuRepository.getZhihuNewsList()
        ...
    }
}
```

真的很完美，跟数据层一毛线关系都没有，利用接口（ZhihuRepository）来控制数据层（ZhihuDataRepository）。真的感觉架构越来越有意思了。我可以在这里处理我们大部分的业务逻辑。

#### 表现层 (PRESENTATION LAYER)

```
@ActivityScope
public class HomePresenter extends BasePresenter<HomeContract.View> implements HomeContract.Presenter {

    private ZhihuNewsListUseCase mZhihuUseCase;
    ...

    @Inject //多个UseCase
    public HomePresenter(ZhihuNewsListUseCase zhihuUseCase ...) {
        mZhihuUseCase = zhihuUseCase;
        ...
    }

    @Override
    public void getZhihuNewsList() {
        mZhihuUseCase.execute(new BaseSubscriber<ZhihuNewsEntity>() {
            @Override
            public void onNext(ZhihuNewsEntity zhihuNewsEntity) {
                mView.showZhihu(zhihuNewsEntity);
            }
        });
    }
}

public interface HomeContract {

    interface View extends IView {

        void showZhihu(ZhihuNewsEntity zhihuNewsEntity);
		...
    }

    interface Presenter extends IPresenter<View> {

        void getZhihuNewsList();
		...
}

public class HomeActivity extends BaseActivity<HomePresenter> implements HomeContract.View {

    @Override
    protected void initData() {
        mPresenter.getZhihuNewsList();
        ...
    }

    @Override
    public void showZhihu(ZhihuNewsEntity zhihuNewsEntity) {
    	 ...
    }
	...
}
```

说实话真的不想贴代码，太麻烦了，但不贴，不太好理解，而与我们相濡以沫的也就是代码了，眼睛莫名一酸。表现层更多的是MVP的概念，所以。。

简单的理下逻辑，Activity发起获取数据信号（View）->调用Presenter接口（Presenter）->调用UseCase接口（Domain）->调用Repository接口（Data）->拿到原始数据->回调Repository（Data）->回调UseCase（Domain）->回调Presenter（Presenter）->回调Activity（View）

如果这都看到不懂，买块豆腐撞死算了。

![img](http://crazysunj.com/img/yukuaiwanshua.jpg)

### Why

其实How并不是很重要，但却占了很大篇幅。

![img](http://crazysunj.com/img/aha.jpeg)

你认为Why是最重要的吗？这个问题留在心中，先来看看Why。

- Easy to maintain
- Easy to test
- Very cohesive
- Decoupled

![img](http://crazysunj.com/img/heiren_wtf.jpg)

看见高内聚，低耦合我就头疼。说说其它吧，易于维护，这样的结构已经能说明了，易于测试也好理解，各个模块独立，来看看这：

- 展示层 (Presentation Layer) : 使用android instrumentation和espresso进行集成和功能测试
- 领域层 (Domain Layer) :使用JUnit和Mockito进行单元测试
- 数据层 (Data Layer) :使用Robolectric（因为依赖于Android SDK中的类）进行集成测试和单元测试

说来惭愧，嫌麻烦，我在开源项目里并没有写测试，相信我以后会加的，我也不是那种不负责任的人，献上大佬的一个关于clean的开源项目[Android-CleanArchitecture](https://github.com/android10/Android-CleanArchitecture)。

## 题外骚话

关于clean的差不多到这里结束了，如果有什么问题，可以联系我，一起讨论。先喝一碗鸡汤为敬，学会平静的对待生活中的不完美之处，适应自己的情绪，了解如何让它们自然宣泄出去。说来也好笑，我的一个库[MTRVA](https://github.com/crazysunj/MultiTypeRecyclerViewAdapter)也受到了我学习架构时的影响，它是RecyclerViewAdapter的相当于Presenter层的一个东西，我们总是免不了在Adapter中对资源或者说数据的处理，这就跟Activity一锅粥一样，它让你只关注UI逻辑。有人说，我的界面简单，啥都不用算，给我list，setAdapter，搞定。额。。。那确实不用。同理，简单的项目也并不需要什么复杂的架构，相信从头看到尾的同学，很明显的看出clean架构的缺点，就是繁琐。架构本然就是在需求中不断演进。祝你能够搭建属于你们的架构。最后，感谢一直支持我的人！

## 传送门

Github：<https://github.com/crazysunj/>

博客：<http://crazysunj.com/>