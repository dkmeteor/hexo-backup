title: CoordinatorLayout中的坑
date: 2016-03-30 09:56:02
tags: android
---

踩完了坑记录一下，CoordinatorLayout使用介绍请看chrisbanes的[cheesesquare](https://github.com/chrisbanes/cheesesquare)

如果你只打算学习一下CoordinatorLayout然后写2个Demo试试，那么本文并没有什么卵用，但是如果你打算在生产环境使用CoordinatorLayout，那么强烈推荐阅读一下本文，可以减少很多弯路，这个东西看起来很好，但是实际上坑也很多。

###前言
很多应用主页常见的构造模式

一个包含ActionBar和Banner的header+ViewPager的组合模式
比如这样：

![img](/images/appbar_extend.png)


然后需要滚动的时候能够将 ActionBar和Banner滚出界面，但是又需要ViewPager的TabLayout能固定在屏幕顶部
比如滚动后是这样：


![img](/images/appbar_close.png)

Github上有很多sticky-viewpager xx-header-viewpager之类的项目提供类似的功能，但是大部分都会存在各种事件冲突问题，比如滑动不流畅，卡顿，弹跳之类的。

好在Android新的SupportLibrary提供了CoordinatorLayout能直接实现类似功能，具体如何使用参考 SupportLibrary中CoordinatorLayout的使用文档即可，例子也可以直接看[cheesesquare](https://github.com/chrisbanes/cheesesquare)

我就谈下使用过程中里面的坑：

#### 不支持ListView,不支持ScrollView，低版本不兼容ViewCompat.setNestedScrollingEnabled


PS.更新

support 23.1+新增了

    ViewCompat.setNestedScrollingEnabled(listView/gridview,true); 

可以给ListView提供NestedScrolling支持,但是只在LOLIPOP+版本上生效，下面的方案也一样。

低版本只能使用RecyclerView

---

一些文档/博客文章上说 可以使用一个 可滚动的组件
还有些错误的文章说 支持ListView

实际上该组件必须实现了 NestedScrollingChild接口

你可以使用RecyclerView,RecyclerView自身就实现了该接口，如果你要使用其他组件，那么可能需要继承原来的View并实现NestedScrollingChild接口

一个实现了NestedScrollingChild的Listview demo如下：

    public class NestedScrollingListView extends ListView implements NestedScrollingChild {

	    private final NestedScrollingChildHelper mScrollingChildHelper;

	    public NestedScrollingListView(Context context) {
	        super(context);
	        mScrollingChildHelper = new NestedScrollingChildHelper(this);
	        setNestedScrollingEnabled(true);
	    }

	    public NestedScrollingListView(Context context, AttributeSet attrs) {
	        super(context, attrs);
	        mScrollingChildHelper = new NestedScrollingChildHelper(this);
	        setNestedScrollingEnabled(true);
	    }

	    @Override
	    public void setNestedScrollingEnabled(boolean enabled) {
	        mScrollingChildHelper.setNestedScrollingEnabled(enabled);
	    }

	    @Override
	    public boolean isNestedScrollingEnabled() {
	        return mScrollingChildHelper.isNestedScrollingEnabled();
	    }

	    @Override
	    public boolean startNestedScroll(int axes) {
	        return mScrollingChildHelper.startNestedScroll(axes);
	    }

	    @Override
	    public void stopNestedScroll() {
	        mScrollingChildHelper.stopNestedScroll();
	    }

	    @Override
	    public boolean hasNestedScrollingParent() {
	        return mScrollingChildHelper.hasNestedScrollingParent();
	    }

	    @Override
	    public boolean dispatchNestedScroll(int dxConsumed, int dyConsumed, int dxUnconsumed,
	                                        int dyUnconsumed, int[] offsetInWindow) {
	        return mScrollingChildHelper.dispatchNestedScroll(dxConsumed, dyConsumed,
	                dxUnconsumed, dyUnconsumed, offsetInWindow);
	    }

	    @Override
	    public boolean dispatchNestedPreScroll(int dx, int dy, int[] consumed, int[] offsetInWindow) {
	        return mScrollingChildHelper.dispatchNestedPreScroll(dx, dy, consumed, offsetInWindow);
	    }

	    @Override
	    public boolean dispatchNestedFling(float velocityX, float velocityY, boolean consumed) {
	        return mScrollingChildHelper.dispatchNestedFling(velocityX, velocityY, consumed);
	    }

	    @Override
	    public boolean dispatchNestedPreFling(float velocityX, float velocityY) {
	        return mScrollingChildHelper.dispatchNestedPreFling(velocityX, velocityY);
	    }
	}


其他View也可以使用类似的方式实现。

只对 5.0+ 生效，低版本无效。


#### RecyclerView滑动的时候不流畅

目前已知 r23.1 & r 23.2  都存在此问题，Google code的提交代码里倒是有相关修复记录，但是目前还没有发布。

直接原因是Fling Direction错误，导致Fling事件被吃掉了，ACTION_UP/ACTION_CANCEL事件一旦发生，scroll动作直接停止

StackOverFlow上一个高票解决方案是重写一个FlingBehavior，并配置给AppBar.

注意，这个Behavior是给AppBar的，不是给下面的可滚动组件的。

	<android.support.design.widget.AppBarLayout
    android:id="@+id/appbar"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    app:layout_behavior="your.package.FlingBehavior">
    <!--your views here-->
 	</android.support.design.widget.AppBarLayout>

---


	public class FlingBehavior extends AppBarLayout.Behavior {

	    private static final int TOP_CHILD_FLING_THRESHOLD = 3;
	    private boolean isPositive;

	    public FlingBehavior() {
	    }

	    public FlingBehavior(Context context, AttributeSet attrs) {
	        super(context, attrs);
	    }

	    @Override
	    public boolean onNestedFling(CoordinatorLayout coordinatorLayout, AppBarLayout child, View target, float velocityX, float velocityY, boolean consumed) {
	        if (velocityY > 0 && !isPositive || velocityY < 0 && isPositive) {
	            velocityY = velocityY * -1;
	        }
	        if (target instanceof RecyclerView && velocityY < 0) {
	            final RecyclerView recyclerView = (RecyclerView) target;
	            final View firstChild = recyclerView.getChildAt(0);
	            final int childAdapterPosition = recyclerView.getChildAdapterPosition(firstChild);
	            consumed = childAdapterPosition > TOP_CHILD_FLING_THRESHOLD;
	        }
	        consumed=false;
	        return super.onNestedFling(coordinatorLayout, child, target, velocityX, velocityY, consumed);
	    }

	    @Override
	    public void onNestedPreScroll(CoordinatorLayout coordinatorLayout, AppBarLayout child, View target, int dx, int dy, int[] consumed) {
	        super.onNestedPreScroll(coordinatorLayout, child, target, dx, dy, consumed);
	        isPositive = dy > 0;
	    }

注意consumed计算这部分，会影响滚动形式

请参考父类滚动的实现代码：

	
 	@Override
    public boolean onNestedFling(final CoordinatorLayout coordinatorLayout,
                final AppBarLayout child, View target, float velocityX, float velocityY,
                boolean consumed) {
            boolean flung = false;

            if (!consumed) {
                // It has been consumed so let's fling ourselves
                flung = fling(coordinatorLayout, child, -child.getTotalScrollRange(),
                        0, -velocityY);
            } else {
                // If we're scrolling up and the child also consumed the fling. We'll fake scroll
                // upto our 'collapsed' offset
                if (velocityY < 0) {
                    // We're scrolling down
                    final int targetScroll = -child.getTotalScrollRange()
                            + child.getDownNestedPreScrollRange();
                    if (getTopBottomOffsetForScrollingSibling() < targetScroll) {
                        // If we're currently not expanded more than the target scroll, we'll
                        // animate a fling
                        animateOffsetTo(coordinatorLayout, child, targetScroll);
                        flung = true;
                    }
                } else {
                    // We're scrolling up
                    final int targetScroll = -child.getUpNestedPreScrollRange();
                    if (getTopBottomOffsetForScrollingSibling() > targetScroll) {
                        // If we're currently not expanded less than the target scroll, we'll
                        // animate a fling
                        animateOffsetTo(coordinatorLayout, child, targetScroll);
                        flung = true;
                    }
                }
            }

            mWasNestedFlung = flung;
            return flung;
        }

确认你需要滚动哪一步分，你可以简单根据RecyclerView展示的first item的position来判断Recyclerview是否在top位置，也有另一种代码如下：

	@Override
    public boolean onNestedFling(CoordinatorLayout coordinatorLayout, AppBarLayout child, View target, float velocityX, float velocityY, boolean consumed) {
        if (target instanceof RecyclerView) {
            final RecyclerView recyclerView = (RecyclerView) target;
            consumed = velocityY > 0 || recyclerView.computeVerticalScrollOffset() > 0;
        }
        return super.onNestedFling(coordinatorLayout, child, target, velocityX, velocityY, consumed);
    }





另一个解决方案是使用 [smooth-app-bar-layout](https://github.com/henrytao-me/smooth-app-bar-layout)

具体如何使用请参考它自己的文档

当header是Toolbar时，demo运作良好，当headerview高度较大时，滑动会发生剧烈抖动，弹跳，暂时还没去看它源代码，可能是我使用不正确


#### 不支持adjustResize
可以参考这个问题
http://stackoverflow.com/questions/35599125/adjustresize-does-not-work-with-coordinatorlayout

解决方案是可以通过ViewTreeObserver.OnGlobalLayoutListener监听界面改变，然后人工处理padding bottom

	public class KeyboardUtil {
	    private View decorView;
	    private View contentView;

	    public KeyboardUtil(Activity act, View contentView) {
	        this.decorView = act.getWindow().getDecorView();
	        this.contentView = contentView;

	        //only required on newer android versions. it was working on API level 19
	        if (Build.VERSION.SDK_INT >= 19) {
	            decorView.getViewTreeObserver().addOnGlobalLayoutListener(onGlobalLayoutListener);
	        }
	    }

	    public void enable() {
	        if (Build.VERSION.SDK_INT >= 19) {
	            decorView.getViewTreeObserver().addOnGlobalLayoutListener(onGlobalLayoutListener);
	        }
	    }

	    public void disable() {
	        if (Build.VERSION.SDK_INT >= 19) {
	            decorView.getViewTreeObserver().removeOnGlobalLayoutListener(onGlobalLayoutListener);
	        }
	    }


	    //a small helper to allow showing the editText focus
	    ViewTreeObserver.OnGlobalLayoutListener onGlobalLayoutListener = new ViewTreeObserver.OnGlobalLayoutListener() {
	        @Override
	        public void onGlobalLayout() {
	            Rect r = new Rect();
	            //r will be populated with the coordinates of your view that area still visible.
	            decorView.getWindowVisibleDisplayFrame(r);

	            //get screen height and calculate the difference with the useable area from the r
	            int height = decorView.getContext().getResources().getDisplayMetrics().heightPixels;
	            int diff = height - r.bottom;

	            //if it could be a keyboard add the padding to the view
	            if (diff != 0) {
	                // if the use-able screen height differs from the total screen height we assume that it shows a keyboard now
	                //check if the padding is 0 (if yes set the padding for the keyboard)
	                if (contentView.getPaddingBottom() != diff) {
	                    //set the padding of the contentView for the keyboard
	                    contentView.setPadding(0, 0, 0, diff);
	                }
	            } else {
	                //check if the padding is != 0 (if yes reset the padding)
	                if (contentView.getPaddingBottom() != 0) {
	                    //reset the padding of the contentView
	                    contentView.setPadding(0, 0, 0, 0);
	                }
	            }
	        }
	    };


	    /**
	     * Helper to hide the keyboard
	     *
	     * @param act
	     */
	    public static void hideKeyboard(Activity act) {
	        if (act != null && act.getCurrentFocus() != null) {
	            InputMethodManager inputMethodManager = (InputMethodManager) act.getSystemService(Activity.INPUT_METHOD_SERVICE);
	            inputMethodManager.hideSoftInputFromWindow(act.getCurrentFocus().getWindowToken(), 0);
	        }
	    }
	}

---

	KeyboardUtil keyboardUtil = new KeyboardUtil(this, findViewById(android.R.id.content));

	//enable it
	keyboardUtil.enable();


---

ps.  
fitSystemWindow 和 adjustResize 和 FLAG_TRANSLUCENT_STATUS 一起使用 一样也会有冲突，造成界面缩放不正常。  当时也是用的这个解决方案。

### CoordinatorLayout的onScrollListener只支持21+

解决方案：
在Behavior里监听滚动，并把数据传出来

代码和后一个问题贴在一起

注意 我这里只监听了 onDependentViewChanged
并在这里调用onScroll接口，然后外部实际使用的数据是 headerview.getTop , 并不能覆盖全部情况，但是我本身只用来做状态栏渐变色，有其它使用场景请自行修改

### 当顶部容器不使用Toolbar时，Measure会有问题

解决方案：
自定义一个 Behavior,继承AppBarLayout.ScrollingViewBehavior

并重载onMeasureChild方法

	public class PatchedScrollingViewBehavior extends AppBarLayout.ScrollingViewBehavior {

	    private OnScrollListener onScrollListener;

	    public PatchedScrollingViewBehavior() {
	        super();
	    }

	    public PatchedScrollingViewBehavior(Context context, AttributeSet attrs) {
	        super(context, attrs);
	    }
	   
	    @Override
	    public boolean onMeasureChild(CoordinatorLayout parent, View child, int parentWidthMeasureSpec, int widthUsed, int parentHeightMeasureSpec, int heightUsed) {

	        Log.e("####", "onMeasureChild");


	        if (child.getLayoutParams().height == -1) {
	            List dependencies = parent.getDependencies(child);
	            if (dependencies.isEmpty()) {
	                return false;
	            }

	            AppBarLayout appBar = findFirstAppBarLayout(dependencies);
	            if (appBar != null && ViewCompat.isLaidOut(appBar)) {
	                if (ViewCompat.getFitsSystemWindows(appBar)) {
	                    ViewCompat.setFitsSystemWindows(child, true);
	                }

	                int scrollRange = appBar.getTotalScrollRange();
	//                int height = parent.getHeight() - appBar.getMeasuredHeight() + scrollRange;
	                int parentHeight = View.MeasureSpec.getSize(parentHeightMeasureSpec);
	                int height = parentHeight - appBar.getMeasuredHeight() + scrollRange;
	                int heightMeasureSpec = View.MeasureSpec.makeMeasureSpec(height, View.MeasureSpec.AT_MOST);
	                parent.onMeasureChild(child, parentWidthMeasureSpec, widthUsed, heightMeasureSpec, heightUsed);
	                return true;
	            }
	        }

	        return false;
	    }

	    @Override
	    public void onNestedScroll(CoordinatorLayout coordinatorLayout, View child, View target, int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed) {
	        super.onNestedScroll(coordinatorLayout, child, target, dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed);
	    }

	    private static AppBarLayout findFirstAppBarLayout(List<View> views) {
	        int i = 0;
	        for (int z = views.size(); i < z; ++i) {
	            View view = (View) views.get(i);
	            if (view instanceof AppBarLayout) {
	                return (AppBarLayout) view;
	            }
	        }
	        return null;
	    }

	    @Override
	    public boolean onDependentViewChanged(CoordinatorLayout parent, View child,
	                                          View dependency) {
	        if (onScrollListener != null) {
	            onScrollListener.onScroll(parent, child, dependency);
	        }
	        return super.onDependentViewChanged(parent, child, dependency);
	    }

	    public interface OnScrollListener {
	        void onScroll(CoordinatorLayout parent, View child,
	                      View dependency);

	    }

	    public OnScrollListener getOnScrollListener() {
	        return onScrollListener;
	    }

	    public void setOnScrollListener(OnScrollListener onScrollListener) {
	        this.onScrollListener = onScrollListener;
	    }
	}



REF:
http://stackoverflow.com/questions/30923889/flinging-with-recyclerview-appbarlayout 
https://github.com/henrytao-me/smooth-app-bar-layout 
https://code.google.com/p/android/issues/detail?id=177729 





