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
### 整体框架

#### OKHttp 总流程图

![](/img/in-post/post-Android/OKHttp/whole流程.png)


#### 整体架构

![](/img/in-post/post-Android/OKHttp/framework.png)



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