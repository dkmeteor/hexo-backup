title: 是否有必要在Android项目中使用IOC框架
id: 211
categories:
  - Blog
date: 2015-01-28 16:21:10
tags:
---

本来只是想找个注解库用一下
结果顺便看了看相关的库

首先是
butterknife   [https://github.com/JakeWharton/butterknife](https://github.com/JakeWharton/butterknife)

在任何项目中使用butterknife都是正确且没有问题的. 非常轻量级的库.
可以缩减一部分代码.
不过对一般应用而言..90%的时间是处理业务逻辑....View binding这种工作量其实很小...而且在IDE有代码提示的情况下其实也不会多敲几下键盘,  findviewById(R.id.text)
有智能提示的比如 AS 打 fv然后回车就好了.....对比 InjectView(R.id.text)

onClick绑定一般会用公共的OnClickListener写在一起然后用Switch Case去区分每个按钮的事件..
阅读性上还可以...也不算复杂....用butterknife可以直接做方法绑定...略微方便了一点点.

总统而言  便捷性上和可阅读性上有一定提升,虽然推荐使用,但是影响没有大到一定要用的地步.


其次是
Dagger [https://github.com/square/dagger](https://github.com/square/dagger)

Dagger其实和butterknife一样也是<span style="color: #999999;"> </span><span style="color: #444444;">[JakeWharton](https://github.com/JakeWharton) 写的, 算是一家的产品吧.</span>
<span style="color: #444444; font-family: Helvetica, arial, freesans, clean, sans-serif, 'Segoe UI Emoji', 'Segoe UI Symbol'; font-size: small;">View binding注解部分和</span>butterknife几乎一致.可以认为没有区别.
Dagger额外提供了 IOC框架 功能

最后是
RoboGuice [https://github.com/roboguice/roboguice](https://github.com/roboguice/roboguice)  和
google guice [https://github.com/google/guice](https://github.com/google/guice)

据说 Dagger2.0 也被移交给Google管理了

RoboGuice在View Binding上和以上2个库没有区别
也额外提供了 IOC框架功能

它和 Dagger 的区别是
RoboGuice 是 运行时 通过反射实现的IOC
Dagger 是通过 <span style="color: #333333;">APT(Annotation Process Tool)  在运行时 生成辅助类 实现的 IOC</span>
<span style="color: #333333;"> </span>
<span style="color: #333333;">原理上来讲 Dagger性能当然比 </span>RoboGuice 要好...但是 如果不是大量通过IOC生成对象...
仅仅普通使用应当差别可以忽略不计..未测试...


* * *

以上都是背景...下面进入正题...


不考虑RoboGuice和Dagger对代码的精简作用,是否有必要在Android上使用IOC框架?
(因为只考虑代码精简的话,butterknife就可以了....IOC更多的是设计上的考虑)

首先把这个文章看了2遍
[https://github.com/android-cn/android-open-project-analysis/tree/master/dagger](https://github.com/android-cn/android-open-project-analysis/tree/master/dagger)

又翻了一下Spring中使用IOC的例子...

IOC的核心是解耦...
解耦的目的是 .... 修改耦合对象时不影响另外一个对象...降低模块之间的关联

从Spring来看.....在Spring中IOC更多的是依靠 xml的配置...

而以上Android上的IOC框架均不使用xml配置系统....而且也没有必要....
毕竟不是web要去考虑server/system/db/web容器的差异...

常见的例子..


<pre class="lang:default decode:true ">public class MainActivity extends Activity {
    @Inject Boss boss;
    ...
}</pre>
&nbsp;



就看 [扔物线](https://github.com/rengwuxian) 写的这个例子好了..

对于这个设计而言...谁被解耦了?

MainActivity和Boss被解耦了.....更确切的讲....MainActivity和Boss的构造方法解耦了...
如果你下面需要使用Boss对象,仍然需要调用Boss中的方法...那么Boss和MainActivity依然存在依赖关系..
好处是,我修改Boss的构造方法不会影响MainActivity了...我可以修改Provider去自己获取参数..
仅此而已?

Android中 Activity 一般不会参与复用..你不会去再写一个Activity继承MainActivity,也不会去生成多个MainActivity....即使同时存在多个MainActivity实例也是归系统管理..和你没有关系.... 所有通常我认为只有Boss是会复用的会变化的会被继承的...MainActivity的变化只和它自身有关系,不会影响系统中其它部分..

仔细思考,即使是在IOC框架中,耦合性也并没有消失,只是将耦合的部分从 Class之间转移到了 Class和IOC容器之间
回到上面这个部分...

你觉得 Activity 在 Android app中 处于 一个什么角色??
Activity不能充当容器角色么?

UI / 业务逻辑 / DB / Conection / Cache
所有组件都只和Activity产生依赖关系,互相之间互不影响....

在Activity外再嵌套一层IOC容器来解耦偶到底有无必要?
Activity为什么还需要解耦?

画一下2个模型图

![IOC](/images/IOC.png)


有区别?
我构造一个场景
Activity需要做请求网络的操作,操作完成后写DB和Cache,IOC给你注入一个 HttpMananger / DBManager / CacheManager 又如何,你不要调用它们的方法吗? 耦合性并没有降低。






* * *



总之我认为在这样的典型场景下使用IOC并无必要，因为目前为止我看到的Android上使用IOC的例子均是类似这样注入一个单例/工程 管理类。



如果其中某一个模块复杂度非常高，可以被分离成多个子模块，那么在子模块间使用IOC进行解耦是很不错的方案。遗憾的是，目前没遇到这样的应用。

我猜想在某一个专业领域非常深入的应用可能会更需要这样的功能。



比如图像处理应用。使用颜色渲染器，光线渲染，分层/块/区处理，自动裁切，构造Gif/短视频 等等等，其在单一功能上流程长复杂度高，那么使用IOC分离这些模块非常棒。

遗憾的是目前我常做的这些 社交/电商 应用 在层级上非常非常浅，更多的UI/交互/业务逻辑 之类的玩意，这些东西属于很恶心很麻烦，但是不复杂也不难的东西。



恩。。。发到SegmenFault看看别人是什么看法。