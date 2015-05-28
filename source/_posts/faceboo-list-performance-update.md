title: 关于 Facebook新闻页优化 的分析
id: 252
categories:
  - android
  - Blog
date: 2015-03-17 14:01:52
tags:
---

关于 Facebook新闻页优化 的分析


#私货

简单一点概括，就是Facebook在ListView页面中，将原来的View-Model-Binder再次拆解成粒度更细的模型。


在原本的`View-Model-Binder`模型中，依靠将部分的View设置成`visible` 或 `gone` 来控制功能块的显示 （转帖/图片/etc）


在优化过的模型中，原本的View被拆解成更细粒度的小View，然后只加载需要的部分


原文没有提及，但应当是利用

    public int getItemViewType(int position) {
    return 0;
    }
    
    public int getViewTypeCount() {
    return 1;
    }





实现的。



翻了一下ListView源代码

不同type的View被分类cache在RecycleBin中



    public void setViewTypeCount(int viewTypeCount) {
        if (viewTypeCount &lt; 1) {
            throw new IllegalArgumentException("Can't have a viewTypeCount &lt; 1");
        }
        //noinspection unchecked
        ArrayList&lt;View&gt;[] scrapViews = new ArrayList[viewTypeCount];
        for (int i = 0; i &lt; viewTypeCount; i++) {
            scrapViews[i] = new ArrayList&lt;View&gt;();
        }
        mViewTypeCount = viewTypeCount;
        mCurrentScrap = scrapViews[0];
        mScrapViews = scrapViews;
    }





参考RecycleBin的setViewTypeCount部分，此处用ArrayList[]来cache所有type的view


不过由于被设置成gone的View本身就不参与meaure和layout,也不会参与draw,只会额外占据一些内存。

所以Facebook做了这些优化以后  性能只提升了10%


在View的性能优化中，我觉得10%太少了，而新的模型复杂度变高了很多。


原本由于ViewType的原因，需要将Model和View的实现细节暴露在 Adapter中，Facebook这个优化实现似乎为了保持其封装性

于是将View-Model-Binder又通过Definition又封装了一层。



综合考虑的话....我觉得意义不大。



优化掉了一些无用View,但是增加了额外的中间层，很难说这样的改动是好是坏。

IOS开发可能更习惯View-Binder写法

Android开发可能会更习惯直接使用ViewHolder去操作



而Facebook这个写法更复杂，增加了额外的Definition层，Binder也根据ViewType被拆解成多个。

---



考虑一下直接用最大粒度的View来做ViewType区分？

思考了一下....参考Faacebook或微博这个界面...应当是不行的...


同屏最多也就 2~4条 post

直接用Item做type区分的话....由于数据随机性的问题

cache会被经常性的加载和释放.....反而起不到作用...


使用细粒度的子View做type区分的话，在cache上有优势的多..

---

**结论：有用，必要性不足。**


**鉴于这个Definition模型的复杂度，非微博，facebook,朋友圈类型的复杂Item，并不需要使用此优化方法。**


**正常使用View-model-binder模型就好了**

---
中文版
http://blog.aaapei.com/article/2015/02/facebookxin-wen-ye-listviewyou-hua

&nbsp;

### 基础知识

android系统每隔16.7ms发出一个渲染信号，通知ui线程进行界面的渲染。为了达到流畅的体验，应用程序需要在这个时间内完成应用逻辑，使系统达到60fps。当一个Listview被添加到布局时，其关联的adapter的getView方法将会被回调。在16.7毫秒这样一个时间单元内，可见listitem单元的getView方法将被按照顺序执行。在大多数情况下，由于其他绘图行为的存在，例如measure和draw，getVIew实际分配到执行时间远低于16ms。一旦listview包含复杂控件时，在16毫秒内不能完成渲染，用户只能看到上一祯的结果，这时就发生了掉帧。

### Facebook新闻页介绍

Facebook的新闻页是一个复杂的listview控件，如何使它获得流畅的滚动体验一直困扰我们。 首先，新闻页的每一条新闻的可见区域非常大，包含一系列的文本以及照片；其次，新闻的展现类型也很多样，除了文本以及照片，新闻的附件还可包含链接、音频、视频等。除此之外，新闻还可以被点赞、被转载，导致一个新闻会被其他新闻包含在内。当新闻被大量用户转载时，甚至会出现一条新闻占据两个屏幕的情况。加上android用户的机型多为中低端设备，这使我们在16.7ms内完成新闻页的渲染变的非常困难。

![](https://fbcdn-dragon-a.akamaihd.net/hphotos-ak-xpa1/t39.2365-6/10935993_1534797460105141_1373600061_n.png)

### 新闻页最初架构

在2012年，我们将新闻页从web-view转化成本地控件，在最初的那个版本中，基于View-Model-Binder设计模型，我们为新闻listitem创建了一个自定义StoryView类，这个类有一个bindModel方法，该方法用于和数据进行绑定。代码是这样的：

![](https://fbcdn-dragon-a.akamaihd.net/hphotos-ak-xap1/t39.2365-6/10935990_1412843022342801_1297058886_n.png)StoryView的包含的子控件都会有一个bindModel方法，例如HeadVIew通过该方法与其相关的数据进行绑定。

这种设计，代码非常直观清晰，但他的缺点也很明显：

*   listview复用机制不能有效的工作,Android's recycling mechanism does not work well in this case: Every item in the ListView was usually a StoryView, but once bound to a story, two StoryViews would be radically different and recycling one into the other wasn't effective.（这一段存疑，直接放原文）
*   逻辑嵌套：采用bindModel绑定控件和数据，业务逻辑与视图逻辑耦合，导致逻辑类层次非常深；
*   布局嵌套非常深：不但导致低效的视图渲染，例如新闻被不停的转载的极端场景下还会导致栈溢出；
*   bindModel方法逻辑过重：bindModel方法在当用户滚动列表时被ui线程回调，由于所有的数据解析都在这个方法内，导致该方法耗时

以上这些问题虽有他们单独的解决方法，例如我们可以自己设计一套回收机制解决storyView复用问题。但基于维护成本和开发时间考虑，我们决定进行一次重构。

### 重构方案

重构工作大约是一年之前开始的，为了解决前一个架构的问题，首先我们决定将一条新闻分隔成多个listview item。例如，新闻的headerview将是一个独立的listitem。这样，我们可以利用android回收机制，HeaderView新闻子控件将被不同的新闻复用。另外，切分成小view也使得内存占用更小，在之前的架构中，Storyview部分的可见会导致这个Storyview被加载到内存中，而现在，粒度更小，只有可见的子控件才会被加载。

![](https://fbcdn-dragon-a.akamaihd.net/hphotos-ak-xap1/t39.2365-6/10935983_984256741587871_980206636_n.png)

另一个大的修改是，我们将视图逻辑和数据逻辑分离，StoryView被分离成两个类： 只负责展现的视图类，以及一个Binder类。视图类仅包含set方法（例如HeaderView包含了setTitle，setSubTitle。setProfiePic等等）。Binder类包含了原来的bindMethod的逻辑，binder类包含三个方法：prepare，bind，unbind。 bind方法调用view的set方法设置数据，unbind清理视图数据，prepare方法在cpu空闲期间做一些预初始化工作，例如进行click事件绑定、数据格式化、创建spannable等等，它会在getView方法之前被调用 ![](https://fbcdn-dragon-a.akamaihd.net/hphotos-ak-xfp1/t39.2365-6/10956894_918624611495337_1619622974_n.png)

我们遇到的技术难点是Binder的设计，由于StoryView被拆分不同的子控件，一条新闻可能会包含多个不同的Binder。而在之前，我们只需要根据视图的树结构进行结构化赋值。因此，我们引进了_PartDefinition_类，PartDefinition负责维护一条新闻包含哪些子控件、包含Binder的类型以及为新闻创建Binder类，有两种类型的PartDefinition：单个PartDefinition以及PartDefinition集合。

![](https://fbcdn-dragon-a.akamaihd.net/hphotos-ak-xfa1/t39.2365-6/10935981_1536551233276267_1103658334_n.png)

一个新闻在重构之后的PartDefinition结构是这样的：

![](https://fbcdn-dragon-a.akamaihd.net/hphotos-ak-xaf1/t39.2365-6/10935975_856616717694467_1297407005_n.png)

### 结论

*   采取新的架构，内存错误减少了17%，总crash率减少了8%，彻底解决涨溢出问题
*   渲染时间减少了10%，大新闻场景不再掉帧
*   精简了原来的自定义回收机制，同时在重构过程中增加了单元测试

&nbsp;

英文原文：

https://code.facebook.com/posts/879498888759525/fast-rendering-news-feed-on-android/

&nbsp;

If you work on an Android app (or any touch screen based app actually), there's a very good chance you have at least one activity based on a ListView. In many cases, it may be the screen of your app where users interact with the app the most. In the case of Facebook for Android, this ListView is your News Feed. Keeping News Feed working well in Facebook's Android app presents a variety of engineering challenges starting with performance considerations for touchscreens.

## Why ListView Gets Complicated

ListView presents an interesting performance problem that wasn't as dominant on non-touch interfaces - the smoothness of the scrolling experience. Users expect that the content will seamlessly keep pace with their fingers as they scroll on a touchscreen. This need wasn't as important when using a keyboard and mouse.

To get your list to appear as if it scrolls smoothly, you need to render about 60 frames of it per second. Basic arithmetic translates this challenge into rendering one frame in under 16.7 milliseconds. This is pretty easily done for most frames with the exception of a notable subset. Whenever a new view goes into the viewport in your ListView, the ListView will call its adapter's`getView` method. That method, in turn, has to get a view, and bind all its content into it in less than 16.7ms. (In fact, since other actions have to happen, such as measuring and drawing, there's even less time). This is easily achievable for simple content, such as the views that contain text only. It becomes challenging when trying to render rich content which contains complicated views — a lot of text and images, for example. Android's simple tool to solve this is view recycling: The adapter's`getView` method will get an existing `convertView` to use if one was made before. This trick works very well, but only when your views are similar to each other.

## Rendering the Facebook News Feed

Facebook's News Feed, a popular screen in the Facebook for Android app, is an extreme example of a complicated ListView. This makes delivering a smooth scrolling experience especially hard.

First, each story you see in the feed is pretty tall, and contains several pieces of text and photos. This makes the challenge of binding all of the story's data without skipping frames significantly harder.

Secondly, there are multiple variations of stories in your feed. Aside from a simple story containing some text and a photo, you can see multiple variations of attachments: a preview for a link, an album, a video, etc. The story might be shared, in which case the story contains another story inside it. Hardest of all, we need to render aggregated stories which means one story can actually be composed of several stories that are related. A good example of this is when many of your friends wish you a happy birthday. These aggregated stories are the most challenging to render, as one of them can easily be twice as tall as the screen of the device.

As an extra challenge, the typical Android phone is not a high-end device. So the amount of computations you can fit in under 16.7ms is likely less than the amount on the majority of phones you develop on.

![](https://fbcdn-dragon-a.akamaihd.net/hphotos-ak-xpa1/t39.2365-6/10935993_1534797460105141_1373600061_n.png)

## How it used to work

Around 2012, we started working on converting News Feed from a web-view based solution to native rendering. We knew this was going to be a complicated task due to the sheer number of features news feed supported already at that time. Our design was based on the following concept: For every piece of a story in the feed, we created a custom view class. That custom view class would have a method called `bindModel` which would take the object this view was supposed to display and update the view to display it.

So, this meant to display a story we had a `StoryView` class, which among other things, had a`HeaderView` inside it to display the header of the story.

![](https://fbcdn-dragon-a.akamaihd.net/hphotos-ak-xap1/t39.2365-6/10935990_1412843022342801_1297058886_n.png)

This approach was simple and intuitive for finding the right piece of code or knowing where to add it, but had various drawbacks:

*   Android's recycling mechanism does not work well in this case: Every item in the ListView was usually a `StoryView`, but once bound to a story, two `StoryViews` would be radically different and recycling one into the other wasn't effective.
*   Complicated hierarchies: Using a `bindModel` method inside views meant coupling the business logic with the layout logic. Once something as complicated as a view for displaying a story existed, it was natural to subclass it when you needed a variation of that logic. This resulted in a complicated and hard to navigate hierarchy of classes, which is something to avoid.
*   Deep-View hierarchy: Logically grouping things meant putting them inside the same view which resulted in a very deep hierarchy of views. This means worse performance for Android when measuring views, and in extreme cases meant actual crashes due to stack overflows of the number of views. This cases tended to happen in something as an aggregated story, where the recursive “story in a story” rendering caused a very deep hierarchy.
*   Computationally-intensive method: There is no nice and consistent way to execute code before`bindModel` is called. Since `bindModel` is called while the user is scrolling, it's a pretty bad time to do heavy work. Since this is the place where you unpack a complicated story object into the various parts, that would not always be the case, which made `bindModel` a computationally intense method to run for some of the views.

All of these problems were solvable, but each required its own custom solution, which took time, and made maintenance more complicated. For example, to solve the recycling problem we had to implement our own custom recycler to work on top of the list view. With that in mind, we decided to rewrite our rendering code to avoid these pitfalls.

## Making News Feed Scrolling Smoother

About a year ago we felt it was the time to invest in a new architecture for our rendering code for News Feed. We knew we wanted to avoid the pitfalls of the previous design, and try to get rid of our more fragile code. For that purpose we considered a new idea: Splitting each story in the News Feed to several items in the ListView. Using this idea, the header part of a story will be its own item in the ListView. This nice trick results in various advantages right away. First, this makes Android's recycling effective again. If before two custom views for a story were two different to have one recycled into the other, now, recycling is happening on a sub story level.

So a story's header view is being recycled with a different header, and these have the same layout. The second big advantage is that splitting the views lets us store less views in memory, and bind a story over several frames. If before a huge aggregated story presented a difficult task of binding it without skipping a frame, now this story is split into many parts, and only one of those needs to be bound during frame. This effectively amortizes the binding time for a story over several frames, resulting in less skipped frames.

![](https://fbcdn-dragon-a.akamaihd.net/hphotos-ak-xap1/t39.2365-6/10935983_984256741587871_980206636_n.png)

The other idea we incorporated to the design is decoupling the binding logic from the custom Views themselves. For this purpose we basically split our previous custom view into two classes: A simpler “dumb” custom view, and a Binder class. The custom view is now limited to a basic set of setters (such as “setTitle”, “setSubtitle” and “setProfilePic” for the header) and is unaware of a story object. The binder class is the replacement for the `bindModel` method. It has three methods: `prepare`, `bind`, and `unbind`. The `bind` method takes a view and sets the right info from the story on it, the `unbind`reverses this and does cleanup which might be necessary (usually it's not needed) and lastly, the`prepare` method is meant to solve the issue of performing hard work during binding time. It is called before the first time a binder is bound, and intelligently scheduled when there's free CPU time on the UI thread. This makes it ideal for allocating click listeners, formatting strings, building spannables and so forth.

![](https://fbcdn-dragon-a.akamaihd.net/hphotos-ak-xfp1/t39.2365-6/10956894_918624611495337_1619622974_n.png)

To make our adapter work, we just need to generate the right list of binders. This is not a simple task; there are many types of binders, and each story has a different number of them according to the parts it should render. In the previous design, this was solved by the hierarchy of views, but now that there's no “one view” per story we needed to create it in a different way. We decided to do this by creating a `PartDefinition` class to define each possible part of a story. There are two types of part definitions: A single part definition which defines what views it needs, whether it is needed for a specific story, and how to create the correct binder for it. The second type is a group part definition that lets us logically group several parts into one.

![](https://fbcdn-dragon-a.akamaihd.net/hphotos-ak-xpf1/t39.2365-6/10935981_1536551233276267_1103658334_n.png)

Using these classes we rebuilt the hierarchy of parts of a story. Now to generate the list of binders for a story, we simply start from the root part and recursively expand it to the list of single parts, which we then use to generate binders.

![](https://fbcdn-dragon-a.akamaihd.net/hphotos-ak-xaf1/t39.2365-6/10935975_856616717694467_1297407005_n.png)

This rewrite effort has resulted in many benefits:

*   The number of out-of-memory errors experienced by people using Facebook has been reduced by 17% and the number of total errors was reduced by 8%. Some errors, such as stack overflows in the view hierarchy have disappeared.
*   The maximum time it takes to render a frame was reduced by 10%, after additional optimizations and removal of custom recycling code that were no longer needed. Additional simplifications are expected to improve by such amounts once more. Big jumps that resulted from loading a tall complicated story have disappeared.
*   We were able to simplify reusing our feed code with different layouts, which helped in the creation of the stand alone app for Facebook Groups on Android.
*   The new design has lent itself to improving our code quality. More teams can contribute to News Feed while being sandboxed from other teams working on parallel features. We also used this opportunity to make sure our code is testable and well covered. The new feed code has a line coverage with unit tests of 70% compared to 17% in the old code.

This has been a great undertaking to replace such a core piece of code seamlessly, but the hard work was worth it as it paid of both in improving the performance, and the reliability and maintainability of the code.



---

