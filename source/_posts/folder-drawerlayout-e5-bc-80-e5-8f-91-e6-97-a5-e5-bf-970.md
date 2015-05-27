title: Folder-DrawerLayout 开发日志0
id: 153
categories:
  - Blog
date: 2014-12-28 18:42:40
tags:
---

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

[![Screenshot_2014-12-29-02-23-26](http://dk-exp.com/wp-content/uploads/2014/12/Screenshot_2014-12-29-02-23-26-168x300.png)](http://dk-exp.com/wp-content/uploads/2014/12/Screenshot_2014-12-29-02-23-26.png)

[![Screenshot_2014-12-29-02-25-20](http://dk-exp.com/wp-content/uploads/2014/12/Screenshot_2014-12-29-02-25-20-168x300.png)](http://dk-exp.com/wp-content/uploads/2014/12/Screenshot_2014-12-29-02-25-20.png)

&nbsp;

又仔细看了下MorningCall....发现还有个渐变阴影...应该是在挂verts之前用shader先渲染在原图上就好了

x轴也不是均匀的....首先是要把波长写成x相关的函数多代换一层.....然后这个挤压效果还要把x轴总长按y轴位置等比压缩一下....原效果是 5*15的...这棱角这么大.....不知道加大verts点数对性能有多大影响...

&nbsp;

记录一下,元旦再弄了...有点蛋疼......一次要把这么多玩意载入脑子里面....一旦被<span style="color: #333333;">interrupt就完了...看来是只适合半夜写了...</span>