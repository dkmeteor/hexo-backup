title: Android Studio Gradle构建速度优化
id: 247
categories:
  - android
  - Blog
date: 2015-03-16 16:48:01
tags:
---


1.

[https://www.timroes.de/2013/09/12/speed-up-gradle/](https://www.timroes.de/2013/09/12/speed-up-gradle/)

create a file named **gradle.properties** in the following directory:


*   /home/&lt;username&gt;/.gradle/ (_Linux_)
*   /Users/&lt;username&gt;/.gradle/ (_Mac_)
*   C:\Users\&lt;username&gt;\.gradle (_Windows_)
Add this line to the file:


    org.gradle.daemon=true


From now on Gradle will use a daemon to build, whether you are using Gradle from command line or building in Android Studio. You could also place the **gradle.properties** file to the root directory of your project and commit it to your SCM system. But you would have to do this, for every project (if you want to use the daemon in every project).


关于这个写法有几个变种，都尝试了一下，并没有用。


参考Android Studio配置选项

![speed-up-gradle](/images/gradle-build-speed-up-android-studio-setting.png)

Android Studio 1.1.0+版本中该配置已经默认打开。


* * *


2.
用命令行Build

可以参考这个讨论串
[https://plus.google.com/u/0/+RicardoAmaral/posts/e9PG6vSN5w3](https://plus.google.com/u/0/+RicardoAmaral/posts/e9PG6vSN5w3)

gradle assembleDebug
实测速度快50%左右 原理不明
回头写个脚本 build完成再自动安装运行应该就好了

更新下

gradle installDebug 就可以了，不用自己在gradle里搞task了，省事.
明天测试下性能



* * *


3.
Android模块化编程之引用本地的aar
[http://stormzhang.com/android/2015/03/01/android-reference-local-aar/](http://stormzhang.com/android/2015/03/01/android-reference-local-aar/)

目前把项目中引用的Library都打成了本地aar，减少编译的文件，编译速度显著加快





* * *


4.
我觉得TMD还是把公司的破电脑扔了换个新的才能从根本上解决问题