title: ArrayMap-synchronization-problem
date: 2017-05-18 17:15:02
tags: Android
---

一个比较难搞的Bug,有遇到的可以参考一下  
难搞的原因和以前一样, 如果一个bug出现在google issue里 又没有 给出 workaround,那基本就没救了


错误输出为 java.lang.String cannot be cast to java.lang.Object[]

    java.lang.ClassCastException: java.lang.String cannot be cast to java.lang.Object[]
	at android.util.ArrayMap.allocArrays(ArrayMap.java:187)
	at android.util.ArrayMap.put(ArrayMap.java:459)
	at android.app.ActivityThread.handleCreateService(ActivityThread.java:2913)
	at android.app.ActivityThread.access$1900(ActivityThread.java:165)
	at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1459)
	at android.os.Handler.dispatchMessage(Handler.java:102)
	at android.os.Looper.loop(Looper.java:150)
	at android.app.ActivityThread.main(ActivityThread.java:5621)
	at java.lang.reflect.Method.invoke(Native Method)
	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:794)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:684)


错误可能发生在任何使用ArrayMap的任何地方,比如Fragment,Intent,Bundle
Android Framework中大量使用ArrayMap存储,所以你自己不使用也仍有可能遇到此问题

目前已经确认是Android系统本身的一个Bug,相关issue可以参考  
https://issuetracker.google.com/issues/37114373

还有一篇详细分析给出了具体错误原因 和 重现路径  
https://gist.github.com/LanderlYoung/579872afffc62a5f837e654a6f1eab89

不想看详细内容的话也可以简单理解为 ArrayMap 存在线程安全问题

那么理所当然的,不要主动在backgroud thread中使用ArrayMap,确保都在主线程中使用就不会有问题

但是,管得了自己的代码,管不了第三方SDK的代码,在我根本没有使用过ArrayMap的情况下,错误以极低概率发生
每天umeng都能抓到若干crash记录

首先第一步是定位问题  
现在我已经知道了,很可能是线程同步问题,但是由于我自身没有使用ArrayMap,所以很可能是某个第三方库在使用造成的问题

准备一个原生系统(自己手机刷一个原生系统 或者 模拟器),然后在Android Studio中使用对应的版本的系统源码,就可以进行源码断点了
第三方ROM由于代码行数 和 源码不一致,一般是很难断点的


我这里主要是定位问题,所以在 android.util.ArrayMap.put 上进行了条件断点(在断点上点右键 输入 condition)

condition : Log.e("######",Thread.currentThread().toString())==0

中断不会发生,但是会输出所有 调用android.util.ArrayMap.put的线程

结果如下  
![img](/images/arraymap-thread.png)

可以看到除了主线程外  还有另外2个线程在使用ArrayMap

移除condition看了一下调用栈
可以看到其中一个是
run:124, l$1 (com.tencent.beacontsa.core.eve (广点通SDK里带的东西)

另外一个是 thead-pool 是 umeng 在使用

目前初步的解决方案是 把umeng event统计都停了(本来也没在用,实质上数据都记在GA)
广点通BannerAD 延迟 5秒加载





