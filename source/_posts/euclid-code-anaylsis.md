title: Euclid 源码分析
id: 267
categories:
  - android
  - Blog
date: 2015-03-19 17:30:38
tags:
---

Euclid 源码分析

Euclid
https://github.com/Yalantis/Euclid

效果比较惊艳，clone下来分析一下源码~

99%的UI动效都是Trick.....
看完源码蛋都碎了....

* * *

看源码之前比较疑惑如何做到Activity切换时动效这么流畅
还以为是 android.Transition Framework 的一个我不知道的用法..

表现形式上很像新的转场动画特效（Shared elements between activities）

> android.Transition Framework can be used for three main things:
>   - Animate View elements in transitions between activites (or fragments)
>   - Animate shared elements (hero views) in transitions between activities (or fragments)
>   - Animate View elements from one activity scene to another.

Github上有个低版本的兼容库 https://github.com/tianzhijiexian/ActivityOptionsICS
使用在Intent上传递Bitmap做到View的传递，不过只有视觉上的效果，因为传过去的并不是View，只是用指定View生成的Bitmap

我自己的ScreenTransitionTool原理也和这个一样.......不过当时没写完...丢下了就有点捡不起来了..

这个时候就觉得 IOS 的 ViewController系统优势的多,Controller切换时可以同时访问前后2个Controller

回归正题

Euclid可以做到这个效果的原因真的很傻逼，因为它的List界面和Detail界面在同一个Activity中....

所以.....剩下就是普通的 ObjectAnimator ......... sad ....