title: Blur-Drawer
id: 224
categories:
  - android
  - Blog
date: 2015-02-08 15:56:18
tags:
---

![blur-drawer](http://dk-exp.com/wp-content/uploads/2015/02/blur-drawer.gif)

https://github.com/charbgr/BlurNavigationDrawer

对客户这个UX实在心里没底....赶紧了找了套备胎先放着....

回头让我再改我就用这一套...

&nbsp;

这一套代码和我的Folder-DrawerLayout一样 是直接从android.support.v4.widget.DrawerLayout继承下来的...

兼容和升级会非常方便

&nbsp;

Blur算法部分和我常用的那一套也一样。。。高版本走RenderScript，低版本用StackBlur...支持先ScaleDown再Blur..性能也OK..不用再做额外的性能优化..

考虑到Blur半径比较大...其实Scale还可以开大一点...

* * *

&nbsp;

目前看来唯一一个问题是 Blur的触发点在ActionBarDrawerToggle里的onDrawerSlide里....

项目目前把ActionBar干掉了...也没用ToolBar    顶上现在是个自定义Header....

不知道会有多大影响。。。

&nbsp;

不过最坏情况也不过把这部分逻辑摘出来扔在Drawerlayout的Listener里...时间上考虑还比较safe.