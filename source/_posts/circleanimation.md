title: CircleAnimation
id: 271
categories:
  - android
date: 2015-03-19 17:55:51
tags:
---

![circle_animation](/images/circle_animation_demo.gif)



下午和同事讨论转场效果，晚上蛋疼试了一下。

做起来还比较简单，不过RevealAnimator整个系列API 还全是 @Hide，最后反射了几个View的私有API hack了一下..


加速器曲线没仔细调，用sin函数凑合了一下


目前限制还比较多，依赖Container，仅支持API21， 因为反射的关系，也不用谈什么兼容性了...我自己都不知道那几个@hide 方法是哪个版本加的..grepcode不知道为毛挂着代理也访问不了，一下子还不知道去哪里查，@hide 的API官方文档上压根就找不到..


考虑兼容性要正常使用的话 动画的位置需要用 DecorView的根容器置换一下，然后这一套私有API要全部扔掉，换成ViewGroup里的canvas<span class="pl-k">.</span>clipPath(CirclePath)去实现
