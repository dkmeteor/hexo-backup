title: Eclipse  Unhandled event loop exception
id: 54
categories:
  - Blog
date: 2014-08-10 15:11:53
tags:
---

<span style="color: #333333;">Eclipse使用一段时间后， 突然弹出这个</span>

<span style="color: #333333;">![error1](http://dk-exp.com/wp-content/uploads/2014/08/error1-300x213.jpg)</span>
 <span style="color: #333333;">点击OK后弹出下面这个</span>
 ![error2](http://dk-exp.com/wp-content/uploads/2014/08/error2-300x144.jpg)
<span style="color: #333333;">再点击 No....过一会上面按个又会弹出...</span>

<span style="color: #333333;">家里的电脑配的环境，本来就用的少，不影响正常工作，就是很烦.....</span>
<span style="color: #333333;">-------------------</span>
<span style="color: #333333;">因为这个问题特意重新下载了最新的 adt-bundle-windows-x86_64-20140702 </span>
<span style="color: #333333;">问题依然存在....同时，我公司的电脑用的同样版本的Eclipse就没有这个问题...</span>

<span style="color: #333333;">百度了下...有说和XX杀毒冲突的，有让改AVD配置的,有让改</span><span style="font-weight: bold; color: #333333;">eclipse.exe名字的。。。。</span><span style="color: #333333;">试验了下没什么用.....</span>

<span style="color: #333333;">Any other solution?------------------</span>
<span style="color: #333333; font-size: small;"><span style="color: #000000;">2014/08/10</span></span>

把整个 .metadata 删掉了  好了....当然 整个workspace里的工程都没了  全部要重新导入...<span style="color: #333333; font-size: small;"><span style="color: #000000;">---------------------</span></span>
<span style="color: #333333; font-size: xx-large;"><span style="color: #ff0000;">2014/08/10</span></span>
<span style="color: #333333; font-size: xx-large;"><span style="color: #ff0000;">又出现了.......百度到一个帖子说是和<span style="font-family: Arial;">hydradm.exe hydradm64.exe 冲突....</span></span></span>
<span style="color: #333333; font-family: Arial;"><span style="font-size: xx-large;"><span style="color: #ff0000;">这2个是 AMD显卡驱动带的催化剂...我把这2个进程杀掉看看</span></span></span>
<span style="color: #333333; font-family: Arial;"><span style="font-size: xx-large;"><span style="color: #ff0000;">
</span></span></span>
<span style="color: #333333; font-family: Arial;"><span style="font-size: xx-large;"><span style="color: #ff0000;">杀掉以后目前还没出现过...</span></span></span>