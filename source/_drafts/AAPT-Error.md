title: AAPT Error
id: 32
categories:
  - Algorithm
tags:
---

一个 Error executing aapt: Return code -1073741819的bug折腾了一晚上...aapt大大的有问题啊...明明是个很普通的res丢失问题..

&nbsp;

错误信息是这样的.......

[![QQ图片20140807231535](http://dk-exp.com/wp-content/uploads/2014/08/QQ图片20140807231535-300x43.jpg)](http://dk-exp.com/wp-content/uploads/2014/08/QQ图片20140807231535.jpg)

&nbsp;

&nbsp;

原因是**action_settings**不存在

&lt;menu xmlns:android="http://schemas.android.com/apk/res/android" &gt;

&lt;item
android:id="@+id/action_settings"
android:orderInCategory="100"
android:showAsAction="never"
android:title="**@string/action_settings**"/&gt;

&lt;/menu&gt;

&nbsp;

换了一个别的工程测试了一下，很正常，错误信息报的也很正常 Resource not found...

[![QQ图片20140807231931](http://dk-exp.com/wp-content/uploads/2014/08/QQ图片20140807231931-300x135.jpg)](http://dk-exp.com/wp-content/uploads/2014/08/QQ图片20140807231931.jpg)

&nbsp;

不知道这个com.example.android.supportv7里什么东西在影响aapt生成res文件....

&nbsp;

在windos-&gt;preferences--android-build里把