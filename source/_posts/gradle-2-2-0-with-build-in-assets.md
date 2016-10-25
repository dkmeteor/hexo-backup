title: gradle 2.2.0 with build in assets(baidu ad sdk)
date: 2016-10-24 17:45:23
tags: android
---
前几天给一个老项目发版时,升级gradle到2.2.2,发现baidu广告SDK一直无法正确加载.
提示 load __xadsdk__remote__final__.jar failed

切换回 `com.android.tools.build:gradle:2.2.0-alpha4` 后恢复正常.

但是使用老版本gradle编译会一直失败并提示

``` java
	Error:(1, 0) Plugin is too old, please update to a more recent version, or set ANDROID_DAILY_OVERRIDE environment variable to "245e4d403bf33d23690fe32bf814235bf6949f57"
	<a href="fixGradleElements">Fix plugin version and sync project</a><br><a href="openFile:/Users/dk/Documents/git/***(****/******/build.gradle">Open File</a>
```

虽然设置ANDROID_DAILY_OVERRIDE后暂时可用,但是下次编译时又要重新处理.

稍微看了一下,发现原因是这样
百度广告SDK中有一个build in jar
目录结构大概是这样

	Baidu_MobAds_SDK.jar
		- assets
			- `__xadsdk__remote__final__.jar`
		- com.baidu........
			- 若干class
		- META-INF

在使用gradle2.2.0-alpha4时,jar包中的assets会自动merge到apk的assets中,但是升级到2.2.0~2.2.2时,编译出的apk中不再包含`__xadsdk__remote__final__.jar`

暂时没有看到任何文档或者log中表述 gradle 2.2.0-alpha4 -> gradle 2.2.0 修改了assets的处理方式

目前我的处理方式是,解包Baidu_MobAds_SDK.jar后将 `__xadsdk__remote__final__.jar` 直接拷贝到工程assets目录下,即可使用最新的gradle正常编译.
当然你也可以回退到alpha4/rc1

顺便,这事有1个月了,也不见百度的集成文档更新一下.
