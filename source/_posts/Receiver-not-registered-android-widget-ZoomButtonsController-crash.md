title: 'Receiver not registered: android.widget.ZoomButtonsController crash'
date: 2016-05-31 12:28:39
tags: android
---
#好吧,继续整理 最近一段时间遇到的比较离奇的Bug

这个Bug log是这样

	java.lang.IllegalArgumentException: Receiver not registered: android.widget.ZoomButtonsController$1@487a4290
	at android.app.ActivityThread$PackageInfo.forgetReceiverDispatcher(ActivityThread.java:793)
	at android.app.ContextImpl.unregisterReceiver(ContextImpl.java:913)
	at android.content.ContextWrapper.unregisterReceiver(ContextWrapper.java:331)
	at android.widget.ZoomButtonsController.setVisible(ZoomButtonsController.java:404)
	at android.widget.ZoomButtonsController$2.handleMessage(ZoomButtonsController.java:178)
	at android.os.Handler.dispatchMessage(Handler.java:99)
	at android.os.Looper.loop(Looper.java:123)
	at android.app.ActivityThread.main(ActivityThread.java:4627)
	at java.lang.reflect.Method.invokeNative(Native Method)
	at java.lang.reflect.Method.invoke(Method.java:521)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:858)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:616)
	at dalvik.system.NativeStart.main(Native Method)

因为这个Log Trace里没有任何我的代码,所以感觉上和我的应用没什么关系,而且也看不出是在哪个界面哪个地方发生的,当时也没办法复现.
只是在Umeng的后台会不断收集到这样的crash log,不多,但是偶尔就会有那么几个.
后来花时间研究了一下,发现应用中只有Webview中有这个东西.

这样有针对性的来测试,很快就找到了重现路径. zoom controller有个默认的渐隐动画,只要在动画进行重退出Activity,就会重现这个crash

同时也找到了几个相关issue:
https://code.google.com/p/android/issues/detail?id=15694
http://stackoverflow.com/questions/5267639/how-to-safely-turn-webview-zooming-on-and-off-as-needed
http://stackoverflow.com/questions/4908794/webview-throws-receiver-not-registered-android-widget-zoombuttonscontroller

解决方案是各种hack
1. 如果不需要直接禁用即可
2. 重载webview,destroy的时候处理zoomButtonController
3. 重载Activity,延迟2秒destroy webview
4. finish之前把webview及所有子view从rootview里remove掉. (这个没试过)