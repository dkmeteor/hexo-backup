title: Android 6.0 刷机各种坑记录(防砖)
date: 2015-10-15 13:55:31
tags:
---
从preview3刷成了正式版6.0

几个坑有点蛋疼,记录下,免得下次刷成砖.

- 下载下来的hammerhead-mra58k-factory-52364034.tar文件,解压缩后会获得一个无后缀的文件,修改后缀为zip后,再解压一次,才能看到flash脚本

- 直接执行flash-all. 刷preview3的时候,是直接flash-all就好了,这次刷正式版的时候,执行flash脚本以后一直提示system.img not found,检查了一下压缩包,system.img在里面,压缩包也没损坏,反复试了几次都是这个错误
不能直接一次全刷上,好在可以把image-hammerhead-mra58k.zip压缩包解压了,然后手动一个一个刷img


    fastboot flash bootloader bootloader-hammerhead-hhz12k.img
    fastboot flash radio radio-hammerhead-m8974a-2.0.50.2.27.img
    
    fastboot reboot-bootloader
    
    fastboot flash recovery recovery.img
    fastboot flash boot boot.img
    fastboot flash system system.img

- 刷完以后重启系统,正常开机,开始 应用优化 ,速度很慢.不过刷机以后能保留数据和应用不用去重装也是不错.然而这是个错误.进入系统以后发现几乎所有应用闪退.用root explorer的时候发现emulated文件夹下什么都看不见,才意识到可能是新的权限系统的问题.在设置-应用里翻了一下,果然所有非系统应用权限都是无.尝试修改应用权限失败,修改后重启没有变化.然后因为各种全家桶应用在不停的重启-崩溃-重启,整个手机非常卡.
- 老老实实重启fastboot 然后


    fastboot flash cache cache.img
    fastboot flash userdata userdata.img

- 重启系统后,正常开机,依然很慢,然后就是开机引导和蛋疼的联网验证.这里有个很蛋疼的问题,这一步其实是要绑定google账户,因为那啥啥的原因,你肯定是联不上的,但是你手机插着sim卡,就会自动开启3G,当然你还是联不上,然后这一步也无法跳过.虽然有个跳过按钮,但是放了半个小时还是灰的.
- 正确的方法是,连接wifi(必须),高级里面proxy放一个https代理.这里应该是必须要https,反正我用http代理没成功过,不过免费代理本来就不稳定,也不确定.
- 然后等上一会,就发现可以跳过了,或者直接绑上也可以,我刷preview的时候是直接绑好了,这次可以跳过我就跳过了.

- 另外个方案是 不联网,把sim卡拔出来,应该也是能跳过这个验证步骤的,不过我取卡针不知道扔哪去了,卡拿不出来..

- 目前看起来还没发现什么问题,和preview3看起来也没什么区别.