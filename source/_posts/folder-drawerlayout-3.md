title: Folder-DrawerLayout 开发日志2
id: 168
categories:
  - Blog
date: 2015-01-12 16:59:42
tags:
---

&nbsp;

[![Folder-DrawerLayout](http://dk-exp.com/wp-content/uploads/2015/01/Folder-DrawerLayout1.gif)](http://dk-exp.com/wp-content/uploads/2015/01/Folder-DrawerLayout1.gif)

&nbsp;

[![Folder-DrawerLayout2](http://dk-exp.com/wp-content/uploads/2015/01/Folder-DrawerLayout2.gif)](http://dk-exp.com/wp-content/uploads/2015/01/Folder-DrawerLayout2.gif)

&nbsp;

弧形褶皱添加完毕.....有些bug....

数值上也需要调整一下....需要构造一个 x=f(y)

当 y=0 或 y=height 时   f(y) = 0 的函数 。。

本来是写了个二次函数。。但是二次函数是对称的。。只能保证中点在 height/2时满足条件....形状和斜率太对称很违和，不够自然...

&nbsp;

可能写2个2次函数拼起来也可以....反正y轴上其实只有6个采样点.....都是插值插出来的....曲线不光滑也看不出来..........

&nbsp;

* * *

&nbsp;

另外一个隐患是函数太复杂了。。。本来是为了清晰起见把每个步骤都分开计算。。

循环上有些冗余。。。但是对比图形计算的部分。。浪费的性能应该不在一个数量级上。。

但是类似这样的代码

<pre class="lang:java decode:true"> private float[] applyScaleXEffect(float offset) {
        for (int i = 0; i &lt; 6; i++)
            for (int j = 0; j &lt; 51; j++) {
                meshVerts[i * 102 + 2 * j] = meshVerts[i * 102 + 2 * j]
                        * (1.0f + 0.0f * offset) ;

                meshVerts[i * 102 + 2 * j]=(offset) * meshVerts[i * 102 + 2 * j] *  (1+ (meshVerts[i * 102 + 2 * j+1]-1000)*(meshVerts[i * 102 + 2 * j+1]-1000)/10000 /width);
            }
        return meshVerts;
    }</pre>

可阅读性已经基本没有了。。。。考虑到性能问题。。。最后可能还要把冗余的循环计算都合并起来。。。最后可能变成一个超长的公式。。。。。。。估计要做到一次成型。。然后再也不维护了。。

&nbsp;

可阅读性要求很高。。。但是对性能又很敏感。。。。。我想想到底要怎么处理这个结构。。。

如果证实冗余的循环对性能影响不大的话。。。还是尽量把每个步骤分开写吧。。。优化的问题扔最后了

&nbsp;

数值调整完以后要赶紧把 能拆出来的变量拆出来，能写成常量的写成常量。。不然过几天鬼还记得这个10000是怎么算出来的....

&nbsp;

&nbsp;

* * *

&nbsp;

&nbsp;

顺便发现用Google看函数图挑形状真是方便..

[![google](http://dk-exp.com/wp-content/uploads/2015/01/google-300x221.jpg)](http://dk-exp.com/wp-content/uploads/2015/01/google.jpg)

&nbsp;