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

**参考博客**
- [okhttp源码解析](https://blog.csdn.net/json_it/article/details/78404010)
- [Android DiskLruCache完全解析，硬盘缓存的最佳方案](https://blog.csdn.net/guolin_blog/article/details/28863651)
- [HTTP版本及异同整理](https://blog.csdn.net/json_it/article/details/78312311)
- [Okhttp3 总结研究](https://blog.csdn.net/u012881042/article/details/79759203)
- [okhttp 流程和优化的实现](https://blog.csdn.net/jun_tong/article/details/79808634)


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
## PART I——概述

---
### 整体框架

#### OKHttp 总流程图

![](/img/in-post/post-Android/OKHttp/whole流程.png)


#### 整体架构

![](/img/in-post/post-Android/OKHttp/framework.png)

---
### OKHttp 异步流程实现

![](/img/in-post/post-Android/OKHttp/okhttp_full_process2.png)

当我们请求的时候会新建一个 RealCall 的对象，创建之后会通过 dispatcher 去在线程池中分配线程，这个 dispatcher 的主要作用就是调度请求，这里面有三个队列，**作用分别是存储异步正在运行的任务**，**存储异步正在准备运行的任务**，**还有同步运行的队列**，通过这个类为我们的任务分配一个线程去运行

#### 拦截器的主要功能如下：

1. **RetryAndFollowUpInterceptor（负责失败重试，重定向的）**
    - 概述：主要的作用就是请求时候创建一个 StreamAllocation 对象，这个对象创建的时候会根据请求的协议不同创建不一样的对象
2. **BridgeInterceptor**(负责把用户构造的请求转换为发送到服务器的请求、把服务器返回的响应转换为用户友好的响应的)
    - 概述：主要就是把我们的 request 请求加上一些请求头，打包成真正的网络请求的 request，在请求返回的时候，通过 gzip 把我们能的 response 进行压缩
3. **CacheInterceptor**(负责读取缓存的，如果有缓存就拦截并返回，也负责更新缓存)
    - 概述：负责读取我们的缓存，如果我们设置了 cache，那么先从这个里面读取，如果读取不到的那么就把 request 和 caseResponse 构建一个 CaheStategy 对象，然后判断这个对象是否有效，如果有效则直接返回，如果无效，那么我们请求，如果请求返回304，证明资源没过期，我们可以读取本地的缓存
4. **ConnectInterceptor**(负责和服务器建立连接的)
    - 概述：这个就比较复杂了，这个里面呢主要是进行 socket 的连接，在这个拦截器里首先获取了一个 streamAllocation 对象，然后通过这个对象获取了一个 RealConnection 对象，然后通过这个对象去获取一个 httpcode 对象，这个对象是一个接口，那具体实现有两种一个是 http1，一个是 http2，它会根据我们的请求创建不同的 httpCode ，通过这个对象可以进行下一个拦截器的操作，在我们获取 realconnection 的同时我们调用了 connect 这个方法，这个方法底层调用的就是 socket.connect 的方法，就是进行了 socket 的连接了
5. 配置 OkHttpClient 时设置的 **networkInterceptors**
6. **CallServerInterceptor**(负责向服务器发送请求数据、从服务器读取响应数据的)

>注意：我们每个拦截器在 intercept 方法中都会调用 process 这个方法，这个方法的作用就是去执行下一个拦截器，那么如果我们自定义拦截器的话，需要调用 chain.proceed 方法，不然的话我们的请求就会在拦截器里面卡主。

---
### okhttp 中有哪些优化，优化是怎么实现的

**1.多路复用**
- 概述：之前我们的请求是每一次请求会建立一个链接，请求结束就关闭链接，我们都知道 TCP/IP 请求是需要握手的，那握手就会消耗相应的时间，所以在我们的 okhttp 中，我们会复用之前的链接进行请求，这样请求速度就快了很多。
- 如果是这样的话就会出现两个问题，`第一，我们怎么判断链接是否可用`，`第二是我们不需要的链接怎么回收`。
- 从上面我们知道我们会在 StreamAllocation.newStream 方法中获取 RealConnection，在获取的时候我们会判断有没有之前的链接可以复用，复用的条件是这样判断的：
    1. 如果此链接的负载数目超过指定数目（表现为RealConnection的allocations集合的数量超过该链接指定的数量）或者noNewStreams为true时，此链接不可复用。 
    2. StreamAllocation 所持有的Address对象和RealConnection的Address非主机部分不同，则此链接不可复用。至于非主机部分的判定是在Address的equalsNonHost方法来体现。两者Adress对象的非主机部分相等的标准就是dns,Authenticator对象、协议、CA授权验证标准、端口等信息全部相等。
    3. 在1、2判定条件都为true的话，如果两个Address对象的host或者说url中的host一样，则此链接可复用，正如注释说说，添加1、2、3都满足的话，那么此时这个链接就是This connection is a perfect match。

第一个问题我们解决了，现在我们来解决第二个问题，我们都知道链接如果多了我们如果不回收的话就会卡死，那么我们的链接是怎么回收的呢，链接回收主要降到的就是 ConnectionPool 这个类，这个类中有一个 clean up 方法，我们来看一下他里面做了什么：
- clean up 方法：(标记清除法)
    1. 首先标记出最不活跃的链接（空闲链接），之后进行清除
    2. 如果被标记的链接空闲 socket 超过 5 个，时间大于 5 分钟，那么直接清除。
    3. 如果此链接空闲，但是不足五分钟，则返回剩余时间，并进行标记，以供下次清除。
    4. 如果没用空闲链接的话，则五分钟之后再进行清理
- 判断是否为空闲链接：
    1. 遍历其中的 StreamAllocation，判断是否为空，如果为空，则没有引用这个 StreamAllocation
    2. 如果引用数量为 0 ，则为空闲链接

多路复用的原理就讲到这里了，其实很简单，只要吧 ConnectionPool 这个类看明白就很好理解了

**2.缓存**
- 概述：我们都知道，好的框架都会有缓存的功能，通过缓存我们可以很快的访问我们的资源，那 okhttp 也不例外，从上面的流程中我们可以看到， CacheInterceptor 主要是做缓存的，那么我们来了解一下他的流程是什么：

okhttp 的缓存策略是，key 为 Request的 url 的 MD5 值，value 为 response。
1. 如果在 okhttpclient 初始化的时候配置了 cache，那么我们则从缓存中读取 caseResponse。
2. 如果没有指定，那么我们将 request 和 caseResponse 构建一个 CacheStrategy 的类
3. 判断 cachestrategy 是否有效，如果 request 和 caseResponse 都为空，直接返回 504
4. 如果 request == null ，cacheResponse 不为空，则返回
5. 如果为空，那么我们就进行网络请求，如果返回了 304 且我们本地有缓存，那么说明我们的缓存没有过期，可以继续使用



**3.线程池的引入**
- 概述：我们之前的请求都是每次都会新建线程去进行请求，这样的话我们如果有 100 个请求就会有 100 个线程，那么会消耗很大的资源，当引入线程池之后，我们就不需要频繁的去创建线程，而且可以复用线程，这样就很节省时间了。

**4.可以进行压缩**
- 在流程中我们讲到，当我们 response 通过 bridgeInterceptor 处理的时候会进行 gzip 压缩，这样可以大大减小我们的 response ，他不是什么情况下都压缩的，只有支持的时候才会 进行压缩。



---
### 运用到的设计模式：

**单例模式**：（`建议用单例模式创建okHttpClient`）OkHttpClient， 可以通过 new OkHttpClient() 或 new OkHttpClient.Builder() 来创建对象， 但是---特别注意， OkHttpClient() 对象最好是共享的， 建议使用单例模式创建。 因为每个 OkHttpClient 对象都管理自己独有的线程池和连接池。 这一点很多同学，甚至在我经历的团队中就有人踩过坑， 每一个请求都创建一个 OkHttpClient 导致内存爆掉

**外观模式** : OKHttpClient 里面组合了很多的类对象。其实是将OKHttp的很多功能模块，全部包装进这个类中，让这个类单独提供对外的API，这种设计叫做外观模式（外观模式：隐藏系统的复杂性，并向客户端提供了一个客户端可以访问系统的接口）

**Builder模式** : OkHttpClient 比较复杂， 太多属性， 而且客户的组合需求多样化， 所以OKhttp使用建造者模式（Build模式：使用多个简单的对象一步一步构建成一个复杂的对象，一个 Builder 类会一步一步构造最终的对象）

**工厂方法模式**：Call接口提供了内部接口Factory(用于将对象的创建延迟到该工厂类的子类中进行，从而实现动态的
配置，工厂方法模式。（工厂方法模式：这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。）

**享元模式**：在Dispatcher的线程池中，所用到了享元模式，一个不限容量的线程池 ， 线程空闲时存活时间为 60 秒。线程池实现了对象复用，降低线程创建开销，从设计模式上来讲，使用了享元模式。（享元模式：尝试重用现有的同类对象，如果未找到匹配的对象，则创建新对象，主要用于减少创建对象的数量，以减少内存占用和提高性能）

**责任链模式**：很明显，在okhttp中的拦截器模块，执行过程用到。OkHttp3 的拦截器链中， 内置了5个默认的拦截器，分别用于重试、请求对象转换、缓存、链接、网络读写（责任链模式：为请求创建了一个接收者对象的链。这种模式给予请求的类型，对请求的发送者和接收者进行解耦。这种类型的设计模式属于行为型模式。在这种模式中，通常每个接收者都包含对另一个接收者的引用。如果一个对象不能处理该请求，那么它会把相同的请求传给下一个接收者，依此类推。）

**策略模式** ：CacheInterceptor 实现了数据的选择策略， 来自网络还是来自本地？ 这个场景也是比较契合策略模式场景， CacheInterceptor 需要一个策略提供者提供它一个策略（锦囊）， CacheInterceptor 根据这个策略去选择走网络数据还是本地缓存。
缓存的策略过程：
1. 请求头包含 "If-Modified-Since" 或 "If-None-Match" 暂时不走缓存
2. 客户端通过 cacheControl 指定了无缓存，不走缓存
3. 客户端通过 cacheControl 指定了缓存，则看缓存过期时间，符合要求走缓存。
4. 如果走了网络请求，响应状态码为 304（只有客户端请求头包含 "If-Modified-Since" 或 "If-None-Match" ，服务器数据没变化的话会返回304状态码，不会返回响应内容）， 表示客户端继续用缓存。

（策略模式：一个类的行为或其算法可以在运行时更改。这种类型的设计模式属于行为型模式。策略模式中，我们创建表示各种策略的对象和一个行为随着策略对象改变而改变的 context 对象。策略对象改变 context 对象的执行算法。）


---
## PART II——详细分析

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

- `Connections`：连接远程服务器的物理连接；
- `Streams`：基于Connection的逻辑Http请求/响应对。一个连接可以承载多少个Stream都是有限制的，Http1.x连接只能承载一个Stream，而一个Http2.0连接可以承载多个Stream（支持并发请求，并发请求共用一个Connection）；
- `Calls`：逻辑Stream序列，典型的例子是一个初始请求及其后续的请求。We prefer to keep all streams of a single call on the same  connection for better behavior and locality.

对于同步和异步请求，唯一的区别就是异步请求会放在`线程池（ThreadPoolExecutor）`中去执行，而同步请求则会在当前线程中执行，注意：`同步请求会阻塞当前线程。`

- 对于Http1.1，call - 1:1 - Stream - 1:1 - connection;
- 对于http2.0，call - 1:1 - **Stream - N:1** - connection;

由上述流程图，我们可以直观的了解到一次基本的请求包括如下两个部分：`call+interceptors`。
- `call`:最终的请求对象；
- `interceptors`:这是OkHttp最核心的部分，`一个请求会经过OkHttp的若干个拦截器进行处理，每一个拦截器都会完成一个功能模块`，比如CacheInterceptor完成网络请求的缓存。`一个Request经过拦截器链的处理之后，会得到最终的Response`。interceptors里面包括的东西很多东西，后续的源码分析就是以拦截器为主线来进行分析。


---
### 源码分析
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

#### 1、OkHttpClient

首先，我们生成了一个OKHttpClient对象，注意OKHttpClient对象的生成有两种方式：一种是我们使用的方式，另一种是使用建造者（Builder）模式 -- new OkHttpClient.Builder()....Build()。那么这两种方式有什么区别呢？

第一种：
```java
  public OkHttpClient() {
    this(new Builder());
  }
public Builder() {
      dispatcher = new Dispatcher();
      protocols = DEFAULT_PROTOCOLS;
      connectionSpecs = DEFAULT_CONNECTION_SPECS;
      eventListenerFactory = EventListener.factory(EventListener.NONE);
      proxySelector = ProxySelector.getDefault();
      cookieJar = CookieJar.NO_COOKIES;
      socketFactory = SocketFactory.getDefault();
      hostnameVerifier = OkHostnameVerifier.INSTANCE;
      certificatePinner = CertificatePinner.DEFAULT;
      proxyAuthenticator = Authenticator.NONE;
      authenticator = Authenticator.NONE;
      connectionPool = new ConnectionPool();
      dns = Dns.SYSTEM;
      followSslRedirects = true;
      followRedirects = true;
      retryOnConnectionFailure = true;
      connectTimeout = 10_000;
      readTimeout = 10_000;
      writeTimeout = 10_000;
      pingInterval = 0;
    }
```
简单的一句new OkHttpClient()，OkHttp就已经为我们做了很多工作，很多我们需要的参数在这里都获得默认值。各字段含义如下：
- **dispatcher**：直译就是调度器的意思。主要作用是通过双端队列保存Calls（同步&异步Call），同时在线程池中执行异步请求。后面会详细解析该类。
- **protocols**：默认支持的Http协议版本  --   Protocol.HTTP_2, Protocol.HTTP_1_1；
- **connectionSpecs**：OKHttp连接（Connection）配置 -- ConnectionSpec.MODERN_TLS, ConnectionSpec.CLEARTEXT，我们分别看一下：

第二种：默认的设置和第一种方式相同，但是我们可以利用建造者模式单独的设置每一个属性.

注意事项：OkHttpClient强烈建议**全局单例使用**，因为每一个OkHttpClient都有自己单独的连接池和线程池，复用连接池和线程池能够减少延迟、节省内存。


#### 2、RealCall（生成一个Call）
在我们定义了请求对象request之后，我们需要生成一个Call对象，该对象代表了一个准备被执行的请求。Call是可以被取消的。Call对象代表了一个request/response 对（Stream）.还有就是一个Call只能被执行一次。执行同步请求，代码如下（RealCall的execute方法）：
```java
@Override public Response execute() throws IOException {
    synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    captureCallStackTrace();
    eventListener.callStart(this);
    try {
      client.dispatcher().executed(this);
      Response result = getResponseWithInterceptorChain();
      if (result == null) throw new IOException("Canceled");
      return result;
    } catch (IOException e) {
      eventListener.callFailed(this, e);
      throw e;
    } finally {
      client.dispatcher().finished(this);
    }
  }
```
解析：首先如果executed等于true，说明已经被执行，如果再次调用执行就抛出异常。这说明了一个Call只能被执行。注意此处同步请求与异步请求生成的Call对象的区别，执行异步请求代码如下（RealCall的enqueue方法）：
```java
@Override public void enqueue(Callback responseCallback) {
    synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    captureCallStackTrace();
    eventListener.callStart(this);
    client.dispatcher().enqueue(new AsyncCall(responseCallback));
  }
```
可以看到同步请求生成的是RealCall对象，而异步请求生成的是`AsyncCall对象`。AsyncCall说到底其实就是Runnable的子类。
接着上面继续分析，如果可以执行，则对当前请求添加监听器等操作，然后将请求Call对象放入调度器Dispatcher中。最后由拦截器链中的各个拦截器来对该请求进行处理，返回最终的Response。


#### 3、Dispatcher(调度器)

Dispatcher是保存同步和异步Call的地方，并负责执行异步AsyncCall。

![](/img/in-post/post-Android/OKHttp/dispatcher.png)

如上图，`针对同步请求，Dispatcher使用了一个Deque保存了同步任务`；**针对异步请求，Dispatcher使用了两个Deque，一个保存准备执行的请求，一个保存正在执行的请求**，为什么要用两个呢？
- 因为Dispatcher默认支持最大的并发请求是64个，单个Host最多执行5个并发请求，如果超过，则Call会先被放入到readyAsyncCall中，当出现空闲的线程时，再将readyAsyncCall中的线程移入到runningAsynCalls中，执行请求。先看Dispatcher的流程，跟着流程读源码：

![](/img/in-post/post-Android/OKHttp/dispatcher2.png)

#### 4、拦截器链
在依次介绍各个拦截器之前，先介绍一个比较重要的类：RealInterceptorChain，直译就是拦截器链类；这个类在什么地方会用到呢？还是3.2节，RealCall的execute方法有这么一段代码：
```java
Response result = getResponseWithInterceptorChain();
```
没错，在getResponseWithInterceptorChain();方法中我们就用到了这个RealInterceptorChain类。
```java
Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    interceptors.addAll(client.interceptors());
    interceptors.add(retryAndFollowUpInterceptor);
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    interceptors.add(new CacheInterceptor(client.internalCache()));
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
      interceptors.addAll(client.networkInterceptors());
    }
    interceptors.add(new CallServerInterceptor(forWebSocket));
 
    Interceptor.Chain chain = new RealInterceptorChain(interceptors, null, null, null, 0,
        originalRequest, this, eventListener, client.connectTimeoutMillis(),
        client.readTimeoutMillis(), client.writeTimeoutMillis());
 
    return chain.proceed(originalRequest);
  }
```
在该方法中，我们依次添加了用户自定义的`interceptor、retryAndFollowUpInterceptor、BridgeInterceptor、CacheInterceptor、ConnectInterceptor、 networkInterceptors、CallServerInterceptor`，并将这些拦截器传递给了这个`RealInterceptorChain。`拦截器之所以可以依次调用，并最终再从后先前返回Response，都依赖于RealInterceptorChain的**proceed**方法。

可以看到当前拦截器的Response依赖于下一个拦截器的Intercept的Response。因此，就会沿着这条拦截器链依次调用每一个拦截器，当执行到最后一个拦截器之后，就会沿着相反的方向依次返回Response，最终得到我们需要的“终极版”Response。

#### 4.1 重试及 followup拦截器
```java
     @Override
        public Response intercept(Chain chain) throws IOException {
            Request request = chain.request();//获取Request对象
            RealInterceptorChain realChain = (RealInterceptorChain) chain;//获取拦截器链对象，用于后面的chain.proceed(...)方法
            Call call = realChain.call();
            EventListener eventListener = realChain.eventListener();//监听器
 
            streamAllocation = new StreamAllocation(client.connectionPool(), createAddress(request.url()),
                    call, eventListener, callStackTrace);
 
            int followUpCount = 0;
            Response priorResponse = null;
            while (true) {//循环
                if (canceled) {
                    streamAllocation.release();
                    throw new IOException("Canceled");
                }
 
                Response response;
                boolean releaseConnection = true;
                try {
                    response = realChain.proceed(request, streamAllocation, null, null);//调用下一个拦截器
                    releaseConnection = false;
                } catch (RouteException e) {
                    // The attempt to connect via a route failed. The request will not have been sent.
                    if (!recover(e.getLastConnectException(), false, request)) {//路由异常，尝试恢复，如果再失败就抛出异常
                        throw e.getLastConnectException();
                    }
                    releaseConnection = false;
                    continue;//继续重试
                } catch (IOException e) {
                    // An attempt to communicate with a server failed. The request may have been sent.
                    boolean requestSendStarted = !(e instanceof ConnectionShutdownException);
                    if (!recover(e, requestSendStarted, request)) throw e;连接关闭异常，尝试恢复
                            releaseConnection = false;
                    continue;//继续重试
                } finally {
                    // We're throwing an unchecked exception. Release any resources.
                    if (releaseConnection) {
                        streamAllocation.streamFailed(null);
                        streamAllocation.release();
                    }
                }
 
                // Attach the prior response if it exists. Such responses never have a body.
                if (priorResponse != null) {//前一个重试得到的Response
                    response = response.newBuilder()
                            .priorResponse(priorResponse.newBuilder()
                                    .body(null)
                                    .build())
                            .build();
                }
                //Figures out the HTTP request to make in response to receiving {@code userResponse}. This will
                //either add authentication headers, follow redirects or handle a client request timeout. If a
                //follow-up is either unnecessary or not applicable, this returns null.
                // followUpRequest方法的主要作用就是为新的重试Request添加验证头等内容
                Request followUp = followUpRequest(response);
                
                if (followUp == null) {//如果一个请求得到的响应code是200，则followUp是为null的。
                    if (!forWebSocket) { streamAllocation.release(); } return response; } 
                closeQuietly(response.body());
                //-------------------------------异常处理--------------------------------------------- 
                // if (++followUpCount > MAX_FOLLOW_UPS) {//超过最大的次数,抛出异常 
                 streamAllocation.release(); 
                throw new ProtocolException("Too many follow-up requests: " + followUpCount); }
            if (followUp.body() instanceof UnrepeatableRequestBody) {
                streamAllocation.release();
            } throw new HttpRetryException("Cannot retry streamed HTTP body", response.code()); 
        } if (!sameConnection(response, followUp.url())) {
            streamAllocation.release();
            streamAllocation = new StreamAllocation(client.connectionPool(), createAddress(followUp.url()), call, eventListener, callStackTrace);
        } else if (streamAllocation.codec() != null) {
            throw new IllegalStateException("Closing the body of " + response + " didn't close its backing stream. Bad interceptor?");
        }
        //--------------------------------------------------------------------------------
        request = followUp;//得到处理之后的Request，以用来继续请求，在哪继续请求？肯定还是沿着拦截器链继续搞呗 
         priorResponse = response;//由priorResponse持有 
         }
    }
 }
```
拦截器主要的作用就是重试及followup(这个followup咋翻译比较贴切呢？)。当一个请求由于各种原因失败了，如果是路由或者连接异常，则尝试恢复，否则，根据响应码（ResponseCode）,followup方法会对Request进行再处理以得到新的Request，然后沿着拦截器链继续新的Request。当然，如果responseCode是200的话，这些过程就结束了。注意看注释。


#### 4.2 BridgeInterceptor

![](/img/in-post/post-Android/OKHttp/BridgeInterceptor.png)

BridgeInterceptor的主要作用就是为`请求（request before）添加请求头`，`为响应（Response Before）添加响应头`。

#### 4.3 CacheInterceptor
在解析CacheInterceptor之前，先看一张关于Http缓存机制的图片（来源于网络）：

![](/img/in-post/post-Android/OKHttp/cache.png)

先看一下缓存的响应头：

![](/img/in-post/post-Android/OKHttp/cache2.png)

看几个与`CacheInterceptor`相关的比较重要的几个类：

![](/img/in-post/post-Android/OKHttp/CacheInterceptor.png)

CacheStrategy是一个缓存策略类，该类告诉CacheInterceptor是使用缓存还是使用网络请求；
Cache是封装了实际的缓存操作；
DiskLruCache:Cache基于DiskLruCache；

根据缓存策略类返回的结果：
1. 如果网络不可用并且无可用的有效缓存，则返回504错误；
2. 继续，如果不需要网络请求，则直接使用缓存；
3. 继续，如果需要网络可用，则进行网络请求；
4. 继续，如果有缓存，并且网络请求返回HTTP_NOT_MODIFIED，说明缓存还是有效的，则合并网络响应和缓存结果。同时更新缓存；
5. 继续，如果没有缓存，则写入新的缓存；

我们可以看到，CacheStrategy在CacheInterceptor中起到了很关键的作用。该类决定了是网络请求还是使用缓存。

大致流程如下：（if-else的关系）
1. 没有缓存，直接网络请求；
2. 如果是https，但没有握手，直接网络请求；
3. 不可缓存，直接网络请求；
4. 请求头nocache或者请求头包含If-Modified-Since或者If-None-Match，则需要服务器验证本地缓存是不是还能继续使用，直接网络请求；
5. 可缓存，并且ageMillis + minFreshMillis < freshMillis + maxStaleMillis（意味着虽过期，但可用，只是会在响应头添加warning），则使用缓存；
6. 缓存已经过期，添加请求头：If-Modified-Since或者If-None-Match，进行网络请求；


#### 4.4 ConnectInterceptor（核心，连接池）
ConnectInterceptor器如其名，是一个连接相关的拦截器。这个拦截器是这几个拦截器里面代码最少的。但是少并不意味着很简单。先看一下ConnectIntercepor中比较重要的几个类及其含义：

![](/img/in-post/post-Android/OKHttp/ConnectInterceptor.png)

- `RouteDataBase`：这是一个关于路由信息的白名单和黑名单类，处于黑名单的路由信息会被避免不必要的尝试；
- `RealConnecton`：Connect子类，主要实现连接的建立等工作；
- `ConnectionPool`:连接池，实现连接的复用；
- `StreamAllocation`：直译就是流分配。流是什么呢？我们知道Connection是一个连接远程服务器的物理Socket连接，而Stream则是基于Connection的逻辑Http 请求/响应对。StreamAllocation会通过ConnectPool获取或者新生成一个RealConnection来得到一个连接到Server的Connection连接，同时会生成一个HttpCodec用于下一个CallServerInterceptor，以完成最终的请求；
- `HttpCodec`： Encodes HTTP requests and decodes HTTP responses。（源码注释哦）。针对不同的版本，OkHttp为我们提供了HttpCodec1（Http1.x）和HttpCodec2(Http2).

>一句话概括就是：分配一个Connection和HttpCodec，为最终的请求做准备。

**Connection和Stream的关系：Http1.x是1:1的关系,而Http2是1对多的关系。就是说一个http1.x连接只能被一个请求使用，而一个Http2连接是对应多个Stream的，多个Stream的意思是Http2连接支持并发请求，即一个连接可以被多个请求同时使用的。**

**还有，Http1.1的keep-alive机制的作用是保证连接使用完不关闭，当下一次请求与连接的Host相同的时候，连接可以直接使用，不用再次创建（节省资源，提高了性能）。**

![](/img/in-post/post-Android/OKHttp/ConnectInterceptor2.png)

在Connectinterceptor中，起到关键作用的就是`ConnectionPool`，既然这么关键我们就来看看这个连接池吧。

![](/img/in-post/post-Android/OKHttp/ConnectInterceptorConnectionPool.png)

在目前的版本下，连接池默认是可以保持`5个空闲的连接`。这些空闲的连接如果`超过5分钟不被使用，则将被连接池移除`。
当然，这些默认的数值在未来的okhttp版本中，会被改变的。另外，`这两个数值支持开发人员修改`。

ConnectionPool中比较关键的几个点，`线程池（ThreadPoolExecutor）、队列（Deque）、路由记录表`；
1. 线程池：用于支持连接池的cleanup任务，清除idle线程；
2. 队列：存放待复用的连接；
3. 路由记录表：前面已讲，不再叙述；

#### 4.5 CallServerInterceptor

该拦截器就是利用HttpCodec完成最终请求的发送