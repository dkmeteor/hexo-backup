title: 播放网络音频时,MediaPlayer在Prepare阶段访问任何API都会造成主线程阻塞
date: 2016-05-27 17:48:00
tags: android
---
#播放网络音频时,MediaPlayer在Prepare阶段访问任何API都会造成线程阻塞

播放本地音频时,prepare阶段很短,基本不会发生这种情况,但是播放网络音频时,无论是使用prepare() 还是 prepareAsync()
MediaPlayer都会进入lock状态,此时调用MediaPlayer的任何API,比如stop/release/reset时,都会直接阻塞调用的线程,直到Prepare完成或失败.应用会抛出ANR. 

当时文件服务使用的是七牛,据观察,七牛CDN在我这里似乎有点问题,请求成功率大概只有95%左右,在糟糕的网络环境下,这个问题更容易出现

这个问题比较蛋疼,由于当时需求是需要在前台播放一段网络音频,不需要后台播放,所以MediaPlayer对象直接由前台持有,切换音频时,不销毁MediaPlayer,而是stop() & reset()以后重新Prepare().

但是发现用户快速切换音频时,有时会发生anr,抓了一些anr log以后发现,anr竟然是在stop()release()上发生的.

尝试过新建Thread去prepare,也试过prepareAsync(),都不解决问题,阻塞是在调用stop/reset时发生的.

相关的Google issue : [https://code.google.com/p/android/issues/detail?id=959](https://code.google.com/p/android/issues/detail?id=959)


最后的我的解决方案是,将MediaPlayer做一下包装,prepare完成前不允许访问其任何方法(比如 stop getDuration release reset ...),需要访问的时候,比如退出界面的时候Stop/release 单独开一个Thread来call这些方法,这样也只会阻塞一个异步线程.

这样配置以后基本解决了该问题