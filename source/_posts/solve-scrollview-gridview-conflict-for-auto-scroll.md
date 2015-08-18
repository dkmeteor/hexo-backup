title: ScrollView 中嵌套 GridView 导致 ScrollView 默认不停留在顶部的解决方案和分析
id: 247
categories:
  - android
  - Blog
date: 2015-02-27 14:29:01
tags:
---
`ScrollView`中嵌套 `GridView` 导致 `ScrollView` 默认不停留在顶部的解决方案和分析

发生情况大概是我在ScrollView顶部放了一个ViewPager用来做广告Banner,底部放了个GridVie, 来实现一个类似9宫格效果的展示.

然后出现的状况是,当我获取完数据并调用notifyDataSetChanged();后 ScrollView自动滚到了最底部,也就是GridView所在的位置.

研究了一下,获取了一些解决方案

-- 让界面顶部的某一个View获取focus
-- grid.setFocusable(false);
-- 手动scrollto(0,0)
-- 重写ScrollView中的computeScrollDeltaToGetChildRectOnScreen,让该方法返回0

目前简单的用setFocusable(false)解决了该问题

试着分析一下这个问题产生的原因. 从解决方案反推,可以发现这个问题和 `focus` 有关系

一个猜测是 notifyDataSetChanged()之后,grid由于加载了数据的关系高度产生了变化

这导致了ScrollView内部重新走了 onLayout / onMeaure 流程 在这个流程中 ScrollView会将自身滚动到 获得 focus 的 child 位置

上面关于focus的解决方案即是从这个角度去解决问题

跟踪一下调用链


    protected void onLayout(boolean changed, int l, int t, int r, int b) {
     super.onLayout(changed, l, t, r, b);
     mIsLayoutDirty = false;
     // Give a child focus if it needs it
     if (mChildToScrollTo != null && isViewDescendantOf(mChildToScrollTo, this)) {
     scrollToChild(mChildToScrollTo);
     }
     ...
    }


可以看到 onLayout 的时候确实会将ScrollView滚动到focus child位置

    private void scrollToChild(View child) {
     child.getDrawingRect(mTempRect);
    
     /* Offset from child's local coordinates to ScrollView coordinates */
     offsetDescendantRectToMyCoords(child, mTempRect);
    
     int scrollDelta = computeScrollDeltaToGetChildRectOnScreen(mTempRect);
    
     if (scrollDelta != 0) {
     scrollBy(0, scrollDelta);
     }
    }

从代码逻辑上来看 避免 GridView获取focus可以解决该问题.

而重写ScrollView中的computeScrollDeltaToGetChildRectOnScreen则是从另一个角度解决该问题

而scrollToChild这个方法会根据computeScrollDeltaToGetChildRectOnScreen的返回值来计算滚动的位置

重载computeScrollDeltaToGetChildRectOnScreen让其返回0 会导致ScrollView内布局产生变化时,不能正确滚动到focus child位置,当然,这也就是我们想要的效果,布局变化时ScrollView不需要自己去滚动.

至于computeScrollDeltaToGetChildRectOnScreen代码太长就不贴了

大致代码是 根据当前 scrollY和focus child 的 rect.bottom 去计算要滚到哪

逻辑理顺以后觉得这个问题也没什么奇怪的.
