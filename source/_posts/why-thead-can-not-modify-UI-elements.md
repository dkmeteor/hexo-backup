title: 从源码分析 对 非UI线程不允许访问UI元素的 理解
id: 304
categories:
  - android
  - Blog
date: 2015-05-05 16:43:11
tags:
---

在eoe回答了一个问题：
http://www.eoeandroid.com/forum.php?mod=viewthread&amp;tid=579938&amp;page=1

索性把内容整理一下...

---

做Android开发初期大部分开发者都会遇到下面这个问题.

    android.view.viewroot$calledfromwrongthreadexception: only the original thread that created a view hierarchy can touch its views.

原因大半都是 由于在异步线程中视图 更新界面中的元素

百度一搜,可以搜到大量解决办法.

结果提炼一下: 大致就是讲 UI元素/视图元素 必须在 UI线程(主线程)中更新修改.

新手这样理解就够了.

---

但是仔细看这个Exception说明

`only the original thread that created a view hierarchy can touch its views.`

里面并没有提到 主线程/UI 线程,只是提到,必须是创建`View hierarchy`的线程

!注意区分

创建View的线程和创建View hierarchy的不一定是同一个线程,不要理解错了,下面在源码中具体分析

观察ViewRoot源码中抛出calledfromwrongthreadexception的位置

    void checkThread() {

        if (mThread != Thread.currentThread()) {

            throw new CalledFromWrongThreadException(
                    "Only the original thread that created a view hierarchy can touch its views.");
        }
    }

可以注意到 这里只检查了 当前执行线程和 mThread是否为同一个线程

    public ViewRoot(Context context) {

         super();
         if (MEASURE_LATENCY &amp;&amp; lt == null) {
             lt = new LatencyTimer(100, 1000);
         }

         // For debug only
         //++sInstanceCount;
         // Initialize the statics when this class is first instantiated. This is
         // done here instead of in the static block because Zygote does not
         // allow the spawning of threads.

         getWindowSession(context.getMainLooper());

         mThread = Thread.currentThread();

由ViewRoot的构造函数可以看到,mThread被绑定在了 创建`viewRoot`的线程上.

当然大部分情况下,`ViewRoot`都是在主线程中创建的,所以在异步线程中修改view会造成checkThread失败.


mTextView.setText("123");为例子.

跟踪一下调用栈

    -- android.widget.TextView.setText

      -- android.widget.TextView.checkForRelayout

        -- android.view.View.invalidate

            -- android.view.ViewGroup.invalidateChild

              -- android.view.ViewRoot.invalidateChildInParent

                -- android.view.ViewRoot.invalidateChild

                  -- android.view.ViewRoot.checkThread

可以看到 view会不断遍历去获取 ViewParent
一个视图上的View遍历到最后 就是 获取到了 ViewRoot


(ViewRoot本身就是ViewParent的一个子类)

然后就会调用到上面提到的 checkThread()

到这里有一个结论:

UI修改操作确实会验证操作的线程

    * * *

回到标题上来说, 因为 ViewRoot是由UI线程创建的,所以所有被添加到ViewRoot下的子View都必须在UI线程中修改,看起来也很合理.
那么,有可能在非UI线程中创建ViewRoot吗?

其实是可以的.

这里就涉及到ViewRoot和WindowManager之间的关系

demo代码如下:

    package com.dk.asyncui;

    import android.app.Activity;
    import android.graphics.PixelFormat;
    import android.os.Bundle;
    import android.os.Looper;
    import android.view.Menu;
    import android.view.MenuItem;
    import android.view.View;
    import android.view.WindowManager;
    import android.widget.Button;

    public class MainActivity extends Activity {
        private Button mButton ;
        private WindowManager mWindowManager ;

        @Override
        protected void onCreate(Bundle savedInstanceState) {
               super.onCreate(savedInstanceState);
              setContentView(R.layout. activity_main);

               mWindowManager = (WindowManager) getSystemService("window" );

               mButton = new Button(this );
               mButton.setText( "HelloWorld");

              Thread mthread = new Thread(new Runnable() {

                      @Override
                      public void run() {
                           Looper. prepare();
                           WindowManager.LayoutParams wmParams = new WindowManager.LayoutParams();
                           wmParams. type = WindowManager.LayoutParams.TYPE_APPLICATION ;
                           wmParams. format = PixelFormat. RGBA_8888;
                           wmParams. flags = WindowManager.LayoutParams.FLAG_NOT_TOUCH_MODAL ;
                           wmParams. width = 320;
                           wmParams. height = 140;
                            mWindowManager.addView(mButton , wmParams);
                           Looper. loop();

                     }
              });
              mthread.start();
               mButton.setOnClickListener( new View.OnClickListener() {

                      @Override
                      public void onClick(View v) {

                           runOnUiThread( new Runnable() {
                                   public void run() {
                                          mButton.setText( "22222222");
                                  }
                           });
                     }
              });
       }

    }

可以看到,我在异步线程中将 mButton添加到 WindowManager中

查看 WindowManangerImpl中addView的代码：

    public void addView(View view, ViewGroup.LayoutParams params,
            Display display, Window parentWindow) {
        if (view == null) {
            throw new IllegalArgumentException("view must not be null");
        }

        if (display == null) {
            throw new IllegalArgumentException("display must not be null");
        }

        if (!(params instanceof WindowManager.LayoutParams)) {
            throw new IllegalArgumentException("Params must be WindowManager.LayoutParams");
        }
        final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams)params;
        if (parentWindow != null) {
            parentWindow.adjustLayoutParamsForSubWindow(wparams);
        }
        ViewRootImpl root;
        View panelParentView = null;

        synchronized (mLock) {
            // Start watching for system property changes.

            if (mSystemPropertyUpdater == null) {
                mSystemPropertyUpdater = new Runnable() {
                    @Override public void run() {
                        synchronized (mLock) {
                            for (int i = mRoots.size() - 1; i &gt;= 0; --i) {
                                mRoots.get(i).loadSystemProperties();
                            }
                        }
                    }
                };
                SystemProperties.addChangeCallback(mSystemPropertyUpdater);
            }
            int index = findViewLocked(view, false);
            if (index &gt;= 0) {
                if (mDyingViews.contains(view)) {
                    // Don't wait for MSG_DIE to make it's way through root's queue.
                    mRoots.get(index).doDie();
                } else {
                    throw new IllegalStateException("View " + view
                            + " has already been added to the window manager.");
                }
                // The previous removeView() had not completed executing. Now it has.
            }
            // If this is a panel window, then find the window it is being
            // attached to for future reference.
            if (wparams.type &gt;= WindowManager.LayoutParams.FIRST_SUB_WINDOW &amp;&amp;
                    wparams.type &lt;= WindowManager.LayoutParams.LAST_SUB_WINDOW) {
                final int count = mViews.size();
                for (int i = 0; i &lt; count; i++) {
                    if (mRoots.get(i).mWindow.asBinder() == wparams.token) {
                        panelParentView = mViews.get(i);
                    }
                }
            }
            root = new ViewRootImpl(view.getContext(), display);
            view.setLayoutParams(wparams);
            mViews.add(view);
            mRoots.add(root);
            mParams.add(wparams);
        }

        // do this last because it fires off messages to start doing things
        try {
            root.setView(view, wparams, panelParentView);
        } catch (RuntimeException e) {
            // BadTokenException or InvalidDisplayException, clean up.
            synchronized (mLock) {
                final int index = findViewLocked(view, false);
                if (index &gt;= 0) {
                    removeViewLocked(index, true);
                }
            }
            throw e;
        }
    }

可以发现在addView的过程中 WindowMananger创建了 ViewRoot

这就是上面提到的在 异步线程中创建 ViewRoot

上面Demo代码中的 runOnUiThread是为了保证下面的
  mButton.setText( "22222222" );

是在主线程中执行
可以看到,这段代码即使是在主线程中执行,依然抛出了CalledFromWrongThreadException

在特意构造的这个特殊场景下,主线程/UI线程中 不能更新在异步线程中"创建/添加"的UI元素!!

* * *

从源码和设计的角度上分析,我倾向于认为这其实是设计上的一个漏洞,我通过这个漏洞创建了一个非主线程绑定的ViewRoot对象,才会导致上面的结果.