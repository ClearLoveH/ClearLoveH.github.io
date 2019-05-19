---
layout:     post
title:      "Glide 源码解析"
date:       2019-5-19 11:11
author:     "Heng"
header-img: "img/艾欧尼亚.jpg"
catalog: true
tags:
    - Android
---

### Picasso

#### 关于Picasso
picasso是Square公司开源的一个Android图形缓存库，不仅实现了图片异步加载的功能，还解决了android中加载图片时需要解决的一些常见问题：
- 在adapter中需要取消已经不在视野范围的ImageView图片资源的加载，否则会导致图片错位，Picasso已经解决了这个问题；
- 使用复杂的图片压缩转换来尽可能的减少内存消耗；
- 自带内存和硬盘二级缓存功能。

`Picasso 加载图片流程图`
![picasso](/img/in-post/post-Android/Glide/picasso.png)

Picasso的基本用法

        Picasso.with(this).load(“http://nuuneoi.com/uploads/source/playstore/cover.jpg“).into(ivImgPicasso);

        Picasso.with(this).load(“http://nuuneoi.com/uploads/source/playstore/cover.jpg“).resize(768, 432).into(ivImgPicasso);

        Picasso.with(this).load(“http://nuuneoi.com/uploads/source/playstore/cover.jpg“).fit().centerCrop().into(ivImgPicasso);

- 第一种：加载了全尺寸的图片到内存，然后让GPU来实时重绘大小
- 第二种：你需要主动计算ImageView的大小，或者说你的ImageView大小是具体的值（而不是wrap_content），
- 第三种：按统一比例缩放图片(保存图片的尺寸比例)便于图片的二维(宽度和高度)等于或者大于相应的视图的维度，这种方法和Glide加载图片占用的内存几乎是相同的，虽然内存开销差距不大，但是在这个问题上Glide完胜Picasso。因为Glide可以自动计算出任意情况下的ImageView的大小。





---
### Glide与Picasso的区别：

1. 内存：
- 加载同一张图片Picasso，Picasso的内存开销仍然远大于Glide。
2. Image质量的细节：
    - Glide默认的是Bitmap格式是RGB-565
    - Picasso默认ARGB_8888格式
    - Glide加载的图片没有Picasso那么平滑，但是很难察觉
3. 磁盘缓存：      
    - Picasso缓存的是全尺寸的。而Glide缓存的跟ImageView尺寸相同
    - 将ImageView调整成不同大小不管大小如何设置。Picasso只缓存一个全尺寸的。Glide则不同，它会为每种大小的ImageView缓存一次
    - 让Glide既缓存全尺寸又缓存其他尺寸的方法：

            Glide.with(this).load(“http://nuuneoi.com/uploads/source/playstore/cover.jpg“).diskCacheStrategy(DiskCacheStrategy.ALL).into(ivImgGlide);
    - Glide的这种方式优点是加载显示非常快。而Picasso的方式则因为需要在显示之前重新调整大小而导致一些延迟，Glide比Picasso快，虽然需要更大的空间来缓存。

4. Gif动图
    - Glide可以加载Gif动图，Picasso不可以加载动图
    - Glide动画会消耗太多的内存，因此使用时谨慎使用

总结一下他们之间的区别：
1. Glide比Picasso加载速度快，但Glide比Picasso需要更大的空间来缓存；
2. Glide加载图像及磁盘缓存的方式都优于Picasso，且Glide更有利于减少OutOfMemoryError的发生；
3. Glide可以加载Gif动图，Picasso不可以加载动图
4. Picasso加载的图片比Glide加载的图片平滑(可忽略不计)

`Picasso所能实现的功能，Glide都能做，只是所需的设置不同。但是Picasso体积比起Glide小太多如果项目中网络请求本身用的就是okhttp或者retrofit(本质还是okhttp)，那么建议用Picasso，体积会小很多。Glide的好处是大型的图片流，比如gif、Video，如果做美拍这种视频类应用，建议使用。`

---
### Glide新版本4.0的特性