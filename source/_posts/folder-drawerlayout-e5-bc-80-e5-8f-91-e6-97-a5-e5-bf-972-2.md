title: Folder-DrawerLayout 开发日志2
id: 197
categories:
  - Blog
date: 2015-01-20 16:10:30
tags:
---

[![Folder-DrawerLayout](http://dk-exp.com/wp-content/uploads/2015/01/Folder-DrawerLayout3.gif)](http://dk-exp.com/wp-content/uploads/2015/01/Folder-DrawerLayout3.gif)

&nbsp;

暂时告一段落....

周末如果有空估计要砍掉重写一下......

最开始在x轴上做了偏移是个错误....造成了不跟手的感觉......这段逻辑砍掉的话....之前的参数又要重新调整......

&nbsp;

而且又看了下 http://code4app.com/ios/BCMeshTransformView/53bb94c9933bf0fb678b4d4e 和 MorningCall的感觉

它们不需要移出屏幕....所以感觉不大一样.....只要压缩x轴就好了...

我的表现形式其实是个Drawer....x轴上不用压缩或者微量压缩就好了.... 或者压缩完了让它向外侧偏移，而不是内侧...

&nbsp;

这几天有点生病...胃不舒服..........周末要是好了集中赶紧调一下让这个库完结了算了........

再拖一拖那几个公式是干啥的我自己都不认识了...

--------------------------------

或者我应该写一个非Drawer形式的新库？年前不想挖坑了.........