title: GoogleMap without Google Play Service
id: 126
categories:
  - Blog
date: 2014-10-13 17:08:57
tags:
---

今天在QQ群里看到有人研究 地图的问题.

目前由于 国内阉割 Google Play Service的问题，大部分解决方案是

国内相关的 用 百度地图

国外相关的 用 Google地图

然后分开处理...........



然后有人提到 携程 的app可以在没有 Google Play Service的情况下调起Google Map

于是专门下载下来测试了一下，确实可以。

但是使用 UI Automator 把View Tree拉下来的看的时候

很明显可以发现这其实是一个 WebView

![uiautomator-xiecheng](/images/uiautomator-xiecheng.jpg)



所以猜测携程 其实通过WebView调用移动版Google地图的方式使用的Google Map.

不过这样就要走js去控制这里面的逻辑...........不过也可能是Server端做好了直接载入进来的...