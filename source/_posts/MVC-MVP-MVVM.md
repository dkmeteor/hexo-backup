title: 'MVC & MVP & MVVM'
date: 2015-12-21 08:15:11
tags:
---
#MVC & MVP & MVVP

先放2个引用，概念上的讨论我觉得可以参考这2个。

[http://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html](http://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html)

[http://weibo.com/p/1001603808855434892996](http://weibo.com/p/1001603808855434892996)  


我主要想讨论下 这一演化过程,和架构选择的必要性。

###MVC

首先，传统的MVC项目中，layout.xml承担View的角色，Activity承担Controller的角色。

这本身没有什么问题，如果你的Activity代码不足千行（当然不包含网络请求及解析），多拆分几个方法，写好注释，也可以一用。
你也可以通过把其中部分逻辑 模块化抽出的方式 简化Activity代码。很多小型项目或者功能简单的Activity，使用MVC模型就可以了，盲目的使用MVP或MVVM并不是好事。


###MVP

然而随着项目的不断发展和复杂化，Activity承担的内容越来越多，请求数据，响应数据，数据处理，生成UI，修改view的状态，特效实现。

当Activity过于膨胀，我们就需要拆分它。

按职责，拆分为 `业务逻辑` 和 `UI逻辑`，就是Android中当前推荐的MVP实现.

- 业务逻辑

    包含请求数据，响应操作，对数据的处理等等业务相关的代码。
    
- UI逻辑

    包含界面的调整，UI状态的变化，动效等。

业务逻辑通过抽出 Presenter 层实现。
UI逻辑通过构造 ViewInterface并由Activity实现。

由于View和Presenter仅通过  ViewInterface关联，单独修改View或Presenter不会互相影响。

这是个好事，一般工程上小的UI调整总是多如牛毛，避免其影响到业务逻辑层可以减少很多bug的产生。

反过来也是一样。

当然，坏处也有，当你的Presenter需要添加一个新的功能/行为时，你需要在ViewInterface上添加新的接口，并在Activity上实现。
更多的灵活性总是意味着 更复杂的项目层次。

如果你遇到2种问题，那么你应当使用MVP


1. 你需要自动化测试。

MVP的引入会让测试变得非常方便。  
写一个简单的包含mock数据的Presenter就可以测试所有UI逻辑。
可以非常方便的在没有后端的情况下完成所有UI层逻辑及测试

同时 Presenter的测试也变得容易，因为Presenter现在与View是完全分离的，这意味着，测试业务逻辑时，你甚至不需要Android 环境，而且可以简单的mock ui行为。
这可比用脚本在模拟器上跑测试要方便多了。

2. 更高的复用率

Activity或Presenter都可以复用。如果你的项目需要快速迭代，那往往意味着，大量的UI变更，大量的逻辑调整。
分离Activity和Presenter可以减少很多 因为互相影响产生的bug，而且页面和逻辑变得方便调整和复用。

###MVVM

Android上谈MVVM，目前大多是指data binding.
这个东西刚出来的时候还比较火，但是实际用起来坑还比较多，目前没有看到哪个大型项目有实践这个东西。

具体使用可以参考这个文档  [MasteringAndroidDataBinding](https://github.com/LyndonChin/MasteringAndroidDataBinding) 。
看起来不错，用起来都是坑。
