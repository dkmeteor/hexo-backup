title: manifest xml bug with android.intent.action.MAIN
id: 131
categories:
  - android
date: 2014-11-10 17:07:07
tags:
---

# manifest xml bug

使用如下配置

    <intent-filter>
        <action android:name=“android.intent.action.VIEW” />
        <action android:name=“android.intent.action.MAIN” />
        <category android:name=“android.intent.category.LAUNCHER” />
    </intent-filter>

在Eclipse上Run, 可以正常安装,并启动应用 若使用下面这个配置(只是调换了VIEW和MAIN的顺序)
    
    <intent-filter>
        <action android:name=“android.intent.action.MAIN” />
        <action android:name=“android.intent.action.VIEW” />
        <category android:name=“android.intent.category.LAUNCHER” />
    </intent-filter>

在Eclipse上Run后,提示

“No Launcher activity found!

The launch will only sync the application package on the device!”

虽然也能安装成功,但是程序不会自动启动

——————————————————————————-

目测是Eclipse自身的bug, 蛋疼.