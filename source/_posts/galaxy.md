title: Galaxy
id: 237
categories:
  - android
  - Blog
date: 2015-03-01 07:03:07
tags:
---

[![Screenshot_2015-03-01-14-25-18](http://dk-exp.com/wp-content/uploads/2015/03/Screenshot_2015-03-01-14-25-18-576x1024.png)](http://dk-exp.com/wp-content/uploads/2015/03/Screenshot_2015-03-01-14-25-18.png)

https://github.com/dkmeteor/Galaxy


从https://android.googlesource.com/platform/packages/wallpapers/Galaxy4/ clone出来

然后转成了 gradle 工程

之前就吐槽 RenderScript在 Gradle里集成多蛋疼.....


源码库里那个Galaxy4工程是和源码一起编译的....不受 @hide 标识影响

但是我要build就发现.......在SDK21里 全是@hide 的class


把SDK换成15的又 缺一堆字段.....

试了半天....发现只能用 SDK 17编译
