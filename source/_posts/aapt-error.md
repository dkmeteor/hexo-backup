title: AAPT Error
id: 35
categories:
  - Blog
date: 2014-08-07 15:30:26
tags:
---

一个 Error executing aapt: Return code -1073741819的bug折腾了一晚上

...aapt大大的有问题啊...明明是个很普通的res丢失问题..

错误信息是这样的.......

![log](/images/aapt_error_1.jpg)

原因是`action_settings`不存在 

	<menu xmlns:android="http://schemas.android.com/apk/res/android" >;
		<item
		android:id="@+id/action_settings"
		android:orderInCategory="100"
		android:showAsAction="never"
		android:title="@string/action_settings"/>
	<menu>

换了一个别的工程测试了一下，很正常，错误信息报的也很正常 Resource not found...

![log](/images/aapt_error_2.jpg)


不知道这个com.example.android.supportv7里什么东西在影响aapt生成res文件....

在windos->preferences--android-build里把build输出打开看了一下：

2000多行log...

结尾是这样
> > 
> 
> D:\WORK\adt-bundle-windows-x86_64-20140702\adt-bundle-windows-x86_64-20140702\sdk\extras\android\support\v7\mediarouter\res\layout\mr_media_route_list_item.xml)> 
> [2014-08-07 23:22:03 - AndroidTest] (new resource id mr_media_route_list_item from D:\WORK\adt-bundle-windows-x86_64-20140702\adt-bundle-windows-x86_64-20140702\sdk\extras\android\support\v7\mediarouter\res\layout-v17\mr_media_route_list_item.xml)> 
> [2014-08-07 23:22:03 - AndroidTest] (new resource id overlay_display_window from D:\WORK\workspace4.x\AndroidTest\res\layout\overlay_display_window.xml)> 
> [2014-08-07 23:22:12 - AndroidTest] (new resource id sample_> 
> [2014-08-07 23:22:12 - AndroidTest] 'aapt' error. Pre Compiler Build aborted.


猜测就是解析到menu xml的时候aapt挂了吧...
只能这样定位错误了