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
```java
Picasso.with(this).load(“http://nuuneoi.com/uploads/source/playstore/cover.jpg“).into(ivImgPicasso);

Picasso.with(this).load(“http://nuuneoi.com/uploads/source/playstore/cover.jpg“).resize(768, 432).into(ivImgPicasso);

Picasso.with(this).load(“http://nuuneoi.com/uploads/source/playstore/cover.jpg“).fit().centerCrop().into(ivImgPicasso);
```
- 第一种：加载了全尺寸的图片到内存，然后让GPU来实时重绘大小
- 第二种：你需要主动计算ImageView的大小，或者说你的ImageView大小是具体的值（而不是wrap_content），
- 第三种：按统一比例缩放图片(保存图片的尺寸比例)便于图片的二维(宽度和高度)等于或者大于相应的视图的维度，这种方法和Glide加载图片占用的内存几乎是相同的，虽然内存开销差距不大，但是在这个问题上Glide完胜Picasso。因为Glide可以自动计算出任意情况下的ImageView的大小。

#### 重要的类介绍

0. `Picasso`: 图片加载、转换、缓存的管理类。单例模式，通过with方法获取实例，也是加载图片的入口。
1. `RequestCreator`: Request构建类，Builder 模式，采用链式设置该Request的属性(如占位图、缓存策略、裁剪规则、显示大小、优先级等等)。最后调用build()方法生成一个请求（Request）。
2. `DeferredRequestCreator`:RequestCreator的包装类,当创建请求的时候还不能获取ImageView的宽和高的时候，则创建一个DeferredRequestCreator，DeferredRequestCreator里对 target 设置监听，直到可以获取到宽和高的时候重新执行请求创建。
3. `Action`: 请求包装类，存储了该请求和RequestCreator设置的这些属性，最终提交给线程执行下载。
4. `Dispatcher`:分发器，分发执行各种请求、分发结果等等。
5. `PicassoExecutorService`:Picasso使用的线程池，默认池大小为3。
6. `LruCache`:一个使用最近最少使用策略的内存缓存。
7. `BitmapHunter`:**这是Picasso的一个核心的类，开启线程执行下载，获取结果后解码成Bitmap,然后做一些转换操作如图片旋转、裁剪等，如果请求设置了转换器Transformation,也会在BitmapHunter里执行这些转换操作。**
8. `NetworkRequestHandler`:网络请求处理器，如果图片需要从网络下载，则用这个处理器处理。
9. `FileRequestHandler`:文件请求处理器，如果请求的是一张存在文件中的图片，则用这个处理器处理。
10. `AssetRequestHandler`: Asset 资源图片处理器，如果是加载asset目录下的图片，则用这个处理器处理。
11. `ResourceRequestHandler`:Resource资源图片处理器，如果是加载res下的图片，则用这个处理器处理。
12. `ContentStreamRequestHandler`: ContentProvider 处理器，如果是ContentProvider提供的图片，则用这个处理器处理
13. `MediaStoreRequestHandler`: MediaStore 请求处理器，如果图片是存在MediaStore上的则用这个处理器处理。
14. `ContactsPhotoRequestHandler`:ContactsPhoto 请求处理器，如果加载com.android.contacts/ 下的tu图片用这个处理器处理。如：
```java
// e.g. content://com.android.contacts/contacts/38)
//匹配的路径如下：
static {
    matcher = new UriMatcher(UriMatcher.NO_MATCH);
    matcher.addURI(ContactsContract.AUTHORITY, "contacts/lookup/*/#", ID_LOOKUP);
    matcher.addURI(ContactsContract.AUTHORITY, "contacts/lookup/*", ID_LOOKUP);
    matcher.addURI(ContactsContract.AUTHORITY, "contacts/#/photo", ID_THUMBNAIL);
    matcher.addURI(ContactsContract.AUTHORITY, "contacts/#", ID_CONTACT);
    matcher.addURI(ContactsContract.AUTHORITY, "display_photo/#", ID_DISPLAY_PHOTO);
}
```
>上面8-14 是默认的提供的几个处理器，分别处理不同来源的请求。
15. `Response`: 返回的结果信息，Stream流或者Bitmap。
16. `Request`: 请求实体类，存储了应用在图片上的信息。
17. `Target`：图片加载的监听器接口，有3个回调方法，onPrepareLoad 在请求提交前回调，onBitmapLoaded 请求成功回调，并返回Bitmap,onBitmapFailed请求失败回调。
18. `PicassoDrawable`：继承BitmapDrawable，实现了过渡动画和图片来源的标识（就是图片来源的指示器，要调用 setIndicatorsEnabled(true)方法才生效），请求成功后都会包装成BitmapDrawable显示到ImageView 上。
19. `OkHttpDownloader`：用OkHttp实现的图片下载器，默认就是用的这个下载器。
20. `UrlConnectionDownloader`：使用HttpURLConnection 实现的下载器。
21. `MemoryPolicy`: 内存缓存策略，一个枚举类型。
22. `NetworkPolicy`: 磁盘缓存策略，一个枚举类型。
23. `Stats`: 这个类相当于日志记录，会记录如：内存缓存的命中次数，丢失次数，下载次数，转换次数等等，我们可以通过StatsSnapshot类将日志打印出来，看一下整个项目的图片加载情况。
24. `StatsSnapshot` :状态快照，和上面的Stats对应，打印Stats纪录的信息。

### Picasso流程分析

#### 1.获取Picasso instance
首先要获取一个Picasso对象，采用的单例模式
```java
//单例模式获取Picasso 对象
public static Picasso with(Context context) {
    if (singleton == null) {
      synchronized (Picasso.class) {
        if (singleton == null) {
          singleton = new Builder(context).build();
        }
      }
    }
    return singleton;
  }

// 真正new 的地方在build()方法里
 public Picasso build() {
      Context context = this.context;

      if (downloader == null) {
        //配置默认的下载器，首先通过反射获取OkhttpClient,如果获取到了，就使用OkHttpDwownloader作为默认下载器
        //如果获取不到就使用UrlConnectionDownloader作为默认下载器
        downloader = Utils.createDefaultDownloader(context);
      }
      if (cache == null) {
       // 配置内存缓存，默认大小为手机内存的15%
        cache = new LruCache(context);
      }
      if (service == null) {
       // 配置Picaso 线程池，默认核心池大小为3
        service = new PicassoExecutorService();
      }
      if (transformer == null) {
       // 配置请求转换器，默认的请求转换器没有做任何事，直接返回原请求
        transformer = RequestTransformer.IDENTITY;
      }

      Stats stats = new Stats(cache);
      //分发器 
      Dispatcher dispatcher = new Dispatcher(context, service, HANDLER, downloader, cache, stats);

      return new Picasso(context, dispatcher, cache, listener, transformer, requestHandlers, stats,
          defaultBitmapConfig, indicatorsEnabled, loggingEnabled);
    }
```

#### 2.通过load方法生成一个RequestCreator
通过load方法生成一个RequestCreator，用链式api 来构建一个图片下载请求
```java
//load有几个重载方法，参数为string和File 的重载最重都会包装成一个Uri 调用这个方法
 public RequestCreator load(Uri uri) {
    return new RequestCreator(this, uri, 0);
  }
// 如果是加载资源id 的图片会调用这个方法
 public RequestCreator load(int resourceId) {
    if (resourceId == 0) {
      throw new IllegalArgumentException("Resource ID must not be zero.");
    }
    return new RequestCreator(this, null, resourceId);
  }
```

RequestCreator提供了很多的API 来构建请求，如展位图、大小、转换器、裁剪等等,这些API其实是为对应的属性赋值，最终会在into方法中构建请求。

```java
// 配置占位图，在加载图片的时候显示
public RequestCreator placeholder(int placeholderResId) {
    if (!setPlaceholder) {
      throw new IllegalStateException("Already explicitly declared as no placeholder.");
    }
    if (placeholderResId == 0) {
      throw new IllegalArgumentException("Placeholder image resource invalid.");
    }
    if (placeholderDrawable != null) {
      throw new IllegalStateException("Placeholder image already set.");
    }
    this.placeholderResId = placeholderResId;
    return this;
  }

// 配置真正显示的大小
 public RequestCreator resize(int targetWidth, int targetHeight) {
    data.resize(targetWidth, targetHeight);
    return this;
  }
```

#### 3.into 添加显示的View，并且提交下载请求
into方法里面干了3件事情：
1. 判断是否设置了fit 属性，如果设置了，再看是否能够获取ImageView 的宽高，如果获取不到，生成一个DeferredRequestCreator（延迟的请求管理器），然后直接return，在DeferredRequestCreator中当监听到可以获取ImageView 的宽高的时候，再执行into方法。
2. 判断是否从`内存缓存`获取图片，如果没有设置NO_CACHE，则从内存获取，命中直接回调CallBack 并且显示图片。
3. 如果缓存未命中，则生成一个Action，并提交Action。
```java
public void into(ImageView target, Callback callback) {
    long started = System.nanoTime();
   // 检查是否在主线程
    checkMain();

    if (target == null) {
      throw new IllegalArgumentException("Target must not be null.");
    }
   //如果没有url或者resourceId 则取消请求
    if (!data.hasImage()) {
      picasso.cancelRequest(target);
      if (setPlaceholder) {
        setPlaceholder(target, getPlaceholderDrawable());
      }
      return;
    }
    //判断是否设置了fit属性
    if (deferred) {
      if (data.hasSize()) {
        throw new IllegalStateException("Fit cannot be used with resize.");
      }
      int width = target.getWidth();
      int height = target.getHeight();
      if (width == 0 || height == 0) {
        if (setPlaceholder) {
          setPlaceholder(target, getPlaceholderDrawable());
        }
       //如果获取不到宽高，生成一个DeferredRequestCreator（延迟的请求管理器），然后直接return,
      //在DeferredRequestCreator中当监听到可以获取ImageView 的宽高的时候，再执行into方法。
        picasso.defer(target, new DeferredRequestCreator(this, target, callback));
        return;
      }
      data.resize(width, height);
    }

    Request request = createRequest(started);
    String requestKey = createKey(request);
    //是否从内存缓存中获取
    if (shouldReadFromMemoryCache(memoryPolicy)) {
      Bitmap bitmap = picasso.quickMemoryCacheCheck(requestKey);
      if (bitmap != null) {
       //缓存命中，取消请求，并显示图片
        picasso.cancelRequest(target);
        setBitmap(target, picasso.context, bitmap, MEMORY, noFade, picasso.indicatorsEnabled);
        if (picasso.loggingEnabled) {
          log(OWNER_MAIN, VERB_COMPLETED, request.plainId(), "from " + MEMORY);
        }
        if (callback != null) {
          callback.onSuccess();
        }
        return;
      }
    }

    if (setPlaceholder) {
      setPlaceholder(target, getPlaceholderDrawable());
    }
   //内存缓存未命中或者设置了不从内存缓存获取，则生成一个Action ，提交执行。
    Action action =
        new ImageViewAction(picasso, target, request, memoryPolicy, networkPolicy, errorResId,
            errorDrawable, requestKey, tag, callback, noFade);

    picasso.enqueueAndSubmit(action);// 提交请求
  }
```

#### 4.提交、分发、执行请求。
会经过下面这一系列的操作，最终将Action 交给BitmapHunter 执行。

`enqueueAndSubmit －> submit －> dispatchSubmit －> performSubmit：`
```java
//将action 保存到了一个Map 中，目标View作为key
 void enqueueAndSubmit(Action action) {
    Object target = action.getTarget();
    if (target != null && targetToAction.get(target) != action) {
      // This will also check we are on the main thread.
      cancelExistingRequest(target);
      targetToAction.put(target, action);
    }
    submit(action);
  }

// 交给分发器分发提交请求
void submit(Action action) {
    dispatcher.dispatchSubmit(action);
  }

 
//执行请求提交
//1, 先查看保存暂停tag表里面没有包含Action的tag,如果包含，则将Action 存到暂停Action表里
//2，从BitmapHunter表里查找有没有对应action的hunter,如果有直接attach
//3, 为这个请求生成一个BitmapHunter，提交给线程池执行
 void performSubmit(Action action, boolean dismissFailed) {
    // 先查看保存暂停tag表里面没有包含Action的tag,如果包含，则将Action 存到暂停Action表里
    if (pausedTags.contains(action.getTag())) {
      pausedActions.put(action.getTarget(), action);
      if (action.getPicasso().loggingEnabled) {
        log(OWNER_DISPATCHER, VERB_PAUSED, action.request.logId(),
            "because tag '" + action.getTag() + "' is paused");
      }
      return;
    }

    BitmapHunter hunter = hunterMap.get(action.getKey());
    if (hunter != null) {
      hunter.attach(action);
      return;
    }
   // 如果线程池被shutDown,直接return
    if (service.isShutdown()) {
      if (action.getPicasso().loggingEnabled) {
        log(OWNER_DISPATCHER, VERB_IGNORED, action.request.logId(), "because shut down");
      }
      return;
    }
    // 为请求生成一个BitmapHunter
    hunter = forRequest(action.getPicasso(), this, cache, stats, action);
    hunter.future = service.submit(hunter);//提交执行
    hunterMap.put(action.getKey(), hunter);
    if (dismissFailed) {
      failedActions.remove(action.getTarget());
    }

    if (action.getPicasso().loggingEnabled) {
      log(OWNER_DISPATCHER, VERB_ENQUEUED, action.request.logId());
    }
  }
```

#### 5.指定对应的处理器（RequestHandler）
在上面执行的请求的performSubmit 方法里，调用了forRequest 方法为对应的Action 生成一个BitmapHunter,里面有一个重要的步骤，指定请求处理器（在上面一节介绍Picasso有7种请求处理器,看一下对应的代码：
```java
static BitmapHunter forRequest(Picasso picasso, Dispatcher dispatcher, Cache cache, Stats stats,
      Action action) {
    Request request = action.getRequest();
    List<RequestHandler> requestHandlers = picasso.getRequestHandlers();

    // Index-based loop to avoid allocating an iterator.
    //noinspection ForLoopReplaceableByForEach
    for (int i = 0, count = requestHandlers.size(); i < count; i++) {
      RequestHandler requestHandler = requestHandlers.get(i);
     // 循环请求处理器列表，如果找到有能处理这个请求的请求处理器
     // 则生成BitmapHunter
      if (requestHandler.canHandleRequest(request)) {
        return new BitmapHunter(picasso, dispatcher, cache, stats, action, requestHandler);
      }
    }

    return new BitmapHunter(picasso, dispatcher, cache, stats, action, ERRORING_HANDLER);
  }
```

从Picasso里获取一个处理器列表，然后循环列表，看是否有能处理该请求的处理器，如果有，则生成BitmapHunter,那么这个请求处理器的列表在哪儿初始化的呢？请看源码：

```java
// 1,首先调用了getRequestHandlers
 List<RequestHandler> getRequestHandlers() {
    return requestHandlers;
  }

// 2 requestHandlers 列表是在Picasso 构造函数里出实话的
 Picasso(Context context, Dispatcher dispatcher, Cache cache, Listener listener,
      RequestTransformer requestTransformer, List<RequestHandler> extraRequestHandlers, Stats stats,
      Bitmap.Config defaultBitmapConfig, boolean indicatorsEnabled, boolean loggingEnabled) {
 
    ....
   //前面代码省略
 
   // 添加了7个内置的请求处理器
  // 如果你自己通过Builder添了额外的处理器，也会添加在这个列表里面

    int builtInHandlers = 7; // Adjust this as internal handlers are added or removed.
    int extraCount = (extraRequestHandlers != null ? extraRequestHandlers.size() : 0);
    List<RequestHandler> allRequestHandlers =
        new ArrayList<RequestHandler>(builtInHandlers + extraCount);

    // ResourceRequestHandler needs to be the first in the list to avoid
    // forcing other RequestHandlers to perform null checks on request.uri
    // to cover the (request.resourceId != 0) case.
    allRequestHandlers.add(new ResourceRequestHandler(context));
    if (extraRequestHandlers != null) {
      allRequestHandlers.addAll(extraRequestHandlers);
    }
    allRequestHandlers.add(new ContactsPhotoRequestHandler(context));
    allRequestHandlers.add(new MediaStoreRequestHandler(context));
    allRequestHandlers.add(new ContentStreamRequestHandler(context));
    allRequestHandlers.add(new AssetRequestHandler(context));
    allRequestHandlers.add(new FileRequestHandler(context));
    allRequestHandlers.add(new NetworkRequestHandler(dispatcher.downloader, stats));
    requestHandlers = Collections.unmodifiableList(allRequestHandlers);
   
  //后面代码省略 
    ...

  }
```
`小结： 在Picasso 的构造函数里 初始化了内置的7中请求处理器，然后在生成BitmapHunter的时候，循环列表，找到可以处理对应请求的处理器。`

#### 6.重点：BitmapHunter (图片捕获器)
上一节重要类介绍的时候介绍过BitmapHunter，BitmapHunter继承Runnable，其实就是开启一个线程执行最终的下载。看一下源码：
1. run() 方法
```java
@Override public void run() {
    try {
      updateThreadName(data);

      if (picasso.loggingEnabled) {
        log(OWNER_HUNTER, VERB_EXECUTING, getLogIdsForHunter(this));
      }
     //  调用hunt() 方法获取最终结果
      result = hunt();

      if (result == null) {
        dispatcher.dispatchFailed(this);//如果为null，分发失败的消息
      } else {
        dispatcher.dispatchComplete(this);//如果不为null，分发成功的消息
      }
    } catch (Downloader.ResponseException e) {
      if (!e.localCacheOnly || e.responseCode != 504) {
        exception = e;
      }
      dispatcher.dispatchFailed(this);
    } catch (NetworkRequestHandler.ContentLengthException e) {
      exception = e;
      dispatcher.dispatchRetry(this);
    } catch (IOException e) {
      exception = e;
      dispatcher.dispatchRetry(this);
    } catch (OutOfMemoryError e) {
      StringWriter writer = new StringWriter();
      stats.createSnapshot().dump(new PrintWriter(writer));
      exception = new RuntimeException(writer.toString(), e);
      dispatcher.dispatchFailed(this);
    } catch (Exception e) {
      exception = e;
      dispatcher.dispatchFailed(this);
    } finally {
      Thread.currentThread().setName(Utils.THREAD_IDLE_NAME);
    }
  }
```
**当将一个bitmapHunter submit 给一个线程池执行的时候，就会执行run() 方法，run里面调用的是hunt方法来获取结果，看一下`hunt方法`：**
```java
Bitmap hunt() throws IOException {
    Bitmap bitmap = null;
   // 是否从内存缓存获取Bitmap
    if (shouldReadFromMemoryCache(memoryPolicy)) {
      bitmap = cache.get(key);
      if (bitmap != null) {
        stats.dispatchCacheHit();
        loadedFrom = MEMORY;
        if (picasso.loggingEnabled) {
          log(OWNER_HUNTER, VERB_DECODED, data.logId(), "from cache");
        }
        return bitmap;
      }
    }

    data.networkPolicy = retryCount == 0 ? NetworkPolicy.OFFLINE.index : networkPolicy;
   // 请求处理器处理请求，获取结果，Result里可能是Bitmap，可能是Stream
    RequestHandler.Result result = requestHandler.load(data, networkPolicy);
    if (result != null) {
      loadedFrom = result.getLoadedFrom();
      exifRotation = result.getExifOrientation();

      bitmap = result.getBitmap();

      // If there was no Bitmap then we need to decode it from the stream.
      if (bitmap == null) {
        InputStream is = result.getStream();
        try {
          bitmap = decodeStream(is, data);
        } finally {
          Utils.closeQuietly(is);
        }
      }
    }

    if (bitmap != null) {
      if (picasso.loggingEnabled) {
        log(OWNER_HUNTER, VERB_DECODED, data.logId());
      }
      stats.dispatchBitmapDecoded(bitmap);
      if (data.needsTransformation() || exifRotation != 0) {
        synchronized (DECODE_LOCK) {
          if (data.needsMatrixTransform() || exifRotation != 0) {
            //如果需要做转换，则在这里做转换处理，如角度旋转，裁剪等。
            bitmap = transformResult(data, bitmap, exifRotation);
            if (picasso.loggingEnabled) {
              log(OWNER_HUNTER, VERB_TRANSFORMED, data.logId());
            }
          }
          if (data.hasCustomTransformations()) {
           // 如果配置了自定义转换器，则在这里做转换处理。
            bitmap = applyCustomTransformations(data.transformations, bitmap);
            if (picasso.loggingEnabled) {
              log(OWNER_HUNTER, VERB_TRANSFORMED, data.logId(), "from custom transformations");
            }
          }
        }
        if (bitmap != null) {
          stats.dispatchBitmapTransformed(bitmap);
        }
      }
    }

    return bitmap;
  }
```
#### 7.Downloader 下载器下载图片
上面的hunt方法获取结果的时候，最终调用的是配置的处理器的load方法，如下：
```java
RequestHandler.Result result = requestHandler.load(data, networkPolicy);
```
加载网络图片用的是NetworkRequestHandler，匹配处理器，有个canHandleRequest 方法：
```java
@Override public boolean canHandleRequest(Request data) {
    String scheme = data.uri.getScheme();
    return (SCHEME_HTTP.equals(scheme) || SCHEME_HTTPS.equals(scheme));
  }
```
**判断的条件是，Uri带有"http://" 或者 https:// 前缀则可以处理**

我们接下来看一下NetworkRequestHandler的load方法：
```java
@Override public Result load(Request request, int networkPolicy) throws IOException {
    //最终调用downloader的load方法获取结果
    Response response = downloader.load(request.uri, request.networkPolicy);
    if (response == null) {
      return null;
    }

    Picasso.LoadedFrom loadedFrom = response.cached ? DISK : NETWORK;

    Bitmap bitmap = response.getBitmap();
    if (bitmap != null) {
      return new Result(bitmap, loadedFrom);
    }

    InputStream is = response.getInputStream();
    if (is == null) {
      return null;
    }
    // Sometimes response content length is zero when requests are being replayed. Haven't found
    // root cause to this but retrying the request seems safe to do so.
    if (loadedFrom == DISK && response.getContentLength() == 0) {
      Utils.closeQuietly(is);
      throw new ContentLengthException("Received response with 0 content-length header.");
    }
    if (loadedFrom == NETWORK && response.getContentLength() > 0) {
      stats.dispatchDownloadFinished(response.getContentLength());
    }
    return new Result(is, loadedFrom);
  }
```

NetworkRequestHandler最终是调用的downloader 的load方法下载图片。内置了2个Downloader，OkhttpDownloader和UrlConnectionDownloader 。我们以UrlConnectionDownloader为例，来看一下load方法：

```java
@Override public Response load(Uri uri, int networkPolicy) throws IOException {
   // 如果SDK 版本大于等于14,安装磁盘缓存，用的是HttpResponseCache（缓存http或者https的response到文件系统）
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.ICE_CREAM_SANDWICH) {
      installCacheIfNeeded(context);
    }

    HttpURLConnection connection = openConnection(uri);
   //设置使用缓存
    connection.setUseCaches(true);

    if (networkPolicy != 0) {
      String headerValue;
     // 下面一段代码是设置缓存策略
      if (NetworkPolicy.isOfflineOnly(networkPolicy)) {
        headerValue = FORCE_CACHE;
      } else {
        StringBuilder builder = CACHE_HEADER_BUILDER.get();
        builder.setLength(0);

        if (!NetworkPolicy.shouldReadFromDiskCache(networkPolicy)) {
          builder.append("no-cache");
        }
        if (!NetworkPolicy.shouldWriteToDiskCache(networkPolicy)) {
          if (builder.length() > 0) {
            builder.append(',');
          }
          builder.append("no-store");
        }

        headerValue = builder.toString();
      }

      connection.setRequestProperty("Cache-Control", headerValue);
    }

    int responseCode = connection.getResponseCode();
    if (responseCode >= 300) {
      connection.disconnect();
      throw new ResponseException(responseCode + " " + connection.getResponseMessage(),
          networkPolicy, responseCode);
    }

    long contentLength = connection.getHeaderFieldInt("Content-Length", -1);
    boolean fromCache = parseResponseSourceHeader(connection.getHeaderField(RESPONSE_SOURCE));
    // 最后获取InputStream流包装成Response返回
    return new Response(connection.getInputStream(), fromCache, contentLength);
  }
```

**小结：梳理一下调用链, BitmapHunter －> NetworkRequestHandler －> UrlConnectionDownloader(也有可能是OkHttpDownloader),经过这一系列的调用，最后在BitmapHunter 的run 方法中就可以获取到我们最终要的Bitmap。**

#### 8. 返回结果并显示在Target上
在BitmapHunter获取结果后，分发器分发结果,通过Hander处理后，执行performComplete方法：

```java
//1,
void performComplete(BitmapHunter hunter) {
    // 这里将结果缓存到内存
    if (shouldWriteToMemoryCache(hunter.getMemoryPolicy())) {
      cache.set(hunter.getKey(), hunter.getResult());
    }
    hunterMap.remove(hunter.getKey());// 请求完毕，将hunter从表中移除
    batch(hunter);
    if (hunter.getPicasso().loggingEnabled) {
      log(OWNER_DISPATCHER, VERB_BATCHED, getLogIdsForHunter(hunter), "for completion");
    }
  }

// 2,然后将BitmapHunter添加到一个批处理列表，通过Hander发送一个批处理消息
private void batch(BitmapHunter hunter) {
    if (hunter.isCancelled()) {
      return;
    }
    batch.add(hunter);
    if (!handler.hasMessages(HUNTER_DELAY_NEXT_BATCH)) {
      handler.sendEmptyMessageDelayed(HUNTER_DELAY_NEXT_BATCH, BATCH_DELAY);
    }
  }
// 3，最后执行performBatchComplete 方法，通过主线程的Handler送处理完成的消息
void performBatchComplete() {
    List<BitmapHunter> copy = new ArrayList<BitmapHunter>(batch);
    batch.clear();
    mainThreadHandler.sendMessage(mainThreadHandler.obtainMessage(HUNTER_BATCH_COMPLETE, copy));
    logBatch(copy);
  }

// 4,最后在Picasso 中handleMessage,显示图片
 static final Handler HANDLER = new Handler(Looper.getMainLooper()) {
    @Override public void handleMessage(Message msg) {
      switch (msg.what) {
        case HUNTER_BATCH_COMPLETE: {
          @SuppressWarnings("unchecked") List<BitmapHunter> batch = (List<BitmapHunter>) msg.obj;
          //noinspection ForLoopReplaceableByForEach
          for (int i = 0, n = batch.size(); i < n; i++) {
            BitmapHunter hunter = batch.get(i);
            hunter.picasso.complete(hunter);
          }
          break;
        }
      //后面代码省略
      ...
  };
// 5,最后回调到ImageViewAction 的complete方法显示图片
@Override public void complete(Bitmap result, Picasso.LoadedFrom from) {
    if (result == null) {
      throw new AssertionError(
          String.format("Attempted to complete action with no result!\n%s", this));
    }

    ImageView target = this.target.get();
    if (target == null) {
      return;
    }

    Context context = picasso.context;
    boolean indicatorsEnabled = picasso.indicatorsEnabled;
   //将结果包装成一个PicassoDrawable 并显示
    PicassoDrawable.setBitmap(target, context, result, from, noFade, indicatorsEnabled);

    if (callback != null) {
      callback.onSuccess(); 回调callback
    }
  }
```

**小结：通过上面一系列的方法调用， performComplete －> batch —> performBatchComplete －> handleMessage －> complete 把BitmapHunter中获取到的结果回调到主线程，并且显示在Target上。**

### 缓存特别说明
`内存缓存`用的是`LRUCache`，大小为 手机内存的15% ,上面代码中已经分析过了，这里不过多说明，这里重点说一下磁盘缓存Disk Cahce。Picasso内存了2个默认的下载器，UrlConnectionDownloader和OkHttpDownloader，它们的`磁盘缓存`实现还是有一些差异的，看一下代码：
```java
public OkHttpDownloader(final File cacheDir, final long maxSize) {
    this(defaultOkHttpClient());
    try {
      client.setCache(new com.squareup.okhttp.Cache(cacheDir, maxSize));
    } catch (IOException ignored) {
    }
  }
```

**在OkHttpDownloader 的构造方法里设置了磁盘缓存，使用的okHttp 的 DiskLruCache 实现的。**

然后看一下UrlConnectionDownloader的磁盘缓存实现，代码：
```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.ICE_CREAM_SANDWICH) {
      installCacheIfNeeded(context);
    }
```
```java
private static void installCacheIfNeeded(Context context) {
    // DCL + volatile should be safe after Java 5.
    if (cache == null) {
      try {
        synchronized (lock) {
          if (cache == null) {
            cache = ResponseCacheIcs.install(context);
          }
        }
      } catch (IOException ignored) {
      }
    }
  }

  private static class ResponseCacheIcs {
    static Object install(Context context) throws IOException {
      File cacheDir = Utils.createDefaultCacheDir(context);
      HttpResponseCache cache = HttpResponseCache.getInstalled();
      if (cache == null) {
        long maxSize = Utils.calculateDiskCacheSize(cacheDir);
        cache = HttpResponseCache.install(cacheDir, maxSize);
      }
      return cache;
    }

    static void close(Object cache) {
      try {
        ((HttpResponseCache) cache).close();
      } catch (IOException ignored) {
      }
    }
  }
```

**UrlConnectionDownloader 的磁盘缓存是用HttpResponseCache实现的**

尽管2种磁盘缓存实现的方式不一样，但是它们的最后结果都是一样的：
1. `磁盘缓存的地址`： 磁盘缓存的地址在：data/data/your package name/cache/picasso-cache /
2. `磁盘缓存的大小`：磁盘缓存的大小为 手机磁盘大小的2% ，不超过50M不小于5M。
3. `缓存的控制方式一样`：都是在请求的header设置Cache-Control的值来控制是否缓存。

#### 缓存清除：
1. 清除内存缓存：调用invalidate方法,如：
```java
Picasso.with(this)
       .invalidate("http://ww3.sinaimg.cn/large/610dc034jw1fasakfvqe1j20u00mhgn2.jpg");
```

但是Picasso没有提供清除全部内存缓存的方法，那就没有办法了吗？办法还是有的,LRUCahce 提供了clear方法的,只是Picasso没有向外部提供这个接口，因此可以通过反射获取到Picasso的cache字段，然后调用clear方法清除。

2. 清除磁盘缓存
很遗憾Picasso没有提供清除磁盘缓存的方法。它没有提供方法我们就自己想办法呗。
思路：很简单，既然我们知道磁盘缓存是存在：data/data/your package name/cache/picasso-cache 这个路径下的，那我们把这个文件夹下面的所有文件清除不就行了。
```java
private void clearDiskCache(){
        File cache = new File(this.getApplicationContext().getCacheDir(), "picasso-cache");
        deleteFileOrDirectory(cache.getPath());
    }

    public static void deleteFileOrDirectory(String filePath){
        if(TextUtils.isEmpty(filePath)){
            return;
        }
        try {
            File file = new File(filePath);
            if(!file.exists()){
                return;
            }
            if(file.isDirectory()){
                File files[] = file.listFiles();
                for(int i=0;i<files.length;i++){
                    deleteFileOrDirectory(files[i].getAbsolutePath());
                }
            }else{
                file.delete();
                Log.e("zhouwei","delete cache...");
            }
        }catch (Exception e){
            e.printStackTrace();
        }

    }
```

---
### Glide
以Glide3.8.0版本来分析，我们先看下最常见使用方法：
```java
Glide.with(fragment)
     .load(myUrl)
     .into(imageView);
```
- with(context)
- load(url)
- into(target)

#### with(context)方法

看一下with(context)d的源码：

```java
public static RequestManager with(Context context) {
        RequestManagerRetriever retriever = RequestManagerRetriever.get();
        return retriever.get(context);
    }

    public static RequestManager with(Activity activity) {
        RequestManagerRetriever retriever = RequestManagerRetriever.get();
        return retriever.get(activity);
    }
    
    public static RequestManager with(FragmentActivity activity) {
        RequestManagerRetriever retriever = RequestManagerRetriever.get();
        return retriever.get(activity);
    }
    
    public static RequestManager with(android.app.Fragment fragment) {
        RequestManagerRetriever retriever = RequestManagerRetriever.get();
        return retriever.get(fragment);
    }
    
    public static RequestManager with(Fragment fragment) {
        RequestManagerRetriever retriever = RequestManagerRetriever.get();
        return retriever.get(fragment);
    }
```

可以看到，with方法有很多，但内容基本一致，都是通过 `RequestManagerRetriever.get()`; 获取一个 `RequestManagerRetriever 对象 retriever` ，然后通过 `retriever.get(context)`; 获取一个 RequestManager 对象并返回。这些with方法关键的不同在于传入的参数不一致，可以是`Context、Activity、Fragment`等等。那么为什么要分这么多种呢？其实我们应该都知道：Glide在加载图片的时候会绑定 with(context) 方法中传入的 context 的生命周期，如果传入的是 Activity ，那么在这个 **Activity 销毁的时候Glide会停止图片的加载。这样做的好处是显而易见的：避免了消耗多余的资源，也避免了在Activity销毁之后加载图片从而导致的空指针问题**。

为了更好的分析 with(context) 中的这两步，我们来看一下 `RequestManagerRetriever` ：
```java
public class RequestManagerRetriever implements Handler.Callback {
    //饿汉式创建单例
    private static final RequestManagerRetriever INSTANCE = new RequestManagerRetriever();
    
    //返回单例对象
    public static RequestManagerRetriever get() {
        return INSTANCE;
    }

    //根据传入的参数，获取不同的RequestManager
    public RequestManager get(Context context) {
        if (context == null) {
            throw new IllegalArgumentException("You cannot start a load on a null Context");
        } else if (Util.isOnMainThread() && !(context instanceof Application)) {
            if (context instanceof FragmentActivity) {
                return get((FragmentActivity) context);
            } else if (context instanceof Activity) {
                return get((Activity) context);
            } else if (context instanceof ContextWrapper) {
                return get(((ContextWrapper) context).getBaseContext());
            }
        }

        return getApplicationManager(context);
    }
    
    //省略无关代码......
}
```

上面这个方法主要是通过传入context的不同类型来做不同的操作。context可以是Application、FragmentActivity、Activity或者是ContextWrapper。我们先看一下当context是Application时的操作：
```java
private RequestManager getApplicationManager(Context context) {
        // 返回一个单例
        if (applicationManager == null) {
            synchronized (this) {
                if (applicationManager == null) {
                    applicationManager = new RequestManager(context.getApplicationContext(),
                            new ApplicationLifecycle(), new EmptyRequestManagerTreeNode());
                }
            }
        }

        return applicationManager;
    }
```

`getApplicationManager(Context context)` 通过单例模式创建并返回了 applicationManager 。我们再来看一下如果传入的context是Activity时的操作：
```java
public RequestManager get(Activity activity) {
        //如果不在主线程或者Android SDK的版本低于HONEYCOMB，传入的还是Application类型的context
        if (Util.isOnBackgroundThread() || Build.VERSION.SDK_INT < Build.VERSION_CODES.HONEYCOMB) {
            return get(activity.getApplicationContext());
        } else {
            //判断当前activity是否被销毁
            assertNotDestroyed(activity);
            android.app.FragmentManager fm = activity.getFragmentManager();
           //通过fragmentGet(activity, fm)获取RequestManager
            return fragmentGet(activity, fm);
        }
    }
```

如果不在主线程或者Android SDK版本过低，走的还是传入Application的方法，这个方法在上面提到过；反之，首先判断当前activity是否被销毁，如果没有被销毁，则通过fragmentGet(activity, fm)获取RequestManager。关键是这个 fragmentGet(activity, fm) ，我们来看一下：

```java
RequestManager fragmentGet(Context context, android.app.FragmentManager fm) {
        //在当前activity中创建一个没有界面的的fragment并add到当前activity中
        RequestManagerFragment current = getRequestManagerFragment(fm);
        RequestManager requestManager = current.getRequestManager();
        if (requestManager == null) {
            //创建一个requestManager
            requestManager = new RequestManager(context, current.getLifecycle(), current.getRequestManagerTreeNode());
            //将requestManager与fragment绑定        
            current.setRequestManager(requestManager);
        }
        return requestManager;
    }
```
`fragmentGet 这个方法主要是在当前activity中创建一个没有界面的的fragment并add到当前activity中，以此来实现对activity生命周期的监听。`
#### 总结：
- 通过RequestManagerRetriever的get获取RequestManagerRetriever单例对象
- 通过retriever.get(context)获取RequestManager，在get(context)方法中通过对context类型的判断做不同的处理：
    - context是Application，通过getApplicationManager(Context context) 创建并返回一个RequestManager对象
    - context是Activity，通过fragmentGet(activity, fm)在当前activity创建并添加一个没有界面的fragment，从而实现图片加载与activity的生命周期相绑定，之后创建并返回一个RequestManager对象
- 我们先来看传入`Application参数`的情况。如果在Glide.with()方法中传入的是一个Application对象，那么这里就会调用带有Context参数的get()方法重载，然后会在调用getApplicationManager()方法来获取一个RequestManager对象。其实这是最简单的一种情况，因为`Application对象的生命周期即应用程序的生命周期，因此Glide并不需要做什么特殊的处理`，它自动就是和应用程序的生命周期是同步的，如果应用程序关闭的话，Glide的加载也会同时终止。
- 接下来我们看传入`非Application参数`的情况。不管你在Glide.with()方法中传入的是Activity、FragmentActivity、v4包下的Fragment、还是app包下的Fragment，最终的流程都是一样的，那就是会向当前的Activity当中`添加一个隐藏的Fragment`。那么这里为什么要添加一个隐藏的Fragment呢？因为Glide需要知道加载的生命周期。很简单的一个道理，如果你在某个Activity上正在加载着一张图片，结果图片还没加载出来，Activity就被用户关掉了，那么图片还应该继续加载吗？当然不应该。可是`Glide并没有办法知道Activity的生命周期，于是Glide就使用了添加隐藏Fragment的这种小技巧`，因为`Fragment的生命周期和Activity是同步的`，如果Activity被销毁了，Fragment是可以监听到的，这样Glide就可以捕获这个事件并停止图片加载了。
- 这里额外再提一句，如果我们是在`非主线程当中使用的Glide`，那么不管你是传入的Activity还是Fragment，都会被强制当成Application来处理。
- 总体来说，with()方法其实就是为了得到一个RequestManager对象而已，然后Glide会根据我们传入with()方法的参数来确定图片加载的生命周期


#### load(url)方法
with(context)返回一个RequestManager，接下来我们看一下RequestManger中的load(url)方法：
```java
public DrawableTypeRequest<String> load(String string) {
        return (DrawableTypeRequest<String>) fromString().load(string);
    }
```
这个方法分两步：`fromString()`、`load(string)`，先看第一个方法：

```java
public DrawableTypeRequest<String> fromString() {
        return loadGeneric(String.class);
    }
```
这个方法返回的是 loadGeneric(String.class) ，我们跟进去：

```java
private <T> DrawableTypeRequest<T> loadGeneric(Class<T> modelClass) {
        ModelLoader<T, InputStream> streamModelLoader = Glide.buildStreamModelLoader(modelClass, context);
        ModelLoader<T, ParcelFileDescriptor> fileDescriptorModelLoader =
                Glide.buildFileDescriptorModelLoader(modelClass, context);
        if (modelClass != null && streamModelLoader == null && fileDescriptorModelLoader == null) {
            throw new IllegalArgumentException("Unknown type " + modelClass + ". You must provide a Model of a type for"
                    + " which there is a registered ModelLoader, if you are using a custom model, you must first call"
                    + " Glide#register with a ModelLoaderFactory for your custom model class");
        }

        //这句是核心，本质是创建并返回了一个DrawableTypeRequest
        return optionsApplier.apply(
                new DrawableTypeRequest<T>(modelClass, streamModelLoader, fileDescriptorModelLoader, context,
                        glide, requestTracker, lifecycle, optionsApplier));
    }
```
在 `loadGeneric(Class<T> modelClass)` 方法中，我们只需要关注核心即可。它的核心是最后一句，方法调用看着很复杂，其实本质是创建并返回了一个DrawableTypeRequest，Drawable类型的请求。再来看 `load(string)` 方法：

```java
@Override
    public DrawableRequestBuilder<ModelType> load(ModelType model) {
        super.load(model);
        return this;
    }
```
需要注意的是这个方法存在于DrawableTypeRequest的父类DrawableRequestBuilder中，这个方法首先调用DrawableRequestBuilder的父类的load方法，然后返回自身。再看一下DrawableRequestBuilder父类中的load方法：

```java
public GenericRequestBuilder<ModelType, DataType, ResourceType, TranscodeType> load(ModelType model) {
        this.model = model;
        isModelSet = true;
        return this;
    }
```
DrawableRequestBuilder的父类是GenericRequestBuilder，从名字中我们也可以看出来，前者是Drawable请求的构建者，后者是通用的请求构建者，他们是子父关系。这个load方法其实是把我们传入的String类型的URL存入了内部的model成员变量中，再将数据来源是否已经设置的标志位 isModelSet 设置为true，意味着我们在调用 `Glide.with(context).load(url)` 之后数据来源已经设置成功了。

使用的是Builder模式

#### into(imageView)方法
简单的说，Glide中的前两步是创建了一个Request，这个Request可以理解为对图片加载的配置请求，需要注意的是仅仅是创建了一个 请求 ，而并没有去执行。在Glide的最后一步into方法中，这个请求才会真实的执行。

我们来DrawableTypeRequest中找一下into方法，发现没找到，那肯定是在他的父类DrawableRequestBuilder中，我们来看一下`DrawableRequestBuilder中的into方法`：
```java
public Target<GlideDrawable> into(ImageView view) {
        return super.into(view);
    }
```
它调用的是父类GenericRequestBuilder的方法，那我们继续看`GenericRequestBuilder的into方法`：

```java
public Target<TranscodeType> into(ImageView view) {
        //确保在主线程
        Util.assertMainThread();
        //确保view不为空
        if (view == null) {
            throw new IllegalArgumentException("You must pass in a non null View");
        }
        //对ScaleType进行配置
        if (!isTransformationSet && view.getScaleType() != null) {
            switch (view.getScaleType()) {
                case CENTER_CROP:
                    applyCenterCrop();
                    break;
                case FIT_CENTER:
                case FIT_START:
                case FIT_END:
                    applyFitCenter();
                    break;
                //$CASES-OMITTED$
                default:
                    // Do nothing.
            }
        }

        //核心
        return into(glide.buildImageViewTarget(view, transcodeClass));
    }
```
可以看到，上面的方法就是into的核心代码了，它定义在`GenericRequestBuilder`这个通用的请求构建者中。方法的核心是最后一行：` into(glide.buildImageViewTarget(view, transcodeClass))` ，首先是通过 `glide.buildImageViewTarget(view, transcodeClass) `创建出一个
Target 类型的对象，然后把这个target传`入GenericRequestBuilder中的into方法中`。我们先来看一下Glide中的
buildImageViewTarget(view, transcodeClass) 方法：

```java
 <R> Target<R> buildImageViewTarget(ImageView imageView, Class<R> transcodedClass) {
        return imageViewTargetFactory.buildTarget(imageView, transcodedClass);
    }
```
这个方法的目的是`把我们传入的imageView包装成一个Target`。内部调用了 imageViewTargetFactory.buildTarget(imageView, transcodedClass) 继续跟进去看一下：

```java
public <Z> Target<Z> buildTarget(ImageView view, Class<Z> clazz) {
        //图片来源是GlideDrawable
        if (GlideDrawable.class.isAssignableFrom(clazz)) {
            //创建GlideDrawable对应的target
            return (Target<Z>) new GlideDrawableImageViewTarget(view);
        } else if (Bitmap.class.equals(clazz)) {
            //如果图片来源是Bitmap，创建Bitmap对应的target
            return (Target<Z>) new BitmapImageViewTarget(view);
        } else if (Drawable.class.isAssignableFrom(clazz)) {
            //如果图片来源是Drawable，创建Drawable对应的target
            return (Target<Z>) new DrawableImageViewTarget(view);
        } else {
            throw new IllegalArgumentException("Unhandled class: " + clazz
                    + ", try .as*(Class).transcode(ResourceTranscoder)");
        }
    }
```
这个方法的的本质是：通过对图片来源类型的判断，创建并返回与图片来源对应的imageViewTarget。获取到相应的target之后，我们来看GenericRequestBuilder中的into方法：

```java
public <Y extends Target<TranscodeType>> Y into(Y target) {
        //确保在主线程
        Util.assertMainThread();
        //确保target不为空
        if (target == null) {
            throw new IllegalArgumentException("You must pass in a non null Target");
        }
        //确保数据来源已经确定，即已经调用了load(url)方法
        if (!isModelSet) {
            throw new IllegalArgumentException("You must first set a model (try #load())");
        }

        //获取当前target已经绑定的Request对象
        Request previous = target.getRequest();

        //如果当前target已经绑定了Request对象，则清空这个Request对象
        if (previous != null) {
            previous.clear();
            //停止绑定到当前target的上一个Request的图片请求处理
            requestTracker.removeRequest(previous);
            previous.recycle();
        }
        
        //创建Request对象
        Request request = buildRequest(target);
        //与target绑定
        target.setRequest(request);
        lifecycle.addListener(target);
        //执行request
        requestTracker.runRequest(request);

        return target;
    }
```

我们梳理一下方法中的逻辑：
- 获取当前target中的Request对象，如果存在，则清空并终止这个Request对象的执行
- 创建新的Request对象并与当前target绑定
- 执行新创建的图片处理请求Request

逻辑还是比较清晰的，这里有一个问题需要说明一下。`为什么要终止并清除target之前绑定的请求呢`？
- 在没有Glide之前，我们处理ListView中的图片加载其实是一件比较麻烦的事情。由于ListView中Item的复用机制，会导致网络图片加载的错位或者闪烁。那我们解决这个问题的办法也很简单，就是给当前的ImageView设置tag，这个tag可以是图片的URL等等。当从网络中获取到图片时判断这个ImageVIew中的tag是否是这个图片的URL，如果是就加载图片，如果不是则跳过。
- 在有了Glide之后，我们处理ListView或者Recyclerview中的图片加载就很无脑了，根本不需要作任何多余的操作，直接正常使用就行了。这其中的原理是Glide给我们处理了这些判断，我们来看一下Glide内部是如何处理的：
```java
public Request getRequest() {
        //本质还是getTag
        Object tag = getTag();
        Request request = null;
        if (tag != null) {
            if (tag instanceof Request) {
                request = (Request) tag;
            } else {
                throw new IllegalArgumentException("You must not call setTag() on a view Glide is targeting");
            }
        }
        return request;
    }
    
    @Override
    public void setRequest(Request request) {
        //本质是setTag
        setTag(request);
    }
```

可以看到， target.getRequest() 和 target.setRequest(Request request) 本质上还是通过`setTag和getTag`来做的处理，这也印证了我们上面所说。

继续回到into方法中，在创建并绑定了Request后，关键的就是 requestTracker.runRequest(request) 来执行我们创建的请求了。
```java
public void runRequest(Request request) {
        //将请求加入请求集合
        requests.add(request);
        
        if (!isPaused) {
            如果处于非暂停状态，开始执行请求
            request.begin();
        } else {
            //如果处于暂停状态，将请求添加到等待集合
            pendingRequests.add(request);
        }
    }
```
这个方法定义在 RequestTracker 中，这个类主要负责`Request的执行，暂停，取消`等等关于图片请求的操作。我们着重看 `request.begin() `，这句代码意味着开始执行图片请求的处理。Request是个接口， request.begin() 实际调用的是Request的子类 `GenericRequest 的begin方法`，我们跟进去看一下：
```java
 @Override
    public void begin() {
        startTime = LogTime.getLogTime();
        if (model == null) {
            onException(null);
            return;
        }

        status = Status.WAITING_FOR_SIZE;
        if (Util.isValidDimensions(overrideWidth, overrideHeight)) {
            //如果长宽尺寸已经确定
            onSizeReady(overrideWidth, overrideHeight);
        } else {
            //获取长宽尺寸，获取完之后会调用onSizeReady(overrideWidth, overrideHeight)
            target.getSize(this);
        }

        if (!isComplete() && !isFailed() && canNotifyStatusChanged()) {
            //开始加载图片，先显示占位图
            target.onLoadStarted(getPlaceholderDrawable());
        }
        if (Log.isLoggable(TAG, Log.VERBOSE)) {
            logV("finished run method in " + LogTime.getElapsedMillis(startTime));
        }
    }
```
- 获取图片的长宽尺寸，如果长宽已经确定，走 `onSizeReady(overrideWidth, overrideHeight) `流程；如果未确定，先获取长宽，再走 onSizeReady(overrideWidth, overrideHeight)

- 图片开始加载，首先显示占位图

可以明白，主要的逻辑还是在`onSizeReady` 这个方法中：
```java
@Override
    public void onSizeReady(int width, int height) {
        if (Log.isLoggable(TAG, Log.VERBOSE)) {
            logV("Got onSizeReady in " + LogTime.getElapsedMillis(startTime));
        }
        if (status != Status.WAITING_FOR_SIZE) {
            return;
        }
        status = Status.RUNNING;

        width = Math.round(sizeMultiplier * width);
        height = Math.round(sizeMultiplier * height);

        ModelLoader<A, T> modelLoader = loadProvider.getModelLoader();
        final DataFetcher<T> dataFetcher = modelLoader.getResourceFetcher(model, width, height);

        if (dataFetcher == null) {
            onException(new Exception("Failed to load model: \'" + model + "\'"));
            return;
        }
        ResourceTranscoder<Z, R> transcoder = loadProvider.getTranscoder();
        if (Log.isLoggable(TAG, Log.VERBOSE)) {
            logV("finished setup for calling load in " + LogTime.getElapsedMillis(startTime));
        }
        loadedFromMemoryCache = true;
        
        //核心代码，加载图片
        loadStatus = engine.load(signature, width, height, dataFetcher, loadProvider, transformation, transcoder,
                priority, isMemoryCacheable, diskCacheStrategy, this);
                
        loadedFromMemoryCache = resource != null;
        if (Log.isLoggable(TAG, Log.VERBOSE)) {
            logV("finished onSizeReady in " + LogTime.getElapsedMillis(startTime));
        }
    }
```
这段代码看起来很复杂，我们只需要关注核心代码：
```java
    engine.load(signature, width, height, dataFetcher, loadProvider, transformation, transcoder, priority, isMemoryCacheable, diskCacheStrategy, this)，
```
我们看一下load方法内部做了什么处理：
```java
public <T, Z, R> LoadStatus load(Key signature, int width, int height, DataFetcher<T> fetcher,
            DataLoadProvider<T, Z> loadProvider, Transformation<Z> transformation, ResourceTranscoder<Z, R> transcoder,
            Priority priority, boolean isMemoryCacheable, DiskCacheStrategy diskCacheStrategy, ResourceCallback cb) {
        Util.assertMainThread();
        long startTime = LogTime.getLogTime();

        final String id = fetcher.getId();
        EngineKey key = keyFactory.buildKey(id, signature, width, height, loadProvider.getCacheDecoder(),
                loadProvider.getSourceDecoder(), transformation, loadProvider.getEncoder(),
                transcoder, loadProvider.getSourceEncoder());

        //使用LruCache获取缓存
        EngineResource<?> cached = loadFromCache(key, isMemoryCacheable);
        if (cached != null) {
            //从缓存中获取资源成功
            cb.onResourceReady(cached);
            if (Log.isLoggable(TAG, Log.VERBOSE)) {
                logWithTimeAndKey("Loaded resource from cache", startTime, key);
            }
            return null;
        }

        //从弱引用中获取缓存
        EngineResource<?> active = loadFromActiveResources(key, isMemoryCacheable);
        if (active != null) {
            //从缓存中获取资源成功
            cb.onResourceReady(active);
            if (Log.isLoggable(TAG, Log.VERBOSE)) {
                logWithTimeAndKey("Loaded resource from active resources", startTime, key);
            }
            return null;
        }

        //开启线程从网络中加载图片......
        EngineJob current = jobs.get(key);
        if (current != null) {
            current.addCallback(cb);
            if (Log.isLoggable(TAG, Log.VERBOSE)) {
                logWithTimeAndKey("Added to existing load", startTime, key);
            }
            return new LoadStatus(cb, current);
        }

        EngineJob engineJob = engineJobFactory.build(key, isMemoryCacheable);
        DecodeJob<T, Z, R> decodeJob = new DecodeJob<T, Z, R>(key, width, height, fetcher, loadProvider, transformation,
                transcoder, diskCacheProvider, diskCacheStrategy, priority);
        EngineRunnable runnable = new EngineRunnable(engineJob, decodeJob, priority);
        jobs.put(key, engineJob);
        engineJob.addCallback(cb);
        engineJob.start(runnable);

        if (Log.isLoggable(TAG, Log.VERBOSE)) {
            logWithTimeAndKey("Started new load", startTime, key);
        }
        return new LoadStatus(cb, engineJob);
    }
```

load方法位于 Engine 类中。load方法内部会从三个来源获取图片数据，我们最熟悉的就是`LruCache`了。如何获取数据过于复杂，这里就不再展开分析，我们这里主要关注图片数据获取到之后的操作。获取到图片数据之后，通过  `cb.onResourceReady(cached)` 来处理，我们来看一下这个回调的具体实现：
```java
@Override
    public void onResourceReady(Resource<?> resource) {
        if (resource == null) {
            onException(new Exception("Expected to receive a Resource<R> with an object of " + transcodeClass
                    + " inside, but instead got null."));
            return;
        }

        Object received = resource.get();
        if (received == null || !transcodeClass.isAssignableFrom(received.getClass())) {
            releaseResource(resource);
            onException(new Exception("Expected to receive an object of " + transcodeClass
                    + " but instead got " + (received != null ? received.getClass() : "") + "{" + received + "}"
                    + " inside Resource{" + resource + "}."
                    + (received != null ? "" : " "
                        + "To indicate failure return a null Resource object, "
                        + "rather than a Resource object containing null data.")
            ));
            return;
        }

        if (!canSetResource()) {
            releaseResource(resource);
            // We can't set the status to complete before asking canSetResource().
            status = Status.COMPLETE;
            return;
        }
        
        //核心是这一句
        onResourceReady(resource, (R) received);
    }
```
我们继续看`onResourceReady(resource, (R) received)`这个方法：
```java
 private void onResourceReady(Resource<?> resource, R result) {
        // We must call isFirstReadyResource before setting status.
        boolean isFirstResource = isFirstReadyResource();
        status = Status.COMPLETE;
        this.resource = resource;

        if (requestListener == null || !requestListener.onResourceReady(result, model, target, loadedFromMemoryCache,
                isFirstResource)) {
            GlideAnimation<R> animation = animationFactory.build(loadedFromMemoryCache, isFirstResource);
            
            //核心，通过调用target的onResourceReady方法加载图片
            target.onResourceReady(result, animation);
        }

        notifyLoadSuccess();

        if (Log.isLoggable(TAG, Log.VERBOSE)) {
            logV("Resource ready in " + LogTime.getElapsedMillis(startTime) + " size: "
                    + (resource.getSize() * TO_MEGABYTE) + " fromCache: " + loadedFromMemoryCache);
        }
    }
```
我们可以看到核心代码：`target.onResourceReady(result, animation)`，其实在这句代码的内部最终是通过：
```java
public class DrawableImageViewTarget extends ImageViewTarget<Drawable> {
    public DrawableImageViewTarget(ImageView view) {
        super(view);
    }

    @Override
    protected void setResource(Drawable resource) {
       view.setImageDrawable(resource);
    }
}
```
本质是通过 `setResource(Drawable resource) `来实现的，在这个方法的内部调用了Android内部最常用的加载图片的方法 view.setImageDrawable(resource) 。

#### into方法总结
- 将imageview包装成imageViewTarget
- 清除这个imageViewTarget之前绑定的请求，绑定新的请求
- 执行新的请求
- 获取图片数据之后，成功则会调用ImageViewTarget中的onResourceReady()方法，失败则会调用ImageViewTarget中的onLoadFailed();二者的本质都是通过调用Android中的imageView.setImageDrawable(drawable)来实现对imageView的图片加载

---
### LruCache源码分析
LruCache来自于 `android.support.v4.util` 中：
```java
public class LruCache<K, V> {
    //存储缓存
    private final LinkedHashMap<K, V> map;
    //当前缓存的总大小
    private int size;
    //最大缓存大小
    private int maxSize;
    //添加到缓存的个数
    private int putCount;
    //创建的个数
    private int createCount;
    //移除的个数
    private int evictionCount;
    //命中个数
    private int hitCount;
    //未命中个数
    private int missCount;

    public LruCache(int maxSize) {
        if (maxSize <= 0) {
            throw new IllegalArgumentException("maxSize <= 0");
        }
        this.maxSize = maxSize;
        this.map = new LinkedHashMap<K, V>(0, 0.75f, true);
    }

    //重新设置最大缓存
    public void resize(int maxSize) {
        //确保最大缓存大于0
        if (maxSize <= 0) {
            throw new IllegalArgumentException("maxSize <= 0");
        }

        synchronized (this) {
            this.maxSize = maxSize;
        }
        //对当前的缓存做一些操作以适应新的最大缓存大小
        trimToSize(maxSize);
    }

    //获取缓存
    public final V get(K key) {
        //确保key不为null
        if (key == null) {
            throw new NullPointerException("key == null");
        }

        V mapValue;
        synchronized (this) {
            mapValue = map.get(key);
            //如果可以获取key对应的value
            if (mapValue != null) {
                //命中数加一
                hitCount++;
                return mapValue;
            }
            //如果根据key获取的value为null，未命中数加一
            missCount++;
        }
        
        //省略无关代码......
    }

    
    public final V put(K key, V value) {
        if (key == null || value == null) {
            throw new NullPointerException("key == null || value == null");
        }

        V previous;
        synchronized (this) {
            //添加到缓存的个数加一
            putCount++;
            //更新当前缓存大小
            size += safeSizeOf(key, value);
            previous = map.put(key, value);
            if (previous != null) {
                //如果之前map中对应key存在value不为null，由于重复的key新添加的value会覆盖上一个value，所以当前缓存大小应该再减去之前value的大小
                size -= safeSizeOf(key, previous);
            }
        }
        //根据缓存最大值调整缓存
        trimToSize(maxSize);
        return previous;
    }

    //根据最大缓存大小对map中的缓存做调整
    public void trimToSize(int maxSize) {
        while (true) {
            K key;
            V value;
            synchronized (this) {
                if (size < 0 || (map.isEmpty() && size != 0)) {
                    throw new IllegalStateException(getClass().getName()
                            + ".sizeOf() is reporting inconsistent results!");
                }
                //当前缓存大小小于最大缓存，或LinkedHashMap为空时跳出循环
                if (size <= maxSize || map.isEmpty()) {
                    break;
                }
                
                //遍历LinkedHashMap,删除顶部的（也就是最先添加的）元素，直到当前缓存大小小于最大缓存，或LinkedHashMap为空
                Map.Entry<K, V> toEvict = map.entrySet().iterator().next();
                key = toEvict.getKey();
                value = toEvict.getValue();
                map.remove(key);
                size -= safeSizeOf(key, value);
                evictionCount++;
            }

          //省略无关代码......
        }
    }

    public final V remove(K key) {
        if (key == null) {
            throw new NullPointerException("key == null");
        }

        V previous;
        synchronized (this) {
            previous = map.remove(key);
            if (previous != null) {
                size -= safeSizeOf(key, previous);
            }
        }

        return previous;
    }

    private int safeSizeOf(K key, V value) {
        int result = sizeOf(key, value);
        if (result < 0) {
            throw new IllegalStateException("Negative size: " + key + "=" + value);
        }
        return result;
    }

     protected int sizeOf(K key, V value) {
        return 1;
    }


    public final void evictAll() {
        trimToSize(-1); // -1 will evict 0-sized elements
    }

    @Override public synchronized final String toString() {
        int accesses = hitCount + missCount;
        int hitPercent = accesses != 0 ? (100 * hitCount / accesses) : 0;
        return String.format(Locale.US, "LruCache[maxSize=%d,hits=%d,misses=%d,hitRate=%d%%]",
                maxSize, hitCount, missCount, hitPercent);
    }
}
```

其实从上面的代码可以看出，LruCache内部主要靠一个`LinkedHashMap`来存储缓存，这里使用LinkedHashMap而不使用普通的HashMap正是看中了它的顺序性，即`LinkedHashMap中元素的存储顺序就是我们存入的顺序`，而HashMap则无法保证这一点。

我们都知道Lru算法就是最近最少使用的算法（least recently used），而LruCache是如何保证在缓存大于最大缓存大小之后移除的就是最近最少使用的元素呢？关键在于 `trimToSize(int maxSize)` 这个方法内部，在它的内部开启了一个循环，遍历LinkedHashMap，删除顶部的（也就是最先添加的）元素，直到当前缓存大小小于最大缓存，或LinkedHashMap为空。这里需要注意的是由于LinkedHashMap的特点，它的存储顺序就是存放的顺序，所以位于顶部的元素就是最近最少使用的元素，正是由于这个特点，从而实现了当缓存不足时优先删除最近最少使用的元素。
```java
//根据最大缓存大小对map中的缓存做调整
    public void trimToSize(int maxSize) {
        while (true) {
            K key;
            V value;
            synchronized (this) {
                if (size < 0 || (map.isEmpty() && size != 0)) {
                    throw new IllegalStateException(getClass().getName()
                            + ".sizeOf() is reporting inconsistent results!");
                }
                //当前缓存大小小于最大缓存，或LinkedHashMap为空时跳出循环
                if (size <= maxSize || map.isEmpty()) {
                    break;
                }
                
                //遍历LinkedHashMap,删除顶部的（也就是最先添加的）元素，直到当前缓存大小小于最大缓存，或LinkedHashMap为空
                Map.Entry<K, V> toEvict = map.entrySet().iterator().next();
                key = toEvict.getKey();
                value = toEvict.getValue();
                map.remove(key);
                size -= safeSizeOf(key, value);
                evictionCount++;
            }

          //省略无关代码......
        }
    }
```



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

[带你全面了解Glide 4的用法](https://blog.csdn.net/guolin_blog/article/details/78582548)

[Glide 4.0.0 RC0 官方说明](https://juejin.im/entry/5924f28eda2f60005d7725b2)