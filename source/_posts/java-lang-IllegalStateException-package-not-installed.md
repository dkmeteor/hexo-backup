title: 'java.lang.IllegalStateException: package not installed'
date: 2016-05-31 12:05:31
tags: android
---
#java.lang.IllegalStateException: package not installed

用了Android Studio的时候偶尔会遇到这个问题.

ref: http://stackoverflow.com/questions/24426635/caused-by-java-lang-illegalstateexception-package-not-installed
ref: http://stackoverflow.com/questions/35675855/android-studio-floatingactionbutton-error

解决方案很简单,gradle重新sync一下就好了.

重现步骤大概是这样,gradle sync/build中,修改lib/compile配置,或者同时在命令行clean build,会导致build过程中某些该编译进去的jar/module丢失,于是运行时会报各种奇怪的Bug.大部分那都是 package not installed / class not found 一类的

这种时候修改一下gradle文件重新sync 就能修复此问题.本质上是IDE/gradle的bug,不知道原因的话解决起来有点蛋疼.