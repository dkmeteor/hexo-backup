title: Weak handler
date: 2015-11-11 12:08:47
tags:
---
#Weak Handler 与 内存泄露

Handler使用不当比较容易造成内存泄露.

比如这个例子:http://www.jianshu.com/p/c49f778e7acf

通常的原因就是 Handler的生命周期和Activity的生命周期不一致.

---

一个通用的场景是 使用 匿名内部类 实例 作为某个 行为/动作的 回调,如果该行为/动作 是异步的,则其返回时间往往无法确定,有造成内存泄露风险.

使用静态内部类,或者妥善处理生命周期,都不会造成内存泄露,反过来,当没有内存泄露风险时,一般直接匿名内部类即可.

这其实是一个特别矛盾的说法.

因为这要求程序员能 能了解到 回调行为是在何时发生的. 

而相反,我们设计接口回调的时候总是尽力屏蔽内部实现细节.

而面对不确定的行为,当然也可以使用静态内部类,或者removeCallback去取消回调(特指Handler),然而这样代码变得繁琐,或者添加不必要的代码.


对于Handler,一个简单的第三方解决方案是使用 android-weak-handler ,该库尝试使用WeakReference解决这个问题,然而却引入新的问题.


https://github.com/badoo/android-weak-handler


核心代码如下:

    public class WeakHandler {
		private final Handler.Callback mCallback; // hard reference to Callback. We need to keep callback in memory
		private final ExecHandler mExec;
		private Lock mLock = new ReentrantLock();
		@SuppressWarnings("ConstantConditions")
		@VisibleForTesting
		final ChainedRef mRunnables = new ChainedRef(mLock, null);

		public WeakHandler() {
			mCallback = null;
			mExec = new ExecHandler();
		}

		public WeakHandler(@Nullable Handler.Callback callback) {
			mCallback = callback; // Hard referencing body
			mExec = new ExecHandler(new WeakReference<>(callback)); // Weak referencing inside ExecHandler
		}

		public WeakHandler(@NonNull Looper looper) {
			mCallback = null;
			mExec = new ExecHandler(looper);
		}

		public WeakHandler(@NonNull Looper looper, @NonNull Handler.Callback callback) {
			mCallback = callback;
			mExec = new ExecHandler(looper, new WeakReference<>(callback));
		}

		public final boolean post(@NonNull Runnable r) {
			return mExec.post(wrapRunnable(r));
		}

		public final boolean postAtTime(@NonNull Runnable r, long uptimeMillis) {
			return mExec.postAtTime(wrapRunnable(r), uptimeMillis);
		}

		public final boolean postAtTime(Runnable r, Object token, long uptimeMillis) {
			return mExec.postAtTime(wrapRunnable(r), token, uptimeMillis);
		}

		public final boolean postDelayed(Runnable r, long delayMillis) {
			return mExec.postDelayed(wrapRunnable(r), delayMillis);
		}

		public final boolean postAtFrontOfQueue(Runnable r) {
			return mExec.postAtFrontOfQueue(wrapRunnable(r));
		}

		public final void removeCallbacks(Runnable r) {
			final WeakRunnable runnable = mRunnables.remove(r);
			if (runnable != null) {
				mExec.removeCallbacks(runnable);
			}
		}

		public final void removeCallbacks(Runnable r, Object token) {
			final WeakRunnable runnable = mRunnables.remove(r);
			if (runnable != null) {
				mExec.removeCallbacks(runnable, token);
			}
		}

		public final boolean sendMessage(Message msg) {
			return mExec.sendMessage(msg);
		}

		public final boolean sendEmptyMessage(int what) {
			return mExec.sendEmptyMessage(what);
		}

		public final boolean sendEmptyMessageDelayed(int what, long delayMillis) {
			return mExec.sendEmptyMessageDelayed(what, delayMillis);
		}

		public final boolean sendEmptyMessageAtTime(int what, long uptimeMillis) {
			return mExec.sendEmptyMessageAtTime(what, uptimeMillis);
		}

		public final boolean sendMessageDelayed(Message msg, long delayMillis) {
			return mExec.sendMessageDelayed(msg, delayMillis);
		}

		public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
			return mExec.sendMessageAtTime(msg, uptimeMillis);
		}

		public final boolean sendMessageAtFrontOfQueue(Message msg) {
			return mExec.sendMessageAtFrontOfQueue(msg);
		}

		public final void removeMessages(int what) {
			mExec.removeMessages(what);
		}

		public final void removeMessages(int what, Object object) {
			mExec.removeMessages(what, object);
		}

		public final void removeCallbacksAndMessages(Object token) {
			mExec.removeCallbacksAndMessages(token);
		}

		public final boolean hasMessages(int what) {
			return mExec.hasMessages(what);
		}

		public final boolean hasMessages(int what, Object object) {
			return mExec.hasMessages(what, object);
		}

		public final Looper getLooper() {
			return mExec.getLooper();
		}

		private WeakRunnable wrapRunnable(@NonNull Runnable r) {
			//noinspection ConstantConditions
			if (r == null) {
				throw new NullPointerException("Runnable can't be null");
			}
			final ChainedRef hardRef = new ChainedRef(mLock, r);
			mRunnables.insertAfter(hardRef);
			return hardRef.wrapper;
		}

		private static class ExecHandler extends Handler {
			private final WeakReference<Handler.Callback> mCallback;

			ExecHandler() {
				mCallback = null;
			}

			ExecHandler(WeakReference<Handler.Callback> callback) {
				mCallback = callback;
			}

			ExecHandler(Looper looper) {
				super(looper);
				mCallback = null;
			}

			ExecHandler(Looper looper, WeakReference<Handler.Callback> callback) {
				super(looper);
				mCallback = callback;
			}

			@Override
			public void handleMessage(@NonNull Message msg) {
				if (mCallback == null) {
					return;
				}
				final Handler.Callback callback = mCallback.get();
				if (callback == null) { // Already disposed
					return;
				}
				callback.handleMessage(msg);
			}
		}

		static class WeakRunnable implements Runnable {
			private final WeakReference<Runnable> mDelegate;
			private final WeakReference<ChainedRef> mReference;

			WeakRunnable(WeakReference<Runnable> delegate, WeakReference<ChainedRef> reference) {
				mDelegate = delegate;
				mReference = reference;
			}

			@Override
			public void run() {
				final Runnable delegate = mDelegate.get();
				final ChainedRef reference = mReference.get();
				if (reference != null) {
					reference.remove();
				}
				if (delegate != null) {
					delegate.run();
				}
			}
		}

		static class ChainedRef {
			@Nullable
			ChainedRef next;
			@Nullable
			ChainedRef prev;
			@NonNull
			final Runnable runnable;
			@NonNull
			final WeakRunnable wrapper;

			@NonNull
			Lock lock;

			public ChainedRef(@NonNull Lock lock, @NonNull Runnable r) {
				this.runnable = r;
				this.lock = lock;
				this.wrapper = new WeakRunnable(new WeakReference<>(r), new WeakReference<>(this));
			}

			public WeakRunnable remove() {
				lock.lock();
				try {
					if (prev != null) {
						prev.next = next;
					}
					if (next != null) {
						next.prev = prev;
					}
					prev = null;
					next = null;
				} finally {
					lock.unlock();
				}
				return wrapper;
			}

			public void insertAfter(@NonNull ChainedRef candidate) {
				lock.lock();
				try {
					if (this.next != null) {
						this.next.prev = candidate;
					}

					candidate.next = this.next;
					this.next = candidate;
					candidate.prev = this;
				} finally {
					lock.unlock();
				}
			}

			@Nullable
			public WeakRunnable remove(Runnable obj) {
				lock.lock();
				try {
					ChainedRef curr = this.next; // Skipping head
					while (curr != null) {
						if (curr.runnable == obj) { // We do comparison exactly how Handler does inside
							return curr.remove();
						}
						curr = curr.next;
					}
				} finally {
					lock.unlock();
				}
				return null;
			}
		}
	}
	
这个类的实现有个问题,为了避免持有Runnable的引用,使用WeakRunnable作为Wrapper,为了能够在Activity销毁时释放Runnable,这里使用的WeakReference去持有Runnable引用.
从目的上看,它确实解决了由于Runnable隐式持有Activity强引用而导致Acticity实例无法销毁的问题

**但是**

作为匿名内部类被传入的Runnable对象如果只存在一个WeakReference,那意味着,任何一次GC操作都将导致其被回收.
就看他自己写的例子


	import com.badoo.mobile.util.WeakHandler;

	public class ExampleActivity extends Activity {

		private WeakHandler mHandler; // We still need at least one hard reference to WeakHandler

		protected void onCreate(Bundle savedInstanceState) {
			mHandler = new WeakHandler();
			...
		}

		private void onClick(View view) {
			mHandler.postDelayed(new Runnable() {
				view.setVisibility(View.INVISIBLE);
			}, 5000);
		}
	}


如果在这5s内系统触发了一次GC操作会怎么样? Runnable会被回收?

将Demo简单修改一下

	protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
       
        WeakHandler  mHandler = new WeakHandler();
        mHandler.postDelayed(new Runnable() {


            @Override
            public void run() {
                System.out.println("####################### Do Something");
                );
            }
        }, 10000);

        mHandler.postDelayed(new Runnable() {


            @Override
            public void run() {
                System.out.println("#######################GC");

                System.gc();
            }
        }, 5000);

    }


看看运行结果,GC操作发生了,而Do Something并没有执行,因为 GC动作发生时, WeakReference持有的Runnable已经被释放了.

通常来讲,GC操作发生的时间不可预料的,你无法预期等待回调的这几秒种内用户做了什么操作,而一旦触发了GC,则回调会被立刻取消

注意,Activity并未被关闭,Runnable回调也会被释放!

如果在大型项目中遇到这样的问题,排查一定是灾难性的.因为GC的不确定性,出现了回调丢失情况一定是随机的,难以预测的,难以重现的.


而处理这个问题的办法是,将mHandler从临时变量 换成 成Activity的一个字段.


然而通常程序员不一定意识这2种写法的区别,而且使用系统提供的Handler时,这2种写法是没有区别的.

最后我检查了下文档,作者还是写了一句话作为提示

>We still need at least one hard reference to WeakHandler

至于使用者是否能意识到问题所在我持怀疑态度.

第三方库就是容易出现这种问题,使用起来能很方便的解决很多问题,然而又很容易挖几个隐蔽的坑给你.
