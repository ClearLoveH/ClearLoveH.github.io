---
layot:     post
title:      "网络与web服务"
subtitle:   "WEB API"
date:       2018-12-9 11:05
author:     "Heng"
header-img: "img/弗雷尔卓德2.jpg"
catalog: true
tags:
    - Android
---

>广东终于入冬了.

---

[项目仓库](https://gitee.com/ClearLoveH/PersonalProject5)

---


## 一、实验题目 

### WEB API

---

## 二、实现内容

- 实验内容：`实现一个bilibili的用户视频信息获取软件`
    - 搜索框只允许正整数int类型，不符合的需要弹Toast提示
    - 当手机处于飞行模式或关闭wifi和移动数据的网络连接时，需要弹Toast提示
    - 由于bilibili的API返回状态有很多，这次我们特别的限制在以下几点
        - 基础信息API接口为： https://space.bilibili.com/ajax/top/showTop?mid=<user_id>
        - 图片信息API接口为基础信息API返回的URL，cover字段
        - 只针对前40的用户id进行处理，即user_id <= 40
        - [2,7,10,19,20,24,32]都存在数据，需要正确显示
    - 在图片加载出来前需要有一个加载条，不要求与加载进度同步
    - 布局和样式没有强制要求，只需要展示图片/播放数/评论/时长/创建时间/标题/简介的内容即可，可以自由发挥
    - 布局需要使用到CardView和RecyclerView
    - 每个item最少使用2个CardView，布局怎样好看可以自由发挥，不发挥也行
    - 输入框以及按钮需要一直处于顶部

- 实验目的
    - 学会使用HttpURLConnection请求访问Web服务
    - 学习Android线程机制，学会线程更新UI
    - 学会解析JSON数据
    - 学习CardView布局技术

---

## 三、课堂实验结果
### (1)实验截图

- 打开程序主页面
    ![](/img/in-post/post-Android/lab5/截图/0.png)
- 输入用户id，要求正整数int类型，不满足的弹Toast提示即可
    ![](/img/in-post/post-Android/lab5/截图/8.png)
- 输入用户id，点击搜索，网络没打开则弹Toast提示网络连接失败（打开手机飞行模式）	
    ![](/img/in-post/post-Android/lab5/截图/7.png)
- 网络打开情况下，输入用户id，不存在相应数据的弹Toast提示
    ![](/img/in-post/post-Android/lab5/截图/9.png)
- 输入用户id = 2，点击搜索，展示图片/播放数/评论/时长/创建时间/标题/简介内容	
    - 加载过程需有progress bar：
        ![](/img/in-post/post-Android/lab5/截图/loading.png)
        ![](/img/in-post/post-Android/lab5/截图/1.png)
- 再次输入用户id = 7，接着上次结果继续展示以上内容
    ![](/img/in-post/post-Android/lab5/截图/2.png)
    ![](/img/in-post/post-Android/lab5/截图/3.png)
    ![](/img/in-post/post-Android/lab5/截图/4.png)

- 重复搜索已经搜过的用户id，弹出toast提示
    ![](/img/in-post/post-Android/lab5/截图/5.png)

### 加分项：根据seek bar完成预览功能
![](/img/in-post/post-Android/lab5/截图/10.png)
![](/img/in-post/post-Android/lab5/截图/11.png)
- 针对用户id为32的情况特殊化处理——隐藏seek bar
    ![](/img/in-post/post-Android/lab5/截图/12.png)
- 拖动前后均显示原图片
    ![](/img/in-post/post-Android/lab5/截图/13.png)


### (2)实验步骤以及关键代码

1. 输入框输入内容限制为正整数：使用正则表达式`Pattern与Matcher`类。
    ```java
    String input = editText.getText().toString();
    Pattern p = Pattern.compile("[1-9]*");
    Matcher m = p.matcher(input);
    if(!m.matches() || input.equals("")){
        Toast.makeText(getApplicationContext(),"Please input positive integers.",Toast.LENGTH_SHORT).show();
    }
    else {
        requestBilibili(input);
    }
    ```
    
2. `（本次实验主要步骤）`——使用HttpURLConnection请求访问bilibili的API，来获取用户视频信息。
    - 这一步为本次实验主要的步骤，主要实现通过HttpURLConnection访问bilibili的API，并把通过get方法请求下来的数据转化成`jsonObject`，再转化成定义好的`recyclerObj`类，从而实现获取用户视频信息，并且把信息保存为我们所可以直接使用的格式。
    - 新建线程来发起对API的请求，在线程中进行网络访问，从而防止阻塞主线程。
        ```java
        private void requestBilibili(final String user_id) {
            Log.i("Tag", "wrong: start" );
            new Thread() {
                @Override
                public void run() {
                    HttpURLConnection connection = null;
                    try {
                        String urlstr="https://space.bilibili.com/ajax/top/showTop?mid=";
                        urlstr += user_id;
                        URL url = new URL(urlstr);
                        connection = (HttpURLConnection) url.openConnection();
                        connection.setRequestMethod("GET");
                        connection.setConnectTimeout(5000);
                        connection.setReadTimeout(5000);
                        connection.setRequestProperty("charset", "UTF-8");
                        ···
                        ···
        }
        ```
    - 网络访问获取下来的数据是以流的形式暂存在connection中的，我们需要把inputStream转换成字符串的形式，再转化成json就比较简单了。
        ```java
        if (connection.getResponseCode() == 200) {
            Log.i("Tag", "request code 200" );
            //把get获取的数据转换成json
            InputStream inputStream = connection.getInputStream();

            InputStreamReader inputStreamReader=new InputStreamReader(inputStream,"UTF-8");
            BufferedReader bufferedReader = new BufferedReader(inputStreamReader);
            StringBuffer stringBuffer = new StringBuffer();
            String temp;
            while ((temp=bufferedReader.readLine())!=null){
                stringBuffer.append(temp);
            }
            bufferedReader.close();
            inputStreamReader.close();
            inputStream.close();

            //把获取的数据流写入stringBuffer之后，创建对应的jsonObject
            String jsonStr = stringBuffer.toString();
            ···
            ···
        }
        ```
    - 得到string之后，我们可以通过java中Gson的方法，可以直接通过jsonString得到对应的json：
        ```java
        RecyclerObj recyclerObj = new Gson().fromJson(jsonStr, RecyclerObj.class);
        ```
    - 但是在此之前，我们先判断一下是否获取到了有用的json，而是否获取到正确的数据，本次bilibili给定的API接口返回的json中，给我们一个Boolean变量——`status`，status为false即为没有正确获取到视频信息，至于原因就返回在"data"中，所以我们需要先处理一遍json，先提取返回的json中的status值，判断为true，才把整个jsonString提供给handler来进行UI的更新（此时json中的data为又一个json，提交给handler来处理即可）；若status为false，则提供给handler错误信息（json中的data此时是一个错误信息的字符串）。
    - 一个注意点就是，Toast是不能在Thread中使用的，因为子线程没有创建Looper.prepare()，`一个Toast的创建需要依赖Handler的`。这种问题可以手动在子线程中添加当前Looper.prepare()，但是这样做会使子线程因为looper的存在，导致一直处于未销毁状态，而占有内存，是不好的做法，所以还是给handler来处理。
        ```java
        JSONObject jsonObject = new JSONObject(jsonStr);
        boolean status = jsonObject.getBoolean("status");
        if(!status){
            Log.i("Tag", "message" );
            String data = jsonObject.getString("data");
            //通过handler传递接收到的json
            Bundle b = new Bundle();
            b.putString("wrong", data);
            Message m = new Message();
            m.setData(b);
            m.what = 1;
            handler.sendMessage(m);
        }
        else {
            Bundle b = new Bundle();
            b.putString("json", jsonStr);
            Message m = new Message();
            m.setData(b);
            m.what = 1;
            handler.sendMessage(m);
        }
    - Handler处理上一部分传来的信息，通过`RecyclerObj recyclerObj = new Gson().fromJson(jsonStr, RecyclerObj.class)`，把string转化成的json再转化成我定义好的recyclerObj类，添加到list中，同时判断当前的视频信息是否已经被添加到list中。
        ```java
        private Handler handler = new Handler(){
        public void handleMessage(Message msg) {
            switch (msg.what){
                case 1:
                    Bundle b = msg.getData();
                    String wrong = b.getString("wrong");
                    String jsonStr = null;
                    if(wrong != null){
                        Toast.makeText(getApplicationContext(),wrong,Toast.LENGTH_SHORT).show();
                    }
                    else {
                        jsonStr = b.getString("json");
                        ···
                        ···
                        ···
                        Boolean flag = true;
                        //这里判断当前的视频信息是否已经被添加到list中
                        if(data != null){
                            RecyclerObj recyclerObj = new Gson().fromJson(jsonStr, RecyclerObj.class);
                            for (RecyclerObj temp : mDataList){
                                if(temp.getData().getAid() == recyclerObj.getData().getAid())
                                    flag = false;
                            }
                            if(flag){
                                mDataList.add(recyclerObj);
                                updateList();
                                state = 1;
                                //此处调用获取预览图的函数
                                getPreviewImage(recyclerObj.getData().getAid());
                            }
                            else Toast.makeText(getApplicationContext(),"You have searched the user, no need to repeat.",Toast.LENGTH_SHORT).show();
                        }
                    ···
                    ···
        ```        

3. 第二部分我们完成了从给出的API接口中获取需要的json并解析成我所预先定义好的类，接下来便是通过list完成在adapter中，实现cardview所有该显示的部分的数据填充。由于json中`cover`给的是一个url，而不是直接可用的bitmap，所以我们在adapter中又需要开启新线程来完成图片的下载，再交由handler更新UI。
    - Thread：
        ```java
        //新建线程下载图片
        new Thread() {
            @Override
            public void run() {
                try {
                    URL url = new URL(mDataList.get(position).getData().getCover());
                    //直接将url.openStream放入decodeStream解析成bitmap
                    Bitmap cover = BitmapFactory.decodeStream(url.openStream());
                    Log.i("Background","download image");
                    //把bitmap与holder一并传给handler
                    bimapHolder temp = new bimapHolder(cover, holder, position);
                    Message msg=Message.obtain();
                    msg.what=1;
                    msg.obj=temp;
                    handler.sendMessage(msg);
                ···
                ···
        ```
    - 这一部分有一个很关键的点就是，Handler因为需要处理UI的更新，而这又是在adapter中，我们需要确定当前的编辑的是第几个itemView，所以我在这里就把viewHolder、下载好的bitmap和当前item在viewHolder中处于的位置position，一并传给handler，从而实现针对性的UI更新。为了实现这个目的，我创建了一个新的类来存储这三个重要信息。
        ```java
        //此类用于handler将获取的bitmap设置在对应的ImageView上，所以需要传递一个ViewHolder参数
        private class bitmapHolder {
            public Bitmap bmp;
            public ViewHolder holder;
            public int position;
            public bitmapHolder(Bitmap b, ViewHolder h,int p) {
                bmp = b;
                holder = h;
                position = p;
            }
        }
        ```
    - handler：handler部分主要注意progress bar该什么时候消失，而视频所有的信息该什么时候显示。此部分代码不贴出来了。


4. 当手机处于飞行模式或关闭wifi和移动数据的网络连接时，我们需要给出Toast，实现思路很简单，在connection建立的地方，添加一个catch来捕获访问网络超时产生的异常——`UnknownHostException`，捕获到该异常后，发给handler消息，让其于UI线程中弹出toast提示。   
    ```java
    catch (UnknownHostException e){
        Message m = new Message();
        m.obj = "网络连接失败";
        m.what = -1;
        handler.sendMessage(m);
        e.printStackTrace();
    } catch (IOException e) {
        ···
        ···
    ```


### `加分项：`
- 本次加分项的内容实现较多，但是基本思路还是与上面的部分思路相同的，我们把获得的json中的`aid`值，作为加分项继续操作的入口。
- 以aid为始，我们需要通过另一个接口 `https://api.bilibili.com/pvideo?aid=<aid>` 来获取加分项需要的信息，通过postman我们可以先了解到返回的json有什么内容，再依此又建立一个新的存储json中我们所需要的信息的类 `PreviewImagesObj`。先附上postman获取的json结果：
    ![](/img/in-post/post-Android/lab5/截图/14.png)
    - 对应创建的 PreviewImagesObj 类：
        ```java
        public class PreviewImagesObj {
        private String[] image;
        private int[] index;
        private int img_x_len;
        private int img_y_len;
        private int img_x_size;
        private int img_y_size;

        public int getCount(){
            Log.i("count",""+(index.length-1));
            return index.length - 1;
        }
        ···
        ···
        ```
    - 从json中可以看出，我们获取的第一个json中，`image[]`即我们所需要的图片，但是打开这个链接可以看到是一堆图片合成的一个大图，所以我们还需要进行切割。
- 总结一下，我们需要实现滑动seek bar显示对应的预览图，我们需要大致四步：`获取并解析json`、`从json中链接下载图片`、`对下载的图片进行切割`，`通过seek bar的滑动事件监听器更新UI`。
- 第一二三步的实现，我分别实现了两个函数 `getPreviewImage` 和 `getPreviewImage2`，开启了两个新线程来分别处理json的获取与解析，大图的下载，而后进行切割，都处理完之后，通知handler更新adapter。
    ```java
    private void getPreviewImage(int aid){
        final String urlstr = "https://api.bilibili.com/pvideo?aid=" + aid;
        new Thread() {
            @Override
            public void run() {
                HttpURLConnection connection = null;
                try {
                    URL url = new URL(urlstr);
                    connection = (HttpURLConnection) url.openConnection();
                    ···
                    ···
                    if (connection.getResponseCode() == 200) {
                        Log.i("Tag", "request code 200" );
                        //把get获取的数据转换成json
                        ······

                        //把获取的数据流写入stringBuffer之后，创建对应的jsonObject
                        String jsonStr = stringBuffer.toString();
                        JSONObject jsonObject = new JSONObject(jsonStr);

                        String data = jsonObject.getString("data");
                        PreviewImagesObj previewImagesObj = new Gson().fromJson(data, PreviewImagesObj.class);

                        //执行第二步网络请求
                        getPreviewImage2(previewImagesObj);
                    }
                ···
                ···
    ```
    ```java
    private void getPreviewImage2(PreviewImagesObj previewImagesObj){
        final String imageUrl = previewImagesObj.getImage()[0];
        final int count = previewImagesObj.getCount();
        final int img_x_len = previewImagesObj.getImg_x_len();
        final int img_y_len = previewImagesObj.getImg_y_len();
        final int img_x_size = previewImagesObj.getImg_x_size();
        final int img_y_size = previewImagesObj.getImg_y_size();
        //新线程用于下载缩略图
        new Thread() {
            @Override
            public void run() {
                try {
                    URL url = new URL(imageUrl);
                    //直接将url.openStream放入decodeStream解析成bitmap
                    Bitmap bitmap = BitmapFactory.decodeStream(url.openStream());
                    List<Bitmap> images = new ArrayList<>();
                    //图片切割
                    for (int i = 0; i < img_y_len; i++) {
                        for (int j = 0; j < img_x_len; j++) {
                            int xValue = j * img_x_size;
                            int yValue = i * img_y_size;
                            Bitmap temp = Bitmap.createBitmap(bitmap, xValue, yValue, img_x_size, img_y_size);
                            if( i*10 + (j+1) >= count)
                                break;
                            else images.add(temp);
                        }
                        if((i+1)*10 > count)
                            break;
                    }
                    previewImagesList.add(images);
                    state = 2;
                    Message m = new Message();
                    m.obj = "update adapter";
                    m.what = 6;
                    handler.sendMessage(m);
                ···
                ···
    ```
- 因为多了一个` List<List<Bitmap>> previewImagesList`来存取切割完的图片组，而我们使用还是要在adapter中，所以这里我们需要修改adapter，构造函数添加一个 List<List<Bitmap>> 参数，从而把切割完图片组传进adapter使用，注意这部分还需要在seekbar的changelistener中实现拖动进度条前后都回到cover的图片，只需在上面下载封面图的时候，我们用一个List<Bitmap>来存储所有的cover即可。
    ```java
    holder.seekBar.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
            @Override
            public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
                //设置seekbar最大值
                holder.seekBar.setMax(previewImagesList.get(position).size());
                if(progress >0 && fromUser)
                    holder.cover.setImageBitmap(previewImagesList.get(position).get(progress - 1));
                else holder.cover.setImageBitmap(covers.get(position));
            }
            ···
            ···
            //放开进度条时要回到cover
            @Override
            public void onStopTrackingTouch(SeekBar seekBar) {
                holder.cover.setImageBitmap(covers.get(position));
                holder.seekBar.setProgress(0);
            }
        });
    ```
- 因为后台多了需要处理的东西（下载新的json，再下载大图片），而这些操作相对第一个仅仅是封面的下载，耗时要多上不少，所以progress bar消失及视频信息内容显示的时机，我们也要进行更改，不然会出现一个问题——`拖动seek bar时，可能图片还没下载并切割好，从而导致应用闪退。`所以我在MainActivity中添加一个int变量`state——0 for no recyclerObj;1 for finishing downloading cover ; 2 for finish downloading preview images`。用于表示MianActivity中处理到什么样的进度了，从而让adapter根据MainActivity的状态state来决定progress bar消失及视频信息内容显示的时机，这样就避免了“手速大于电脑下载速度”的问题了，程序逻辑上也十分合理，progress bar的意义也体现出来。

### 于此，本次实验的基础任务及加分项都基本完成了。

----------

### (3)实验遇到的困难以及解决思路

这次WEB API实验遇到的主要问题有四个！！！

- `Cleartext HTTP traffic to i0.hdslb.com not permitted`
    - 问题来源：Android P http网络请求的问题：
        - Google表示，为保证用户数据和设备的安全，针对下一代 Android 系统(Android P) 的应用程序，将要求默认使用加密连接，这意味着 Android P 将禁止 App 使用所有未加密的连接，因此运行 Android P 系统的安卓设备无论是接收或者发送流量，未来都不能明码传输，需要使用下一代(Transport Layer Security)传输层安全协议，而 Android Nougat 和 Oreo 则不受影响。
        - 因此在Android P 使用HttpUrlConnection进行http请求会出现以下异常
    - 解决思路有三：
        - APP改用https请求
        - targetSdkVersion 降到27以下
        - 更改网络安全配置
        - 我使用的是第三种解决方法：
            - 创建一个network_security_config.xml文件      
                ```xml
                <?xml version="1.0" encoding="utf-8"?>
                <network-security-config>
                    <base-config cleartextTrafficPermitted="true" />
                </network-security-config>
                ```
            - 在AndroidManifest.xml文件下的application标签增加以下属性：`android:networkSecurityConfig="@xml/network_security_config"`

- `android.content.res.Resources$NotFoundException:`
    - 这个报错是在使用json转化给RecyclerObj类时发生的：

        `RecyclerObj recyclerObj = new Gson().fromJson(jsonStr, RecyclerObj.class);`

        检查了一下之后是数据类型没有匹配上的问题，int与String类型不能直接匹配。


- 在Coordinatorlayout中使用RecyclerView导致recyclerview最后一个item显示不全的问题，这个问题我在之前的布局也遇到过，解决办法很简单——把recycleview的高度设置成一个具体的高度值，这样是可以完全显示的，不能使用match_parent或wrap_content。


- 第四个问题，如图：
    ![](/img/in-post/post-Android/lab5/截图/gl_error.png)
    - 这个问题导致我应用启动时一直白屏进不去
    - 原因：经查证，这种问题的出现又很多的原因，但是解决方法都是一样的，就是和android硬件加速器有关，而确定有用的一种解决方法就是关闭硬件加速，但是这样的方式也是没有针对性的。
    - 解决方式：经过我个人测试，我似乎没有把打开的connection关闭掉，添加了下面的代码之后就没有问题了；同时对于我sdk为25的另一台虚拟机并没有这样的问题，即使没关闭connection，所以以后遇到这种问题，直接关闭硬件加速是肯定ok的。
        ```java
        finally {
            if(connection != null)
                connection.disconnect();
        }
        ```

----

## 四、实验思考及感想

实验中的思考与感悟：
- 本次实验基础任务是基本网络访问API接口，综合postman的使用，其实本次实验上起手来是很快的。主要的感触：
    - 这次实验之后学会了使用 HttpURLConnection 请求访问Web服务，之前接触过OKHttp，这次的HttpURLConnection也是比较基础的了。
    - cardview的第一次接触之后也是感觉挺好用的，这种日常用的许多app都会使用的布局，简介美观，在以后的编程中也是很容易使用到的。
    - java提供了那么多json解析的方式给我们，确实是非常方便处理json数据的，此次实验之前我对json数据还是比较陌生的，但是以后遇到json，我可以很轻松的用java来解析了。
    - 这一次实验特别是加分项让我对java多线程部分的认识又进一步加强了，这次实验没有使用上次学会的Rxjava，主要是我认为这次任务较重，所以还是使用了相对简单明了的handler来进行UI的更新，其实差别不大，主要还是要认识到他们存在的目的，就是让我们知道，`UI的更新一定要在UI线程上完成`，包括Toast也是如此。子线程只负责在后台做那些耗时操作，在做多线程的时候定要分清那些工作是只能在主线程完成的，那些工作是需要子线程来完成，下一次就可以体验尝试一下`异步任务类AnsycTask`来进行多线程的任务。
- 这次实验报告内容较多，也可以看出我的收获是很多的，还有最后几个项目，需要学的东西还有很多，继续加油吧。

---

