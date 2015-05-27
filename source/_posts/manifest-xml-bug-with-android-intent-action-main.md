title: manifest xml bug with android.intent.action.MAIN
id: 131
categories:
  - android
date: 2014-11-10 17:07:07
tags:
---

<div>

# manifest xml bug

使用如下配置

    <span style="color: #000080;">&lt;intent-filter&gt;</span>
        <span style="color: #000080;">&lt;action <span style="color: #008080;">android:name</span>=<span style="color: #dd1144;">"android.intent.action.VIEW"</span> /&gt;</span>
        <span style="color: #000080;">&lt;action <span style="color: #008080;">android:name</span>=<span style="color: #dd1144;">"android.intent.action.MAIN"</span> /&gt;</span>
        <span style="color: #000080;">&lt;category <span style="color: #008080;">android:name</span>=<span style="color: #dd1144;">"android.intent.category.LAUNCHER"</span> /&gt;</span><span style="color: #000080;">&lt;/intent-filter&gt;</span>`</pre>

    在Eclipse上Run, 可以正常安装,并启动应用

    若使用下面这个配置(只是调换了VIEW和MAIN的顺序)

    <pre>`<span style="color: #000080;">&lt;intent-filter&gt;</span>
        <span style="color: #000080;">&lt;action <span style="color: #008080;">android:name</span>=<span style="color: #dd1144;">"android.intent.action.MAIN"</span> /&gt;</span>
        <span style="color: #000080;">&lt;action <span style="color: #008080;">android:name</span>=<span style="color: #dd1144;">"android.intent.action.VIEW"</span> /&gt;</span>
        <span style="color: #000080;">&lt;category <span style="color: #008080;">android:name</span>=<span style="color: #dd1144;">"android.intent.category.LAUNCHER"</span> /&gt;</span><span style="color: #000080;">&lt;/intent-filter&gt;</span>

在Eclipse上Run后,提示

"No Launcher activity found!

The launch will only sync the application package on the device!"

虽然也能安装成功,但是程序不会自动启动

-----------------------------------------------------

目测是Eclipse自身的bug, 蛋疼.

</div>