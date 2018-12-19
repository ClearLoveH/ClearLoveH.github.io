---
layot:     post
title:      "网络与web服务"
subtitle:   "Retrofit2与RxJava、OKHttp"
date:       2018-12-16 12:55
author:     "Heng"
header-img: "img/弗雷尔卓德2.jpg"
catalog: true
tags:
    - Android
---

>这个期末过的是真的累。

---

[项目仓库](https://gitee.com/ClearLoveH/PersonalProject5)

---


## 一、实验题目 

## WEB API——Github

---

## 二、实现内容

- 第十五周实验目的
    - 理解Restful接口
    - 学会使用`Retrofit2`
    - 复习使用RxJava
    - 学会使用`OkHttp`

- 实验内容：`实现一个github用户repos以及issues应用`
    - 主界面有两个跳转按钮分别对应两次作业	
    - github界面，输入用户名搜索该用户所有可提交issue的repo，每个item可点击
    - repo详情界面，显示该repo所有的issues	
 
    - 每次点击搜索按钮都会清空上次搜索结果再进行新一轮的搜索
    - 获取repos时需要处理以下异常：HTTP 404 以及 用户没有任何repo
    - 只显示 has_issues = true 的repo（即fork的他人的项目不会显示）
    - repo显示的样式自由发挥，显示的内容可以自由增加（不能减少）
    - repo的item可以点击跳转至下一界面
    - 该repo不存在任何issue时需要弹Toast提示

- `加分项`
   - 加分项：在该用户的该repo下增加一条issue，输入title和body即可

---

## 三、课堂实验结果
### (1)实验截图

- 打开程序主页面
    ![](/img/in-post/post-Android/lab5/截图/week15_7.png)

- 进入Gayhub页面
    ![](/img/in-post/post-Android/lab5/截图/week15_8.png)

- 输入一个不存在的用户，获取404码并提示错误
    ![](/img/in-post/post-Android/lab5/截图/week15_9.png)

- 输入我的id，进入我的Gayhub页面
    ![](/img/in-post/post-Android/lab5/截图/week15_10.png)

- 点击进入一个没有issue的repo，toast提示
    ![](/img/in-post/post-Android/lab5/截图/week15_11.png)

- 进入进入一个有issue的repo，显示所有issues
    ![](/img/in-post/post-Android/lab5/截图/week15_12.png)

#### 加分项：在该用户的该repo下增加一条issue，输入title和body即可

- 新建issue（token在后台已经设置好，此处无输入）
    ![](/img/in-post/post-Android/lab5/截图/week15_13.png)
    ![](/img/in-post/post-Android/lab5/截图/week15_14.png)

- 页面立即刷新，在最下面找到刚刚新建的issue
    ![](/img/in-post/post-Android/lab5/截图/week15_15.png)
    
- 在我的GitHub页面上查看该仓库信息
    ![](/img/in-post/post-Android/lab5/截图/week15_17.png)



### (2)实验步骤以及关键代码

1. 第一步先完成LAUNCHER页面的设计，就两个按钮分别跳转不同的activity，然后再manifest文件中修改`android.intent.category.LAUNCHER`指定的界面即可。

2. 第二步，本次实验第一个关键步骤，`通过api获取user的所有repo`
    - 因为要综合Rxjava和retrofit一起使用，先进行retrofit的基本设置，主要操作分为五部曲。
        1. OkHttp，承载Retrofit的请求
            ```java
            OkHttpClient build = new OkHttpClient.Builder()
                .connectTimeout(3, TimeUnit.SECONDS)
                .readTimeout(3, TimeUnit.SECONDS)
                .writeTimeout(3, TimeUnit.SECONDS)
                .build();
            ```
        2. 构造Retrofit对象访问
            ```java
             Retrofit retrofit = new Retrofit.Builder()
                .baseUrl(baseURL)
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                .client(build)
                .build();
            ```
        3. 构造API访问接口 Interface
            ```java
                // 不是Class 而是 interface
                public interface GitHubService {
                    @GET("/users/{user_name}/repos")
                    Observable<List<Repo>> getRepo(@Path("user_name") String user_name);
                }
            ```
        4. 调用前面定义的 interface
            ```java
            GitHubService service = retrofit.create(GitHubService.class);
            ```
        5. 使用Rxjava——创建observer，用于进行请求完成后在主线程的更新UI（在OnNext()函数中进行）。
            - 更新UI时，同时判断该用户是否没有任何repo，一个repo都没有则进行toast提示；
            - 若repo数不为空，则把adapter绑定的list（用于存储repo的list） clear，然后把传给OnNext()函数的repo list所有的repo添加进adapter绑定的list中，在通过adapter更新UI。
            ```java
            final Observer<List<Repo>> observer = new Observer<List<Repo>>() {
                @Override
                public void onNext(List<Repo> o) {
                    if(o == null){
                        Toast.makeText(getApplicationContext(),"User not found.",Toast.LENGTH_SHORT).show();
                        Log.i("Error", "no such user");
                    }
                    repos.clear();
                    Log.i("state", o.get(0).getName());
                    repos.addAll(o);
                    if(repos.size() == 0)
                        Toast.makeText(getApplicationContext(),"User don`t have any repo that has issues.",Toast.LENGTH_SHORT).show();
                    Log.i("state", repos.get(0).getName()+repos.get(0).getRepo_id()+repos.get(0).getOpen_issues()+repos.get(0).getDescription());
                    myAdapterForRepo.notifyDataSetChanged();
                    currentUser = editText.getText().toString();
                    editText.setText("");
                }
                ···
                ···
                ···
            ```

3. 本次实验第二个关键步骤，`处理搜索user时的404问题：`
    - 处理404，即user不存在的问题，首先我们先通过postman查看该情况下API返回给我们的json：
        ![](/img/in-post/post-Android/lab5/截图/week15_16.png)
    - 发现我们的repo类并不能处理此时返回的json，而在这种情况下，我们看到返回的json并没有显示的状态码给我们，看似没办法处理，其实在这种时候，Rxjava会统一的调用`onError`函数，而不是正常情况下的OnNext()函数，来接收throw的exception，不过我们需要对此时的Exception进行分类处理，对于404的情况，Rxjava接收到的Exception为`HttpException`，然后我们在OnError函数进行错误信息及错误状态码的提示即可，即成功处理搜索user时的404问题；还有其他的exception例如SSLHandshakeException我们也可以单独处理。
    ```java
     @Override
            public void onError(@NonNull Throwable e) {
                if(e instanceof HttpException){
                    //获取对应statusCode和Message
                    HttpException exception = (HttpException)e;
                    int code = exception.response().code();
                    Toast.makeText(getApplicationContext(),"User not found. State: " + code,Toast.LENGTH_SHORT).show();
                }else if(e instanceof SSLHandshakeException){
                    //接下来就是各种异常类型判断
                }
            }
    ```
4. 第三步，点击某个repo，进入仓库并显示该仓库有的issues，同样的，这部分的显示与上一部分的repo的获取并显示的操作是差不多的，只不过我们在从repos页面到issues页面时，需要把当前用户名及其repo的name作为参数传给issues页面，其余的除了interface调用的API不同之外，其他的流程是基本一样的。
    1. OkHttp，承载Retrofit的请求
        - 略
    2. 构造Retrofit对象访问
       - 略
    3. 构造API访问接口 Interface
        ```java
        public interface IssuesService {
            @GET("/repos/{user_name}/{repo_name}/issues")
            Observable<List<Issue>> getIssues(@Path("user_name") String user_name,@Path("repo_name") String repo_name);
            ···
            ···
        }
        ```
    4. 调用前面定义的 interface
       - 略
    5. 使用Rxjava——创建observer，用于进行请求完成后在主线程的更新UI（在OnNext()函数中进行），而且这步少去了判断404的操作，我们只需要添加一下判断当前repo有没有issues即可。
        ```java
        final Observer<List<Issue>> observer = new Observer<List<Issue>>() {
            @Override
            public void onNext(List<Issue> o) {
                Issues.clear();
                Issues.addAll(o);
                if(Issues.size() == 0)
                    Toast.makeText(getApplicationContext(),"This repo don`t have any issue.",Toast.LENGTH_SHORT).show();

                myAdapterForIssue.notifyDataSetChanged();
            }
            ······
            ······
        ```

#### `加分项：在该用户的该repo下增加一条issue`
- 加分项与前面的任务不同之处在于对于API的网络请求方法是不同的，前面的都是GET方法，现在要进行增的操作时，RESTful架构要求我们应使用POST方法：
    - GET - 从指定的资源请求数据。
    - POST - 向指定的资源提交要被处理的数据
- 而使用POST来实现issue的增加，相对前面的GET请求，实现比较复杂一些，我们需要弄懂我们要传的参数，首先增加一条issue，request的body需要的参数，由json得：
    ![](/img/in-post/post-Android/lab5/截图/week15_18.png)
- request的body需要传`title`及`body`两个参数，注意到我们在发送请求时，这两个变量是放在request的body中的，所有发送请求时的参数，我们也需要新建RequestBody来作为参数之一，`title`及`body`放入requestbody之中，不能把他们直接作为参数传过去，制作requestbody是`通过hashmap先转换成json的string格式，再生成对应的requestbody`：
    ```java
    Map<String, String> map = new HashMap<>();//body  map
    map.put("title", input2);
    map.put("body",input3);
    String json = new Gson().toJson(map);
    RequestBody requestBody = RequestBody.create(okhttp3.MediaType.parse("application/json; charset=utf-8"), json);
    ```
- 有了body，我们就可以传API所需要的json过去，但是对仓库的增删改操作是需求权限的，也就是需要认证，而我们采用的认证方式是通过token的方式来实现的，学习计网、做过计网实验以及服务计算的后端服务器设计之后，我对token的认证方式也有了足够的认识，这里的token是要再github中手动生成的，然后我们把这个token添加在Request的header中，只要token是当前访问的用户指定的token，我们正确的增删改请求便会被API所接受。（实验中我预先设置好了token，所有不用输入，提交时会删除我自己的token转而通过edittext来获取输入的token），不过在token的添加上也遇到了小问题，这部分我在后面遇到的问题提到。
    - 对应的interface设计:
        ```java
        @POST("/repos/{user_name}/{repo_name}/issues")
            Observable<Issue> addIssue(@Header("Authorization") String authorization,
                                    @Path("user_name") String user_name, @Path("repo_name") String repo_name,
                                    @Body RequestBody body);
        ```
- 通过postman返回给我们的json可以发现：
    ![](/img/in-post/post-Android/lab5/截图/week15_19.png)
    ![](/img/in-post/post-Android/lab5/截图/week15_20.png)
    - 增加issue成功时，api返回给我们的是一个生成的issue的所有信息，所以我们interface中的`addIssue`方法返回值设为了Observable<Issue>，通过observer，我们正好把返回的issue添加到issuesList中，通过adapter更新UI。
    - 在点击添加issue按钮时，我已经限定了需要输入title与body了，所以没有title与body的错误是被避免了，不过我还要处理token错误的问题，同样的，我们在`onError`中来处理这个异常。
        ```java
        final Observer<Issue> observerIssue = new Observer<Issue>() {
            @Override
            public void onNext(Issue o) {
                Log.i("state",""+ o.getTitle());
                Issues.add(o);
                if(o == null)
                    Toast.makeText(getApplicationContext(),"Create issue failed, need correct token.",Toast.LENGTH_SHORT).show();
                else Toast.makeText(getApplicationContext(),"Create issue successfully.",Toast.LENGTH_SHORT).show();
                Log.i("state", o.getTitle()+o.getCreated_at()+o.getState()+o.getBody());
                myAdapterForIssue.notifyDataSetChanged();
            }
            @Override
            public void onCompleted() {
            }
            @Override
            public void onError(@NonNull Throwable e) {
                if(e instanceof HttpException){
                    //获取对应statusCode和Message
                    HttpException exception = (HttpException)e;
                    int code = exception.response().code();
                    String msg = exception.response().message();
                    Toast.makeText(getApplicationContext(),msg + "\nState: " + code,Toast.LENGTH_SHORT).show();
                }
            }
        };
        ```
    - 处理结果：
        ![](/img/in-post/post-Android/lab5/截图/week15_21.png)
        ![](/img/in-post/post-Android/lab5/截图/week15_22.png)

- 至此，基本完成了本次实验的所有设计。
       
----------

### (3)实验遇到的困难以及解决思路

这次 `WEB API——gayhub` 实验遇到的主要问题有两个！！！


- `Accessing hidden field Lsun/misc/Unsafe;->theUnsafe:Lsun/misc/Unsafe; (light greylist, reflection)`
    ![](/img/in-post/post-Android/lab5/截图/week15_1.png)
    ![](/img/in-post/post-Android/lab5/截图/week15_2.png)
    - 问题情景：往github发送了多次test的post之后，就突然无法再向api继续发送post或get请求了，app界面卡住不动，出现如上报错。
    - 解决方法：我换了一个API更低的虚拟机就可以继续运行了，感觉这问题和tutorial中TA说的demo的问题是一样的，就是https的证书问题，阅读了相关资料之后，要彻底解决此问题，应该需要在APP内添加证书，步骤比较繁琐就没有去尝试，但是问题的原因算是成功知晓了。

    - 参考博客：
        - [Android Https相关完全解析 当OkHttp遇到Https](http://blog.csdn.net/lmj623565791/article/details/48129405)
        - [SSLSOcket在Android6.0中出错原因](https://blog.csdn.net/youngwm/article/details/50707533)
        - [Retrofit Https踩坑记录](https://www.jianshu.com/p/41bb549317ff)


- list的adapter不更新的问题：
    - 这个问题出现原因还是我思路出了问题，刷新repos的时候，其实只用把list clear一下即可，然后再添加所有get到的repos。而我new了一个新的ArrayList，查阅资料发现这样等于是改变了adapter原来指向的内存空间，所有是不会再根据你repo list的变化而通过notify函数自动更新UI了。


- `token的添加问题`
    - 一开始使用postman来尝试添加token时，按照我以前token的使用经验，发现API并没有响应我的请求：
        ![](/img/in-post/post-Android/lab5/截图/week15_24.png)
    - 后来在postman的authorization界面中设置如下：
        ![](/img/in-post/post-Android/lab5/截图/week15_25.png)
    - 在这里添加token，请求才能正确接收，我查看了这种情况下postman发出去的请求：
        ![](/img/in-post/post-Android/lab5/截图/week15_26.png)
    - 发现他在token前加了`Bearer `，这个请求才正确被接收，所以我在后端也为token加上了这个前缀，最终实现正常功能。
    
----

## 四、实验思考及感想

实验中的思考与感悟：
- 本次实验基础任务依旧是基本网络访问API接口，不过这次访问的是GitHub的API，而且访问API的方式与前一周也不一样：Retrofit2与Rxjava综合起来使用，让我们对网络访问的方式又多了几种掌握。
    - 相对本周，上周的HttpURLConnection访问API，就是简单的直接访问url，封装性好，简单易用，适用于轻量级的网络交互、网络请求频繁且传输数据量小的场景下使用。
    - 而Retrofit不仅性能好，处理效率高，而且解耦的更加彻底，职责更加细分，且可与其他框架结合使用（RxJava），更契合RESTful的API设计风格，且支持同步异步请求的方式，二者优劣对比可见。

- 而两个网站基于Restful的API设计风格，在本次实验通过Retrofit体现的更加明晰。
    - 通过构造的API访问接口Interface，于其中详细的定义网络请求的方式，高度的解耦，不仅使用起来明了简单，更让我们在debug时更容易找到出问题的模块。
    - 而这也启发了我们在设计软件的时候也一定要注意各模块之间一定尽可能的减少联系，防止一个模块出现的问题影响到其他模块。
- 而加分项对于POST请求的使用以及请求头部的设置方法的考察，也是非常实用，同时还考察了对token的使用，在token使用过程上遇到的问题也为我以后排好了坑，都是宝贵的项目经验，本次实验感触挺多，收获颇丰，也是这学期下来安卓的最后一次基础实验了，剩下的就是期末项目，一学期下来真的感触很深，以后在工作上安卓也成了我可以选择的一条路，继续加油吧。

---



