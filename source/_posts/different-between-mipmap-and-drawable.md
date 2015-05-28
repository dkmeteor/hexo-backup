title: mipmap 目录和drawable 目录有什么区别
id: 256
categories:
  - android
  - Blog
date: 2015-03-17 15:48:33
tags:
---

我简单总结一下：

使用上没有任何区别,你把它当drawable用就好了。

但是用mipmap系统会在缩放上提供一定的性能优化。
#官方介绍：

> Mipmapping for drawables
> 
>   Using a mipmap as the source for your bitmap or drawable is a simple way to provide a quality image and various image scales, which can be particularly useful if you expect your image to be scaled during an animation.
> 
>   Android 4.2 (API level 17) added support for mipmaps in the Bitmap class—Android swaps the mip images in your Bitmap when you've supplied a mipmap source and have enabled setHasMipMap(). Now in Android 4.3, you can enable mipmaps for a BitmapDrawable object as well, by providing a mipmap asset and setting the android:mipMap attribute in a bitmap resource file or by calling hasMipMap().

&nbsp;

* * *

#应用场景：

> If you know that you are going to draw this bitmap at less than 50% of its original size, you may be able to obtain a higher quality by turning this property on. Note that if the renderer respects this hint it might have to allocate extra memory to hold the mipmap levels for this bitmap.

&nbsp;

* * *

#一个应用实例：

> Nexus 6
>   Screen
>   The Nexus 6 boasts an impressive 5.96” Quad HD screen display at a resolution of 2560 x 1440 (493 ppi). This translates to ~ 730 x 410 dp (density independent pixels).
> 
>   Check your assets
>   It has a quantized density of 560 dpi, which falls in between the xxhdpi and xxxhdpi primary density buckets. **For the Nexus 6, the platform will scale down xxxhdpi assets**, but if those aren’t available, then it will scale up xxhdpi assets.
> 
>   Provide at least an xxxhdpi app icon because devices can display large app icons on the launcher. It’s best practice to place your app icons in mipmap- folders (not the drawable- folders) because they are used at resolutions different from the device’s current density. For example, an xxxhdpi app icon can be used on the launcher for an xxhdpi device.
> 
>   res/
>   mipmap-mdpi/
>   ic_launcher.png
>   mipmap-hdpi/
>   ic_launcher.png
>   mipmap-xhdpi/
>   ic_launcher.png
>   mipmap-xxhdpi/
>   ic_launcher.png
>   mipmap-xxxhdpi/
>   ic_launcher.png # App icon used on Nexus 6 device launcher
>   Choosing to add xxxhdpi versions for the rest of your assets will provide a sharper visual experience on the Nexus 6, but does increase apk size, so you should make an appropriate decision for your app.
> 
>   res/
>   drawable-mdpi/
>   ic_sunny.png
>   drawable-hdpi/
>   ic_sunny.png
>   drawable-xhdpi/
>   ic_sunny.png
>   drawable-xxhdpi/ # Fall back to these if xxxhdpi versions aren’t available
>   ic_sunny.png
>   drawable-xxxhdpi/ # Higher resolution assets for Nexus 6
>   ic_sunny.png

#总结
这个实例总结一下是这样：
Nexus 6 有 493 ppi
它刚好在 xxhdpi和xxxhdpi之间
所以显示的时候需要对xxxhdpi的资源进行缩小
如果你用了mipmap-xxxhdpi,那么这里会对sclae有一个优化，性能更好，占用内存更少。

所以现在官方推荐使用mipmap

> It’s best practice to place your app icons in mipmap- folders (not the drawable- folders) because they are used at resolutions different from the device’s current density.

&nbsp;