title: CircleAnimation Update
id: 276
categories:
  - android
  - Blog
date: 2015-03-22 18:35:39
tags:
---

[![circle_animation2](http://dk-exp.com/wp-content/uploads/2015/03/circle_animation2.gif)](http://dk-exp.com/wp-content/uploads/2015/03/circle_animation2.gif)



修改了实现方式，不再依赖RevealAnimator

也不再依赖

    /** @hide */
    public void setRevealClip(boolean shouldClip, float x, float y, float radius)</pre>

删除了反射的setRevealClip的部分

* * *

新的实现完全基于老的Canvas API，兼容低版本

* * *


缩放的部分加了个bounce效果，不过效果并不好，估计是系数选的不对，看来还是去找一套算好的系数。

最后再把相关接口调整一下，一坨AnimatorSet的嵌套逻辑最后用 playSequentially 整理一下应该就可以结束了。

* * *


顺带 看了下 CircularReveal https://github.com/ozodrukh/CircularReveal/ 的源代码

用这个方案来实现也可以，不过

先draw再clipPath

和

drawCircle with BitmapShader



从结果上来看是完全等价的。

以后需要在ViewGroup上实现该效果可能才用得上这个方案

