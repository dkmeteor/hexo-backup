title: Eclipse  Unhandled event loop exception
id: 54
categories:
  - Blog
date: 2014-08-10 15:11:53
tags:
---
Eclipse使用一段时间后， 突然弹出这个

![error1](/images/eclipse-conflict-error1.jpg)
点击OK后弹出下面这个

![error2](/images/eclipse-conflict-error2.jpg)
再点击 No....过一会上面按个又会弹出...

家里的电脑配的环境，本来就用的少，不影响正常工作，就是很烦.....
-------------------
因为这个问题特意重新下载了最新的 adt-bundle-windows-x86_64-20140702
问题依然存在....同时，我公司的电脑用的同样版本的Eclipse就没有这个问题...

百度了下...有说和XX杀毒冲突的，有让改AVD配置的,有让改eclipse.exe名字的。。。。试验了下没什么用.....

Any other solution?------------------

2014/08/10

把整个 .metadata 删掉了..好了....当然 整个workspace里的工程都没了..全部要重新导入...
---

2014/08/10

又出现了.......百度到一个帖子说是和`hydradm.exe hydradm64.exe` 冲突....
这2个是 AMD显卡驱动带的催化剂...把这2个进程杀掉看看


杀掉以后目前还没出现过...