title: Android 7.0 new APK Signature Scheme v2 problem with multiply channel packer
date: 2016-08-04 14:49:22
tags: android
---
Android 7.0 中新增了 `APK Signature Scheme v2` 签名方式

如果Android Studio升级到 v2.2+,构建APK时默认使用的签名方式就是`APK Signature Scheme v2`

目前比较流行的2套 多渠道打包脚本
- 在APK内注入${channel}.txt 文件
- 在APK的zip info中写入 channel 信息

实质上都会在签名后修改APK文件,目前都会造成 签名认证失败
失败信息如下:

``` shell
	Yuki-Android git:(feature/small-fix) adb install /Users/dk/Documents/yuki-release/v3.1.8/Yuki-base-Direct.apk
	Failed to install /Users/dk/Documents/yuki-release/v3.1.8/Yuki-base-Direct.apk: Failure [INSTALL_PARSE_FAILED_NO_CERTIFICATES: Failed to collect certificates from /data/app/vmdl453897674.tmp/base.apk: META-INF/CERT.SF indicates /data/app/vmdl453897674.tmp/base.apk is signed using APK Signature Scheme v2, but no such signature was found. Signature stripped?]
```

在非7.0设备上可以正常安装,在7.0设备上则无法通过签名认证.
现在可以通过在build.gradle中手动关闭APK Signature Scheme v2来避免此问题

``` java
 signingConfigs {
        release {
            storeFile file("xxxx")
            storePassword "xxxx"
            keyAlias "xxxx"
            keyPassword "xxxx"
            v2SigningEnabled false
        }
        debug {
            storeFile file("xxxx")
            storePassword "xxxx"
            keyAlias "xxxx"
            keyPassword "xxxx"
            v2SigningEnabled false
        }
    }
```

目前暂时这样,考虑到以后v2签名可能会变成一个强制配置,可能需要等`APK Signature Scheme v2`具体文档和认证方式公布后,更新一下现在的多渠道打包脚本
