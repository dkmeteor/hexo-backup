title: 播放网络音频时,MediaPlayer在Prepare阶段访问任何API都会造成主线程阻塞
date: 2016-05-27 17:48:00
tags: android
---
#播放网络音频时,MediaPlayer在Prepare阶段访问任何API都会造成主线程阻塞

播放本地音频时,prepare阶段很短,基本不会发生这种情况,但是播放网络音频时,无论是使用prepare() 还是 prepareAsync()
MediaPlayer都会进入lock状态,此时调用MediaPlayer的任何API,比如getDuration()时,都会直接阻塞调用的线程,直到Prepare完成或失败.

这个问题比较蛋疼,由于当时需求是需要在前台播放一段网络音频,不需要后台播放,所以MediaPlayer对象直接由前台持有,切换音频时,不销毁MediaPlayer,而是reset()以后重新Prepare().

但是发现用户快速切换音频时,有时会发生anr,抓了一些log以后发现,anr竟然是在stop()或reset()上发生的.

尝试过新建Thread去prepare,也试过prepareAsync(),都不解决问题.
