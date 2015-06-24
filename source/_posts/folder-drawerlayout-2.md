title: Folder-DrawerLayout 开发日志1
id: 162
categories:
  - Blog
date: 2015-01-07 17:56:13
tags:
---

&nbsp;

![folder-drawer-layout](/images/Folder-DrawerLayout-develop-1.gif)

动态效果加进去了.....

效果意外的恶心.....y轴的偏移看起来要换个 函数 重新构筑一下...

而且如果矩阵是预设的.....对应的偏移节点方程 其实可以算好了。。。再加个数组直接存在里面吧...

最后再用复杂界面测吧....估计效率是坨shit.....毕竟是 5*50矩阵....