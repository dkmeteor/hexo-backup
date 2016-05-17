title: Android-App-ABI-setting
date: 2016-05-17 10:18:43
tags: android
---
先放结论:
最优性能且不考虑安装包体积
提供armeabi,armeabi-v7a,arm64-v8a,x86_64

全适配+最小体积
提供armeabi,armeabi-v7a

最小体积+99%适配
提供armeabi-v7a

---

首先官方的ABI说明
ref: https://developer.android.com/ndk/guides/abis.html

理想情况下,App应当同时提供armeabi,armeabi-v7a,arm64-v8a,x86,x86_64,mips,mips64的so文件
一些比较小的类库提供全部的so以保证在全平台能以最优性能运行,是没有问题的

但是有时候会有一些较大的类库,比如ffmpeg,opencv之类,一个so文件就有3~5mb,全部提供会让APK体积过于膨胀,这个时候就需要进行适当的权衡.

---
不考虑最优性能的话:

其中 x86(x86_32) 的真实设备实际上并不存在,只有模拟器可以调成x86_32,实际存在的CPU都是x86_64的
并且,所有x86_64的设备都支持Secondary ABI, 大部分配置都是 armeabi/armeabi-v7a , 不考虑性能最优的话,是可以不提供的

mips略过,印象中只有早期起一些android laptop 是这种架构.

---

这样我们需要实际考虑的ABI只有armeabi,armeabi-v7a,arm64-v8a

考虑最小体积,其实只提供一个armeabi-v7a对于99%的机型就足够了
目前就测试结果看
只有 MX4 / MX4 PRO 的5.1/5.1.1 会无法找到so文件,猜测可能Primary ABI是 arm64-v8a 并且 Secondary ABI是 armeabi
完美的避过了v7a ,当然也有可能是别的原因.目前观察到的只有这款机型有问题

---

之后因为armeabi(实际上是V5)的设备现在市面上几乎已经没有了,考虑兼容,当时尝试使用了v7a+v8a的so,没有提供armeabi版本的so
大约也能覆盖98%的设备
上线后发现OPPO R7(全系列几乎都有),魅蓝Metal,魅蓝Note,魅族 PRO 5出现大量crash,马上做了hotfix.
从数据看,只发生在 Oppo和魅族上,其它机型完全没有. 
我也很郁闷,这几款机型CPU都是 arm64的,反而找不到v8a的so,具体是个什么情况我不是很理解

就最终结果来看,使用v7a就能覆盖99%的设备(除了魅族 MX4 / MX 4 PRO)
为了支持这一设备,再配上个armeabi的so就好了

---
主要观察的数据是应用在umeng后台的错误统计
以下是一些测试数据

OPPO R7 V5       OK

OPPO R7 v7 OK

OPPO R7 V5+v7    OK

OPPO R7 V5+v7+V8 OK

OPPO R7 v7+V8 CRASH

OPPO R7 5.0 V7+V8 OK

魅蓝note2 V5   CRASH

魅蓝note2 v7   OK 

魅蓝note2 v7+v8   OK

魅蓝note2 V5+v7+v8 OK   

乐视 V5   CRASH

乐视 v7  	OK

乐视 v7+v8  	OK

乐视 V5+v7+v8    OK

NEXUS 5X V5 CRASH

NEXUS 5X V7 OK

NEXUS 5X V7+V8 OK 

PRO 5 V7 CRASH

PRO 5 V7+V8 CRASH

PRO 5 V5 OK

PRO 5 V5+V7+V8 OK

GALAXY S6 V7 OK

GALAXY S6 V5+V7 OK


最终方案使用 v5+v7,v8a以兼容模式运行32位库
以下设备测试没问题.

OPPO R7 OK

三星 S6 OK

华为 mate S OK

魅蓝note2 OK

魅族 PRO 5 OK

nexus 5X OK

小米4C OK

乐视手机 OK

nexus 5 OK

