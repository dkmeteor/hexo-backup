title: Gradle编译RenderScript
id: 233
categories:
  - android
  - Blog
date: 2015-02-27 15:49:21
tags:
---

Gradle编译RenderScript

有点操蛋，现在官方文档上还是eclipse上的集成方式

https://android.googlesource.com/platform/frameworks/rs/

demo更不知道是哪年的版本

早先还有各种指定renderscript.srcDirs = ['src'] 的写法

随着Gradle和plugin的更新...这破写法一天一变....网上能搜到一打的不一样的写法...没几个能用的...

* * *

比较正确的请参考这个

http://stackoverflow.com/questions/19658145/how-to-use-the-renderscript-support-library-with-gradle/22976675#22976675
以我的项目为例
目前是这样的

apply plugin: 'com.android.application'

android {
compileSdkVersion 21
buildToolsVersion "21.1.2"

defaultConfig {
applicationId "com.dk.heartbeats"
minSdkVersion 17
targetSdkVersion 21
versionCode 1
versionName "1.0"
renderscriptTargetApi 19
renderscriptSupportModeEnabled false
}

buildTypes {
release {
minifyEnabled false
proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
}
}

}

dependencies {
compile fileTree(dir: 'libs', include: ['*.jar'])
compile 'com.android.support:appcompat-v7:21.0.3'
compile 'com.jakewharton:butterknife:6.0.0'
compile files('libs/jtransforms-2.4.jar')
compile files('libs/renderscript-v8.jar')
}

注意这个
`renderscriptSupportModeEnabled true`
和
`compile files('libs/renderscript-v8.jar')`

原则上 当你设置了 `renderscriptSupportModeEnabled true`
gradle应当自动将 v8挂载到项目中，不需要自己去把renderscript-v8.jar拷过来

但是我设置了以后 sync gradle / clean / build 以后依然提示我v8 not found...无奈之下自己把v8 jar拷到lib目录下编译

顺便 这个 renderscript-v8.jar在 `SDK\build-tools\21.1.2\renderscript\lib` 下

没想到的是...这个时候这破玩意好了又告诉我multiply dex......

没办法 我又把上面的关了...

反正这2个使用任意一个应该就可以了....

* * *

renderscriptTargetApi 19

这个 必须要

* * *

你们如果看过以前的工程的话...会发现.rs文件都是直接仍在包名下
比如官方Demo里的 yuv.rs就直接扔在 com.android.rs.livepreview 下面

同时，早期Gradle版本可以通过

sourceSets {
main {
manifest.srcFile 'AndroidManifest.xml'
java.srcDirs = ['src']
resources.srcDirs = ['src']
aidl.srcDirs = ['src']
renderscript.srcDirs = ['src']
res.srcDirs = ['res']
assets.srcDirs = ['assets']
}

instrumentTest.setRoot('tests')
}

指定编译路径
恩
注意

renderscript.srcDirs = ['src']

遗憾的是我实验了一下自己指定renderscript路径毫无反应。
查找了一些资料以后发现需要直接扔在
rs文件夹下...

项目结构像这样

<pre class="lang:default decode:true ">--app
--builds
--libs
--src
    --android test
    --main
        --java
        --res
        --rs
        AndroidManifest.xml</pre>

&nbsp;

意会一下...就是默认gradle项目格式...放在和java同层的rs文件夹里才会生成对应的`ScriptC_yuv.java`文件

这个文件在
`{project folder}\app\build\generated\source\rs\debug\{package name}\` 下

* * *

官方不打算出一个正式的 Gradle 集成 RenderScript的Guide并保持更新么？
这集成方式一版本一变上下不兼容简直醉了