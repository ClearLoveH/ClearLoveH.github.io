---
layout:     post
title:      "OKHttp 源码解析"
date:       2019-5-19 19:19
author:     "Heng"
header-img: "img/恕瑞玛3.jpg"
catalog: true
tags:
    - Android
---

### 关于HTTP

![](/img/in-post/post-Android/OKHttp/HTTP1.png)

![](/img/in-post/post-Android/OKHttp/HTTP2.png)

---
### OKHttp初识
- 在早期的版本中，OkHttp支持Http1.0,1.1,SPDY协议，但是Http2协议的问世，导致OkHttp也做出了改变，OkHttp鼓励开发者使用HTTP2，不再对SPDY协议给予支持。另外，新版本的OkHttp还有一个新的亮点就是支持WebScoket，这样我们就可以非常方便的建立长连接了。
- 作为一个优秀的网络框架，OkHttp同样支持`网络缓存`，OkHttp的缓存基于`DiskLruCache`,对这个类不熟悉的可以[这里学习](http://blog.csdn.net/guolin_blog/article/details/28863651)。DiskLruCache虽然没有被收入到Android的源码中，但也是谷歌推荐的一个优秀的缓存框架。有时间可以自己学习源码，这里不再叙述。
- 在安全方面，OkHttp目前支持了如上图所示的`TLS版本`，以确保一个安全的Socket连接。
- 重试及重定向就不再说了，都知道什么意思，左上角给出了各浏览器或Http版本支持的重试或重定向次数。

#### DiskLruCache

参考博客：https://blog.csdn.net/guolin_blog/article/details/28863651 

由于DiskLruCache并不是由Google官方编写的，所以这个类并没有被包含在Android API当中，我们需要将这个类从网上下载下来，然后手动添加到项目当中。DiskLruCache的源码在Google Source上，地址如下：android.googlesource.com/platform/libcore/+/jb-mr2-release/luni/src/main/java/libcore/io/DiskLruCache.java

- 网易新闻中的数据都是从网络上获取的，包括了很多的新闻内容和新闻图片，如下图所示：

    ![](/img/in-post/post-Android/OKHttp/DiskLruCache1.png)
- 这些内容和图片在从网络上获取到之后都会存入到本地缓存中，因此即使手机在没有网络的情况下依然能够加载出以前浏览过的新闻。而使用的缓存技术不用多说，自然是DiskLruCache了，那么首先第一个问题，这些数据都被缓存在了手机的什么位置呢？
- 其实DiskLruCache并没有限制数据的缓存位置，可以自由地进行设定，但是通常情况下多数应用程序都会将缓存的位置选择为 `/sdcard/Android/data/<application package>/cache `这个路径。选择在这个位置有两点好处：
    - 第一，这是存储在SD卡上的，因此即使缓存再多的数据也不会对手机的内置存储空间有任何影响，只要SD卡空间足够就行。
    - 第二，这个路径被Android系统认定为应用程序的缓存路径，当程序被卸载的时候，这里的数据也会一起被清除掉，这样就不会出现删除程序之后手机上还有很多残留数据的问题。
- 它的客户端的包名是com.netease.newsreader.activity，因此数据缓存地址就应该是 /sdcard/Android/data/com.netease.newsreader.activity/cache ，我们进入到这个目录中看一下，结果如下图所示：
    
    ![](/img/in-post/post-Android/OKHttp/DiskLruCache2.png)
- 可以看到有很多个文件夹，因为网易新闻对多种类型的数据都进行了缓存，这里简单起见我们只分析图片缓存就好，所以进入到bitmap文件夹当中。然后你将会看到一堆文件名很长的文件，这些文件命名没有任何规则，完全看不懂是什么意思，但如果你一直向下滚动，将会看到一个名为journal的文件，如下图所示：
    
    ![](/img/in-post/post-Android/OKHttp/DiskLruCache3.png)
- 上面那些文件名很长的文件就是一张张缓存的图片，每个文件都对应着一张图片，而journal文件是DiskLruCache的一个日志文件，程序对每张图片的操作记录都存放在这个文件中，基本上看到`journal这个文件就标志着该程序使用DiskLruCache技术`了。

---
### OKHttp 流程（以同步请求为例）

#### 基本使用
```java
OkHttpClient client = new OkHttpClient();
                Request request = new Request.Builder().url("http://www.baidu.com")
                        .build();
                try {
                    Response response = client.newCall(request).execute();
                    if (response.isSuccessful()) {
                        System.out.println("成功");
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
``` 

#### 同步请求流程

    ![](/img/in-post/post-Android/OKHttp/流程.png)