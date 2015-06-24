title: Folder-DrawerLayout 开发日志0
id: 153
categories:
  - Blog
date: 2014-12-28 18:42:40
tags:
---


![folder-drawer-layout](/images/Folder-DrawerLayout-develop-0.gif)
![morning-routine](/images/moring-routine.png)



又仔细看了下MorningRoutine....发现还有个渐变阴影...

目测应该是在挂verts之前用shader先渲染在原图上的

x轴看起来也不是均匀的....首

大概先是要把波长写成x相关的函数多代换一层.....

然后这个挤压效果还要把x轴总长按y轴位置等比压缩一下....

原效果verts矩阵是 5*15的...棱角还比较大.....不知道加大verts点数对性能有多大影响...


记录一下,元旦再弄了...有点蛋疼......一次要把这么多玩意载入脑子里面....一旦被interrupt就完了...看来是只适合半夜写了...