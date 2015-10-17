title: SurfaceView flashes black on load
categories:
  - android
date: 2015-10-17 05:37:52
tags:
---

打算更新一下[Bubble-Notification](https://github.com/dkmeteor/Bubble-Notification).

之前所有效果都是通过在WindowMananger上添加DropCover,然后绘制在屏幕最顶层,类似悬浮窗的处理方案,为此应用需要拥有SYSTEM_ALERT权限(这里其实我的误解,使用TYPE_APPLICATION就不需要了).

后来想到参考SwipeBack/ResideMenu之类类库的,可以在DecorView上操作这些东西,这样就不需要权限了.

修改本身蛮简单的,不过修改完后发现,在效果最开始,屏幕会闪烁一下.

断点调了一会,是在SurfaceView被添加进DropCover后发生的,此时SurfaceView上还未进行任何绘制.

奇怪的是第二次再添加SurfaceView时,又不会闪烁了,因为修改以前运行是正常的,排除了绘制的问题以后.

搜索了一下,发现了这个

http://www.mzule.com/%E9%81%BF%E5%85%8D-surfaceview-%E7%AC%AC%E4%B8%80%E6%AC%A1%E6%98%BE%E7%A4%BA%E5%AF%BC%E8%87%B4%E7%9A%84%E9%BB%91%E5%B1%8F/

http://stackoverflow.com/questions/8772862/surfaceview-flashes-black-on-load

防止链接失效摘录一下原文:

    当 SurfaceView 第一次在 Window 里面显示的时候，会触发 IWindowSession.relayout(…) 方法，该方法会 relayout 整个布局，导致屏幕黑屏一下。解决方案，就是在 Activity 加入一个默认就显示的 SurfaceView，可以通过设置它的宽度为 0，来避免用户看见它。这样在 Fragment 里面的 SurfaceView 就已经是第二个 SurfaceView 了。可以重用上一个 SurfaceView 的参数，避免的明显的屏幕黑屏。


StackOverFlow上那个答案是这样:

    I think I found the reason for the black flash. In my case I'm using a SurfaceView inside a Fragment and dynamically adding this fragment to the activity after some action. The moment when I add the fragment to the activity, the screen flashes black. I checked out grepcode for the SurfaceView source and here's what I found: when the surface view appears in the window the very fist time, it requests the window's parameters changing by calling a private IWindowSession.relayout(..) method. This method "gives" you a new frame, window, and window surface. I think the screen blinks right at that moment.

    The solution is pretty simple: if your window already has appropriate parameters it will not refresh all the window's stuff and the screen will not blink. The simplest solution is to add a 0px height plain SurfaceView to the first layout of your activity. This will recreate the window before the activity is shown on the screen, and when you set your second layout it will just continue using the window with the current parameters. I hope this helps.
    
    
在MainActivity上添加了一个0px*0px的SurfaceView以后,问题确实解决了.

---


但是这个解决方案实在太难看了,产生问题的原因也很反常.

查看SurfaceView的源码,可以找到上面所说的调用位置

    protected void updateWindow(boolean force, boolean redrawNeeded){
        ...
         relayoutResult = mSession.relayout(
                            mWindow, mWindow.mSeq, mLayout, mWidth, mHeight,
                                visible ? VISIBLE : GONE,
                                WindowManagerGlobal.RELAYOUT_DEFER_SURFACE_DESTROY,
                                mWinFrame, mOverscanInsets, mContentInsets,
                                mVisibleInsets, mStableInsets, mConfiguration, mNewSurface);
        ...
    } 


尝试了若干办法,绕过或者提前通过getWindow()获取Window对象设置参数都不行.


由于我是提供的类库,如果需要使用者自己去加个SurfaceView那也太操蛋了,这样Hack了一下

    
        public void init(Activity activity) {
		if (mDropCover == null) {
			mDropCover = new DropCover(activity);
			mContainer = new FrameLayout(activity);

			ViewGroup decor = (ViewGroup) activity.getWindow().getDecorView();
			decor.addView(mContainer);

			/**
			 *
			 *  WTF!
			 *
			 *  http://stackoverflow.com/questions/8772862/surfaceview-flashes-black-on-load
			 *
			 */
			SurfaceView s = new SurfaceView(activity);
			mContainer.addView(s);
			mContainer.post(new Runnable() {
				@Override
				public void run() {
					mContainer.removeAllViews();
				}
			});
		}

StackOverFlow上那个提问记录还是2012年的...

---

又仔细思索了一下,这种处理方式还是很操蛋,把这一部分重新改回了使用WindowMananger添加的方式

这里是我的理解有误

使用TYPE_APPLICATION就可以了,不需要权限.

之前使用的是TYPE_SYSTEM_ALERT,所以才需要权限,修改绕了一圈又回来了.

