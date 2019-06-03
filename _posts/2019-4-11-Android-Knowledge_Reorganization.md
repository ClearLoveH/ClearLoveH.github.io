---
layout:     post
title:      "Android 知识点整理"
date:       2019-4-11 11:38
author:     "Heng"
header-img: "img/比尔吉沃特1.jpg"
catalog: true
tags:
    - Android
---

>还要更努力

---

### 红黑树——R-B Tree，全称是Red-Black Tree

红黑树的特性:
1. 每个节点或者是黑色，或者是红色。
2. 根节点是黑色。
3. 每个叶子节点（NIL）是黑色。 [注意：这里叶子节点，是指为空(NIL或NULL)的叶子节点！]
4. 如果一个节点是红色的，则它的子节点必须是黑色的。
5. 从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。

正是由于这些原因使得红黑树是一个`平衡二叉树`

**第一步: 将红黑树当作一颗二叉查找树，将节点插入。**

- 红黑树本身就是一颗二叉查找树，将节点插入后，该树仍然是一颗二叉查找树。也就意味着，树的键值仍然是有序的。此外，无论是左旋还是右旋，若旋转之前这棵树是二叉查找树，旋转之后它一定还是二叉查找树。这也就意味着，任何的旋转和重新着色操作，都不会改变它仍然是一颗二叉查找树的事实。

**第二步：将插入的节点着色为"红色"。**

- 将插入的节点着色为红色，不会违背"特性(5)"！

**第三步: 通过一系列的旋转或着色等操作，使之重新成为一颗红黑树。**

- 第二步中，将插入节点着色为"红色"之后，不会违背"特性(5)"。那它到底会违背哪些特性呢？
- 对于"特性(1)"，显然不会违背了。因为我们已经将它涂成红色了。
- 对于"特性(2)"，显然也不会违背。在第一步中，我们是将红黑树当作二叉查找树，然后执行的插入操作。而根据二叉查找数的特点，插入操作不会改变根节点。所以，根节点仍然是黑色。
- 对于"特性(3)"，显然不会违背了。这里的叶子节点是指的空叶子节点，插入非空节点并不会对它们造成影响。
- 对于"特性(4)"，是有可能违背的！
- 那接下来，想办法使之"满足特性(4)"，就可以将树重新构造成红黑树了。

---

### Android四大组件

**Activity 略**

**Service 略**

#### Content Provider
- `ContentProvider（内容提供者）`是Android中的四大组件之一。主要用于对外共享数据，也就是通过ContentProvider把应用中的数据共享给其他应用访问，其他应用可以通过ContentProvider对指定应用中的数据进行操作。ContentProvider分为系统的和自定义的，系统的也就是例如联系人，图片等数据。
- android中对数据操作包含有：`file, sqlite3, Preferences, ContectResolver与ContentProvider`。前三种数据操作方式都只是针对本应用内数据，程序不能通过这三种方法去操作别的应用内的数据。

- android中提供ContectResolver与ContentProvider来操作别的应用程序的数据。
    - 使用方式: 
        - 一个应用实现ContentProvider来提供内容给别的应用来操作，
        - 一个应用通过ContentResolver来操作别的应用数据，当然在自己的应用中也可以。

    - 以下这段是Google Doc中对ContentProvider的大致概述:
        - 内容提供者将一些特定的应用程序数据供给其它应用程序使用。内容提供者继承于ContentProvider 基类，为其它应用程序取用和存储它管理的数据实现了一套标准方法。然而，应用程序并不直接调用这些方法，而是使用一个 ContentResolver对象，调用它的方法作为替代。ContentResolver可以与任意内容提供者进行会话，与其合作来对所有相关交互通讯进行管理。

**ContentProvider**
- Android提供了一些主要数据类型的ContentProvider，比如音频、视频、图片和私人通讯录等。可在android.provider包下面找到一些Android提供的ContentProvider。通过获得这些ContentProvider可以查询它们包含的数据，当然前提是已获得适当的读取权限。

主要方法：

    　　public boolean onCreate() 在创建ContentProvider时调用

    　　public Cursor query(Uri, String[], String, String[], String) 用于查询指定Uri的ContentProvider，返回一个Cursor

    　　public Uri insert(Uri, ContentValues) 用于添加数据到指定Uri的ContentProvider中

    　　public int update(Uri, ContentValues, String, String[]) 用于更新指定Uri的ContentProvider中的数据

    　　public int delete(Uri, String, String[]) 用于从指定Uri的ContentProvider中删除数据

    　　public String getType(Uri) 用于返回指定的Uri中的数据的MIME类型

- 如果操作的数据属于集合类型，那么MIME类型字符串应该以vnd.android.cursor.dir/开头。
    - 例如：要得到所有person记录的Uri为content://contacts/person，那么返回的MIME类型字符串为"vnd.android.cursor.dir/person"。

- 如果要操作的数据属于非集合类型数据，那么MIME类型字符串应该以vnd.android.cursor.item/开头。
    - 例如：要得到id为10的person记录的Uri为content://contacts/person/10，那么返回的MIME类型字符串应为"vnd.android.cursor.item/person"。

**ContentResolver**
- 当外部应用需要对ContentProvider中的数据进行添加、删除、修改和查询操作时，可以使用ContentResolver类来完成，要获取ContentResolver对象，可以使用Context提供的getContentResolver()方法。
    ```
    　　ContentResolver cr = getContentResolver();

    　　ContentResolver提供的方法和ContentProvider提供的方法对应的有以下几个方法。

    　　public Uri insert(Uri uri, ContentValues values) 用于添加数据到指定Uri的ContentProvider中。

    　　public int delete(Uri uri, String selection, String[] selectionArgs) 用于从指定Uri的ContentProvider中删除数据。

    　　public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs) 用于更新指定Uri的ContentProvider中的数据。

    　　public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) 用于查询指定Uri的ContentProvider。
    ```


**Uri**
- Uri指定了将要操作的ContentProvider，其实可以把一个Uri看作是一个网址，我们把Uri分为三部分。
    - 第一部分是"content://"。可以看作是网址中的"http://"。
    - 第二部分是主机名或authority，用于唯一标识这个ContentProvider，外部应用需要根据这个标识来找到它。可以看作是网址中的主机名，比如"blog.csdn.net"。
    - 第三部分是路径名，用来表示将要操作的数据。可以看作网址中细分的内容路径。




#### Broadcast receiver
BroadcastReceiver 用于异步接收广播Intent。主要有两大类，用于接收广播的：
- `正常广播 Normal broadcasts`（用 Context.sendBroadcast()发送）是完全异步的。它们都运行在一个未定义的顺序，通常是在同一时间。这样会更有效，但意味着receiver不能包含所要使用的结果或中止的API。
- `有序广播 Ordered broadcasts`（用 Context.sendOrderedBroadcast()发送）每次被发送到一个receiver。所谓有序，就是每个receiver执行后可以传播到下一个receiver，也可以完全中止传播--不传播给其他receiver。 而receiver运行的顺序可以通过matched intent-filter 里面的`android:priority`来控制，当priority优先级相同的时候，Receiver以任意的顺序运行。
- 要注意的是，即使是Normal broadcasts，系统在某些情况下可能会恢复到一次传播给一个receiver。 特别是receiver可能需要创建一个进程，为了避免系统超载，只能一次运行一个receiver。
- Broadcast Receiver 并没有提供可视化的界面来显示广播信息。可以使用Notification和Notification Manager来实现可视化的信息的界面，显示广播信息的内容，图标及震动信息。

**生命周期**
- 一个BroadcastReceiver 对象只有在被调用onReceive(Context, Intent)的才有效的，当从该函数返回后，该对象就无效的了，结束生命周期。
- 因此从这个特征可以看出，在所调用的onReceive(Context, Intent)函数里，不能有过于耗时的操作，不能使用线程来执行。对于耗时的操作，请start service来完成。因为当得到其他异步操作所返回的结果时，BroadcastReceiver 可能已经无效了。

**发送广播**
事件的广播比较简单，构建Intent对象，可调用sendBroadcast(Intent)方法将广播发出。另外还有sendOrderedBroadcast()，sendStickyBroadcast()等方法，请查阅API Doc。
1. new Intent with action name

        Intent intent = new Intent(String action);

    或者 只是new Intent, 然后

        intent.setAction(String action);

2. set data等准备好了后，in activity,

        sendBroadcast(Intent); // 发送广播


        
**接收广播**
通过定义一个继承BroadcastReceiver类来实现，继承该类后覆盖其onReceiver方法，并在该方法中响应事件。

**注册Receiver**
1. 静态方式，在AndroidManifest.xml的application里面定义receiver并设置要接收的action。
2. 动态方式, 在activity里面调用函数来注册，和静态的内容差不多。一个形参是receiver，另一个是IntentFilter，其中里面是要接收的action。而且动态注册，需要特别注意的是，`在退出程序前要记得调用Context.unregisterReceiver()方法`·`。一般在activity的onStart()里面进行注册, onStop()里面进行注销。官方提醒，如果在Activity.onResume()里面注册了，就必须在Activity.onPause()注销。

    ```
    public class HelloDemo extends Activity {    
            private BroadcastReceiver receiver;    

            @Override 
            protected void onStart() { 
                    super.onStart(); 

                    receiver = new CallReceiver(); 
                    registerReceiver(receiver, new IntentFilter("android.intent.action.PHONE_STATE")); 
            } 

            @Override 
            protected void onStop() { 
                    unregisterReceiver(receiver); 
                    super.onStop(); 
            } 
    }
    ```

**小结：**
1. 对于sendBroadCast的intent对象，需要设置其action name；
2. 推荐使用显式指明receiver，在配置文件AndroidManifest.xml指明；
3. 一个receiver可以接收多个action;
4. 每次接收广播都会重新生成一个接收广播的对象，再次调用onReceive；
5. 在BroadCast 中尽量不要处理太多逻辑问题，建议复杂的逻辑交给Activity 或者 Service 去处理。



---

### 进程与线程

- 进程是系统进行资源分配和调度的基本单位
- 线程是CPU调度和分派的基本单位,它是比进程更小的能独立运行的基本单位。

**并发**的关键是你有处理多个任务的能力，不一定要同时。**并行**的关键是你有同时处理多个任务的能力。所以说，并行是并发的子集

#### 进程和线程的关系

1. 一个线程只能属于一个进程，而一个进程可以有多个线程，但至少有一个线程。线程是操作系统可识别的最小执行和调度单位。
2. 资源分配给进程，同一进程的所有线程共享该进程的所有资源。 同一进程中的多个线程共享代码段(代码和常量)，数据段(全局变量和静态变量)，扩展段(堆存储)。但是每个线程拥有自己的栈段，栈段又叫运行时段，用来存放所有局部变量和临时变量。
3. 处理机分给线程，即真正在处理机上运行的是线程。
4. 线程在执行过程中，需要协作同步。不同进程的线程间要利用消息通信的办法实现同步。

#### 线程池——Java中的ThreadPoolExecutor类

线程池是一种线程使用模式。线程过多会带来调度开销，进而影响缓存局部性和整体性能。而线程池维护着多个线程，等待着监督管理者分配可并发执行的任务。这避免了在处理短时间任务时创建与销毁线程的代价。线程池不仅能够保证内核的充分利用，还能防止过分调度。可用线程数量应该取决于可用的并发处理器、处理器内核、内存、网络sockets等的数量。 例如，线程数一般取cpu数量+2比较合适，线程数过多会导致额外的线程切换开销。

**组成部分:**

1. 线程池管理器（ThreadPoolManager）:用于创建并管理线程池
2. 工作线程（WorkThread）: 线程池中线程
3. 任务接口（Task）:每个任务必须实现的接口，以供工作线程调度任务的执行。
4. 任务队列:用于存放没有处理的任务。提供一种缓冲机制。

**应用范围:**

1. 需要大量的线程来完成任务，且完成任务的时间比较短。 WEB服务器完成网页请求这样的任务，使用线程池技术是非常合适的。因为单个任务小，而任务数量巨大，你可以想象一个热门网站的点击次数。 但对于长时间的任务，比如一个Telnet连接请求，线程池的优点就不明显了。因为Telnet会话时间比线程的创建时间大多了。
2. 对性能要求苛刻的应用，比如要求服务器迅速响应客户请求。
3. 接受突发性的大量请求，但不至于使服务器因此产生大量线程的应用。突发性大量客户请求，在没有线程池情况下，将产生大量线程，虽然理论上大部分操作系统线程数目最大值不是问题，短时间内产生大量线程可能使内存到达极限，并出现"OutOfMemory"的错误。

---

### 进程间通信（IPC，Interprosses communication）

- 进程是操作系统里最基本的单位，进程的一大特征就是有自己独立的内存地址空间，所以同一进程里的代码能直接访问其地址空间的内存，而其它进程是不能直接访问的。操作系统是多进程的，多个进程配合工作肯定会涉及到互相的通信，故出现了IPC，而对IPC的实现也不是一种，有很多，且有各自的长处，比如Socket就是一种IPC，其长处就是能跨机器进行进程间通信。 
- Android系统中的进程之间不能共享内存，共享内存是最快的 IPC 方式，它是针对其他进程间通信方式运行效率低而专门设计的。
- AIDL（Android Interface Definition Language）是Android系统自定义的接口描述语言。可以用来实现进程间的通讯。
- 在 Android 中，要实现进程间的通讯，一般来说，有以下几种方式：
  - 使用 Bundle
    - 最常见的的是我们通过特定的 Action 或者 data 启动另外一个应用的 Activity 或者 service。我们可以将要传递的数据封装在 bundle 当中。
  - 文件共享
    - 两个应用读取某个文件，从而达到进程通讯的问题，不过这种方法需要处理好文件锁的问题，不然很容易引发数据错乱。适合无并发情景
  - 使用 Messenger
    - 低并发的一对多即时通信
    - Messenger 进行进程间的通讯是串行的，而且是单向的，如果客户端和服务端想进行双向通讯，需要维护两个 Messenger，相对比较麻烦
  - 使用AIDL
  - Content Provider :一对多的进程间数据共享
  - Socket：网络数据交换
- 如果不需要IPC，那就直接实现通过继承Binder类来实现客户端和服务端之间的通信。
- 如果确实需要IPC，但是无需处理多线程，那么就应该通过Messenger来实现。Messenger保证了消息是串行处理的，其内部其实也是通过AIDL来实现。
- 在有IPC需求，同时服务端需要并发处理多个请求的时候，使用AIDL才是必要的！

#### AIDL
设计这门语言的目的是为了实现`进程间通信`。


每一个进程都有自己的Dalvik VM实例，都有自己的一块独立的内存，都在自己的内存上存储自己的数据，执行着自己的操作，都在自己的那片狭小的空间里过完自己的一生。每个进程之间都你不知我，我不知你，就像是隔江相望的两座小岛一样，都在同一个世界里，但又各自有着自己的世界。而AIDL，就是两座小岛之间沟通的桥梁。相对于它们而言，我们就好像造物主一样，我们可以通过AIDL来制定一些规则，规定它们能进行哪些交流——比如，它们可以在我们制定的规则下传输一些特定规格的数据。

总之，通过这门语言，我们可以愉快的在一个进程访问另一个进程的数据，甚至调用它的一些方法，当然，只能是特定的方法。
- 两种AIDL文件：在我的理解里，所有的AIDL文件大致可以分为两类。一类是用来定义parcelable对象，以供其他AIDL文件使用AIDL中非默认支持的数据类型的。一类是用来定义方法接口，以供系统使用来完成跨进程通信的。可以看到，两类文件都是在“定义”些什么，而不涉及具体的实现，这就是为什么它叫做“Android接口定义语言”。 
>注：所有的非默认支持数据类型必须通过第一类AIDL文件定义才能被使用。
- AIDL是为了方便使用Binder框架，其实不用这个也能达到目的，但是用这个就简化了操作。
  - 创建 AIDL 
    - 创建要操作的实体类，实现 Parcelable 接口，以便序列化/反序列化
    - 新建 aidl 文件夹，在其中创建接口 aidl 文件以及实体类的映射 aidl 文件
    - Make project ，生成 Binder 的 Java 文件
  - 服务端 
    - 创建 Service，在其中创建上面生成的 Binder 对象实例，实现接口定义的方法
    - 在 onBind() 中返回
  - 客户端 
    - 实现 ServiceConnection 接口，在其中拿到 AIDL 类
    - bindService()
    - 调用 AIDL 类中定义好的操作请求
- 默认支持的数据类型包括： 
  - Java中的八种基本数据类型，包括 byte，short，int，long，float，double，boolean，char。
  - String 类型。
  - CharSequence类型。
  - List类型：List中的所有元素必须是AIDL支持的类型之一，或者是一个其他AIDL生成的接口，或者是定义的parcelable。List可以使用泛型。
  - Map类型：Map中的所有元素必须是AIDL支持的类型之一，或者是一个其他AIDL生成的接口，或者是定义的parcelable。Map是不支持泛型的。

---

### 线程间通信

1. 通过Handler机制
   - 为什么需要 Handler？
     - 子线程不允许访问 UI
       - 假若子线程允许访问 UI，则在多线程并发访问情况下，会使得 UI 控件处于不可预期的状态。
       - 传统解决办法：加锁，但会使得UI访问逻辑变的复杂，其次降低 UI 访问的效率。
     - 引入 Handler
       - 采用单线程模型处理 UI 操作，通过 Handler 切换到 UI 线程，解决子线程中无法访问 UI 的问题。
2. runOnUiThread方法
       public final void runOnUiThread(Runnable action) {
           if (Thread.currentThread() != mUiThread) {
               mHandler.post(action);
           } else {
               action.run();
           }
       }
   - 从上面的源代码中可以看出，程序首先会判断当前线程是否是UI线程，如果是就直接运行，如果不是则post，这时其实质还是使用的Handler机制来处理线程与UI通讯。
3. View.post(Runnable r) 
4. AsyncTask

---

### Handler

#### Handler 使用

- 方式一： post(Runnable)
  - 创建一个工作线程，实现 Runnable 接口，实现 run 方法，处理耗时操作
  - 创建一个 handler，通过 handler.post/postDelay，投递创建的 Runnable，在 run 方法中进行更新 UI 操作。
        
        new Thread(new Runnable() {
            @Override
            public void run() {
                /**
                    耗时操作
                    */
                handler.post(new Runnable() {
                    @Override
                    public void run() {
                        /**
                            更新UI
                        */
                    }
                });
            }
        }).start();
            
- 方式二： sendMessage(Message)
    - 创建一个工作线程，继承 Thread，重新 run 方法，处理耗时操作
    - 创建一个 Message 对象，设置 what 标志及数据
    - 通过 sendMessage 进行投递消息
    - 创建一个handler，重写 handleMessage 方法，根据 msg.what 信息判断，接收对应的信息，再在这里更新 UI。
    1. 传递消息Message
        ```    
        //1.通过handler实例获取
        Handler handler = new Handler();
        Message message=handler.obtainMessage();
        
        //2.通过Message获取
        Message message=Message.obtain();
        
        //源码中第一种获取方式其实也是内部调用了第二种：
        public final Message obtainMessage(){
            return Message.obtain(this);
        }
        ```
        `不建议直接new Message，Message内部保存了一个缓存的消息池`，我们可以用obtain从缓存池获得一个消息，Message使用完后系统会调用recycle回收，如果自己new很多Message，每次使用完后系统放入缓存池，会占用很多内存的。
        ```
        //传递的数据
        Bundle bundle = new Bundle();
        bundle.putString("msg", "传递我这个消息");
        //发送数据
        Message message = Message.obtain();
        message.setData(bundle);   //message.obj=bundle  传值也行
        message.what = 0x11;
        handler.sendMessage(message);
        
        
        
        //数据的接收
        final Handler handler = new Handler() {
                @Override
                public void handleMessage(Message msg) {
                    super.handleMessage(msg);
                    if (msg.what == 0x11) {
                        Bundle bundle = msg.getData();
                        String date = bundle.getString("msg");
                    }
                }
        };
        ```
    2. 子线程通知主线程更新ui
        ```
        //创建handler
        final Handler handler = new Handler() {
            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
                if (msg.what == 0x11) {
                    //更新ui
                          ......
                }
            }
        };
 
        new Thread(new Runnable() {
            @Override
            public void run() {
                //FIXME 这里直接更新ui是不行的
                //还有其他更新ui方式,runOnUiThread()等          
                message.what = 0x11;     
                handler.sendMessage(message);  
            }
        }).start();
        ```
    

#### Handler 存在的问题

- 内存方面
  - Handler 被作为 Activity 引用，如果为非静态内部类，则会隐式地引用外部类对象。当 Activity finish 时，Handler可能并未执行完，从而引起 Activity 的内存泄漏。故而在所有调用 Handler 的地方，都用静态内部类。
  - 静态内部类不会持有外部类的的引用，当需要引用外部类相关操作时，可以通过弱引用还获取到外部类相关操作，弱引用是不会造成对象该回收回收不掉的问题，不清楚的可以查阅JAVA的几种引用方式的详细说明。
  - 在外部类对象被销毁时，将MessageQueue中的消息清空。例如，在Activity的onDestroy时将消息清空。
- 异常方面
  - 当 Activity finish 时,在 onDestroy 方法中释放了一些资源。此时 Handler 执行到 handlerMessage 方法,但相关资源已经被释放,从而引起空指针的异常。`所以在ondestory中去remove掉我们要处理的事件`，还是有必要的。不想处理就直接try catch或者判空。
  - 有时候你会发现removeCallbacks会失效，不能从消息队列中移除。出现这情况是activity切入后台，再回到前台，此时的runnable由于被重定义，就会和原先的runnable并非同一个对象。所以这么做，加上static即可
- 如何避免
  - 如果是使用 handlerMessage，则在方法中加try catch。
  - 如果是用 post 方法，则在Runnable方法中加try catch。
  - 创建静态内部类 || 持有弱引用MainActivity，GC回收时会被回收掉.
    ```
    public class MainActivity extends AppCompatActivity {
    
        //创建静态内部类
        private static class MyHandler extends Handler{
            //持有弱引用MainActivity,GC回收时会被回收掉.
            private final WeakReference<MainActivity> mAct;
            public MyHandler(MainActivity mainActivity){
                mAct =new WeakReference<MainActivity>(mainActivity);
            }
            @Override
            public void handleMessage(Message msg) {
                MainActivity mainAct=mAct.get();
                super.handleMessage(msg);
                if(mainAct!=null){
                    //执行业务逻辑
                }
            }
        }
        private static final Runnable myRunnable = new Runnable() {
            @Override
            public void run() {
                //执行我们的业务逻辑
            }
        };
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            MyHandler myHandler=new MyHandler(this);
            //延迟5分钟后发送
            myHandler.postDelayed(myRunnable, 1000 * 60 * 5);
        }
    }
    ```

#### Handler 通信机制

- 创建Handler，并采用当前线程的Looper创建消息循环系统；
- Handler通过sendMessage(Message)或Post(Runnable)发送消息，调用enqueueMessage把消息插入到消息队列中；
- Looper循环检测消息队列中的消息，若有消息则取出该消息，并调用该消息持有的handler的dispatchMessage方法，回调到创建Handler线程中重写的handleMessage里执行。

#### 为什么在主线程中创建Handler不需要要用Looper.prepare()和Looper.loop()方法呢？

- 其实不是这样的，App初始化的时候都会执行ActivityThread的main方法，我们可以看看ActivityThread的main()方法都做了什么，在创建主线程的时候Android已经帮我们调用了Looper.prepareMainLooper()

**ThreadLocal是什么**

- ThreadLocal是一个本地线程副本变量工具类。主要用于将私有线程和该线程存放的副本对象做一个映射，各个线程之间的变量互不干扰，在高并发场景下，可以实现无状态的调用，特别适用于各个线程依赖不通的变量值完成操作的场景。

---

### listView优化：

- 使用viewHolder——findViewById方法耗时较大，使用ViewHolder主要是为了可以省去这个时间。通过setTag，getTag直接存储和获取ViewHolder
  - Tag从本质上来讲是就是相关联的view的额外的信息。它们经常用来存储一些view的数据，这样做非常方便而不用存入另外的单独结构。
- 优化getView方法——在getView方法中使用convertView来避免每次getView时都要新建view，缓存中有便直接复用
- 滑动时不加载图片
- 分页加载数据
- 不要在getView中进行大量的逻辑处理或者进行耗时操作
- 减少Item View的布局层级——这是所有layout都必须遵循的，布局层级过深会直接导致View的测量与绘制浪费大量的时间

使用RecyclerView时候，必须指定一个适配器Adapter和一个布局管理器LayoutManager

---

### TCP 三次握手与四次挥手

![](/img/in-post/post-Android/review/TCP3woshou.png)
![](/img/in-post/post-Android/review/TCP4huishou.png)

- 若两次握手会怎么样？
  - 当A和B通信完成后，A之前发送的发起请求的SYN此时到达了B，那么B又会以为建立新的连接，以为这是一个新的请求连接消息，就向A发了一次确认，而对于A而言，他认为他没有给B再次的发消息（因为上次的通话已经结束了），所以A不会理睬这条确认，但是B则会一直在傻傻的等待着A的消息
- 四次挥手
    1. 为什么建立连接协议是三次握手，而关闭连接却是四次握手呢？
        - 这是因为服务端的LISTEN状态下的SOCKET当收到SYN报文的建连请求后，它可以把ACK和SYN（ACK起应答作用，而SYN起同步作用）放在一个报文里来发送。但关闭连接时，当收到对方的FIN报文通知时，它仅仅表示对方没有数据发送给你了；但未必你所有的数据都全部发送给对方了，所以你不可以马上关闭SOCKET,也即你可能还需要发送一些数据给对方之后，再发送FIN报文给对方来表示你同意现在可以关闭连接了，所以它这里的ACK报文和FIN报文多数情况下都是分开发送的。
    2. 为什么TIME_WAIT状态还需要等2MSL后才能返回到CLOSED状态？
        - MSL即Maximum Segment Lifetime，也就是报文最大生存时间，引用《TCP/IP详解》中的话：“它(MSL)是任何报文段被丢弃前在网络内的最长时间。”
        - 这是因为虽然双方都同意关闭连接了，而且握手的4个报文也都协调和发送完毕，按理可以直接回到CLOSED状态（就好比从SYN_SEND状态到ESTABLISH状态那样）；但是因为我们必须要假想网络是不可靠的，你无法保证你最后发送的ACK报文会一定被对方收到，因此对方处于LAST_ACK状态下的SOCKET可能会因为超时未收到ACK报文，而重发FIN报文，所以这个TIME_WAIT状态的作用就是用来重发可能丢失的ACK报文。

---

### Activity生命周期

![](/img/in-post/post-Android/review/activity.png)

- 关于activity的四个状态： running-poused-stopped-killed
  - running->当前显示在屏幕的activity(位于任务栈的顶部)，用户可见状态。
    - paused->依旧在用户可见状态，但是界面焦点已经失去，此Activity无法与用户进行交互。
    - stopped->用户看不到当前界面,也无法与用户进行交互 完全被覆盖.
    - killed->当前界面被销毁，等待这系统被回收
  - Starting ——–>Running 所执行的生命周期顺序 onCreate()->onstart()->onResume()
    - 当前称为活动状态（Running），此activity所处于任务栈的top中，可以与用户进行交互。
  - Running ——>Paused 所执行Activity生命周期中的onPause（）
    - 当前称为暂停状态（Paused），该Activity已失去了焦点但仍然是可见的状态(包括部分可见)。
  - Paused ——>Running所执行的生命周期为:OnResume()
    - 当前重新回到活动状态(Running),此情况用户操作home键，然后重新回到当前activity界面发生。
  - Paused ——>Stoped所执行的生命周期为:onStop()
    - 该Activity被另一个Activity完全覆盖的状态,该Activity变得不可见，所以系统经常会由于内存不足而将该Activity强行结束。
  - Stoped——>killed所执行的生命周期为:onDestroy()
    - 该Activity被系统销毁。当一个Activity处于暂停状态或停止状态时就随处可能进入死亡状态，因为系统可能因内存不足而强行结束该Activity。

#### activity的进程优先级

- 前台进程>可见进程>service进程>后台进程>空进程
- 前台进程:
    1. 当前进程activity正在与用户进行交互。
 	2. 当前进程service正在与activity进行交互或者当前service调用了startForground()属于前台进程或者当前service正在执行生命周期（onCreate(),onStart(),onDestory()）
 	3. 进程持有一个BroadcostReceiver,这个BroadcostReceiver正在执行onReceive()方法
- 可见进程:
    1. 进程持有一个activity，这个activity不再前台，处于onPouse()状态下，当前覆盖的activity是以dialog形式存在的。
    2. 进程有一个service，这个service和一个可见的Activity进行绑定。
- service进程:
    - 当前开启startSerice()启动一个service服务就可以认为进程是一个服务进程。
- 后台进程:
    - activity的onStop()被调用，但是onDestroy()没有调用的状态。该进程属于后台进程。
- 空进程:
    - 改进程没有任何运行的数据了，且保留在内存空间，并没有被系统killed,属于空进程。该进程很容易被杀死。

---

### 保持进程不被杀死——service保活，类似微信QQ
1. Service设置成START_STICKY，kill 后会被重启（等待5秒左右），重传Intent，保持与重启前一样
    ```
    @Override  
    public int onStartCommand(Intent intent, int flags, int startId) {  
        flags = START_STICKY;  
        return super.onStartCommand(intent, flags, startId);  
    }  
    ```
    手动返回START_STICKY，亲测当service因内存不足被kill，当内存又有的时候，service又被重新创建，比较不错，但是不能保证任何情况下都被重建，比如进程被干掉了。
2. 通过 startForeground将进程设置为前台进程，做前台服务，优先级和前台应用一个级别​，除非在系统内存非常缺，否则此进程不会被 kill. 
    ```
    public void MyService.onCreate() {
        super.onCreate();
        Notification notification = new Notification(android.R.drawable.my_service_icon,
               "my_service_name",
                System.currentTimeMillis());
        PendingIntent p_intent = PendingIntent.getActivity(this, 0,
                new Intent(this, MyMainActivity.class), 0);
        notification.setLatestEventInfo(this, "MyServiceNotification, "MyServiceNotification is Running!",   p_intent);
        Log.d(TAG, String.format("notification = %s", notification));
        startForeground(0x1982, notification);   // notification ID: 0x1982, you can name it as you will.
    }
    ```
经本人测试，有些设备上，会在通知栏一直显示，这让用户有点糟心= = 
3. 相较于/data/app下的应用，放在/system/app下的应用享受更多的特权，比如若在其Manifest.xml文件中设置persistent属性为true，则可使其免受out-of-memory killer的影响。如应用程序'Phone'的AndroidManifest.xml文件：
    ```
     <application android:name="PhoneApp"
                 android:persistent="true"
                 android:label="@string/dialerIconLabel"
                 android:icon="@drawable/ic_launcher_phone">
         ...
    </application>
    ```
4. 双进程Service：让2个进程互相保护，其中一个Service被清理后，另外没被清理的进程可以立即重启进程 
5. QQ黑科技:在应用退到后台后，另起一个只有 1 像素的页面停留在桌面上，让自己保持前台状态，保护自己不被后台清理工具杀死 
6. 在已经root的设备下，修改相应的权限文件，将App伪装成系统级的应用（Android4.0系列的一个漏洞，已经确认可行） 
7. Android系统中当前进程(Process)fork出来的子进程，被系统认为是两个不同的进程。当父进程被杀死的时候，子进程仍然可以存活，并不受影响。鉴于目前提到的在Android-Service层做双守护都会失败，我们可以fork出c进程，多进程守护。死循环在那检查是否还存在，具体的思路如下（Android5.0以下可行） 
    - 用C编写守护进程(即子进程)，守护进程做的事情就是循环检查目标进程是否存在，不存在则启动它。 
    - 在NDK环境中将1中编写的C代码编译打包成可执行文件(BUILD_EXECUTABLE)。 
    - 主进程启动时将守护进程放入私有目录下，赋予可执行权限，启动它即可。 
8. 联系厂商，加入白名单

---

### 安卓 service

- 与service无交互
  - 启动服务
  - 使用startService()方法启动的服务，在服务的外部，必须使用stopService()方法停止，在服务的内部可以调用stopSelf()方法停止当前服务。如果使用startService()或者stopSelf()方法请求停止服务，系统会就会尽快销毁这个服务。值得注意的是对于启动服务，一旦启动将与访问它的组件无任何关联，即使访问它的组件被销毁了，这个服务也一直运行下去，直到手动调用停止服务才被销毁
- 与service有交互
  - 绑定服务
  - 与启动服务不同的是绑定服务的生命周期通常只在为其他应用组件(如Activity)服务时处于活动状态，不会无限期在后台运行，也就是说宿主(如Activity)解除绑定后，绑定服务就会被销毁

#### service生命周期

- onCreate()：
  - 首次创建服务时，系统将调用此方法。如果服务已在运行，则不会调用此方法，该方法只调用一次。
- onStartCommand()：
  - 当另一个组件通过调用startService()请求启动服务时，系统将调用此方法。
- onDestroy()：
  - 当服务不再使用且将被销毁时，系统将调用此方法。
- onBind()：
  - 当另一个组件通过调用bindService()与服务绑定时，系统将调用此方法。
- onUnbind()：
  - 当另一个组件通过调用unbindService()与服务解绑时，系统将调用此方法。
- onRebind()：
  - 当旧的组件与服务解绑后，另一个新的组件与服务绑定，onUnbind()返回true时，系统将调用此方法。

1. 启动Service服务
    - 单次：startService() —> onCreate() —> onStartCommand()
    - 多次：startService() —> onCreate() —> onStartCommand() —> onStartCommand()

2. 停止Service服务
    - stopService() —> onDestroy()
3. 绑定Service服务
    - bindService() —> onCreate() —> onBind()
4. 解绑Service服务
    - unbindService() —> onUnbind() —> onDestroy()
5. 启动绑定Service服务
    - startService() —> onCreate() —> onStartCommand() —> bindService() —> onBind()
6. 解绑停止Service服务
    - unbindService() —> onUnbind() —> stopService() —> onDestroy()
7. 解绑绑定Service服务
    - unbindService() —> onUnbind(ture) —> bindService() —> onRebind()

**当我们既想绑定一个Service（为了实现Activity和Service的交互）又想在 Activity停止时，Service不会停止，我们可以先StartService，然后再BindService()。**

这样的话，`当Activity退出的时候，Service的onUnbind()方法就会被调用，但Sercvice并不会停止`，然后我们可以再进入Activity重新绑定该Service，这个时候 Service就会调用onRebind()方法。

但是onRebind()方法被调用还有个前提是先前的**onUnbind()方法返回值为true**，但是如果使用默认的 super.onUnbind(intent)是不行的，这时候我们要手动的使其返回true，再次绑定时onRebind()就会执行了。




#### service后台在实时更新东西，activity需要实时获取呢？

1. 使用接口回调方式，activity实现相应的接口，service通过接口进行回调，比较灵活
   - 可不可以有一种方法当Service中进度发生变化主动通知Activity，答案是肯定的，我们可以利用回调接口实现Service的主动通知，不理解回调方法的可以看看 http://blog.csdn.net/xiaanming/article/details/8703708
2. 使用广播
   - 当我们的进度发生变化的时候我们发送一条广播，然后在Activity的注册广播接收器，接收到广播之后更新ProgressBar
   - 通过广播实现Activity和Service的交互简单容易实现，缺点是发送不广播受系统制约，系统会优先发送系统级的广播，自定义的广播接收器可能会有延迟，在广播里也不能有耗时操作，否则会导致程序无响应
3. 文件共享
   - 使用SharedPreferences进行数据共享对文件格式没有要求，只要读写双方约定好数据格式即可。但是也有局限性，在面对高并发的读写时，这种方式就变得不可靠，很可能会导致读写的数据不一致，所以不建议使用这种方式来进行通信
4. Messenger
   1. 在Service端创建信使对象
创建Messenger需要传入一个Handler对象，所以首先要新建一个Handler，利用Handler来创建信使

          @Override
            public void onCreate() {
                // TODO Auto-generated method stub
                super.onCreate();
                mMessenger = new Messenger(handler);
            }
      
   2. Service端的onBind()方法使用mMessenger.getBinder()返回一个binder对象

          @Override
              public IBinder onBind(Intent intent) {
                  // TODO Auto-generated method stub
                  return mMessenger.getBinder();
              }
   3. 客户端绑定到Service，在onServiceConnected()方法中使用Service返回的IBinder对象创建Messenger对象，通过这个Messenger对象就可以向Service发送消息了

           ServiceConnection connection = new ServiceConnection() {
               public void onServiceConnected(ComponentName name, android.os.IBinder service) {
                   rMessenger = new Messenger(service);
               };
               public void onServiceDisconnected(ComponentName name) {
           
               };
           };
   4. 这样只是实现了客户端向Service发送消息，如果需要Service可以将相应客户端，同样的需要在客户端使用Handler来创建Messenger对象，通过Message将这个Messenger传到Service中，Service获取到客户端的Messenger对象后，也可以向客户端发送消息。
          

---

### 自定义View需要重写哪些函数？——测量、布局、绘制

![](/img/in-post/post-Android/review/view.png)

- onMeasure 
  - 测量本质就是测量本身有多大，也就是给mMeasuredWidth和mMeasuredHeight这两个属性赋值，也就是调用setMeasuredDimension这个方法。另外父view测量子view的时候调用的measure方法，还有一些衍生方法如measureChildWithMargins。
- onLayout 
  - 大多数情况是在自定义ViewGroup中才会重写
  - 作用是子view应该怎样放置，也就是设置子view的mLeft、mTop、mRight、mBottom属性。该方法在View中是空实现，很显然主要用于ViewGroup。父view放置子view的时候调用layout方法。
- onDraw 
  - 具体长什么样。
- onDraw()方法必须有，是用来绘制View图像的
- 如果要改变View 的大小，需要重写onMeasure()方法。
- 如果要改变View在父控件中的位置，需要重写onLayout()方法

注：
- onMeasure()会在初始化之后调用一到多次来测量控件或其中的子控件的宽高；
- onLayout()会在onMeasure()方法之后被调用一次，将控件或其子控件进行布局；
- onDraw()会在onLayout()方法之后调用一次，也会在用户手指触摸屏幕时被调用多次，来绘制控件。



---

### Fragment生命周期

![](/img/in-post/post-Android/review/fragment.png)

- `onAttach(Context context)`
    - 在Fragment和Activity关联上的时候调用，且仅调用一次。在该回调中我们可以将context转化为Activity保存下来，从而避免后期频繁调用getAtivity()获取Activity的局面，避免了在某些情况下getAtivity()为空的异常（Activity和Fragment分离的情况下）。同时也可以在该回调中将传入的Arguments提取并解析，在这里强烈推荐通过setArguments给Fragment传参数，因为在应用被系统回收时Fragment不会保存相关属性，具体之后会讲解。
- `onCreate`
    - 在最初创建Fragment的时候会调用，和Activity的onCreate类似。
- View `onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState)`：
    - 在准备绘制Fragment界面时调用，返回值为Fragment要绘制布局的根视图，当然也可以返回null。注意使用inflater构建View时一定要将attachToRoot指明false，因为Fragment会自动将视图添加到container中，attachToRoot为true会重复添加报错。onCreateView并不是一定会被调用，当添加的是没有界面的Fragment就不会调用，比如调用FragmentTransaction的 add(Fragment fragment, String tag)方法。
- `onActivityCreated` 
    - 在Activity的onCreated执行完时会调用。
- `onStart() `
    - Fragment对用户可见的时候调用，前提是Activity已经started。
- `onResume()`
    - Fragment和用户之前可交互时会调用，前提是Activity已经resumed。
- `onPause()`
    - Fragment和用户之前不可交互时会调用。
- `onSaveInstanceState()`
    - 通常在onPause之后调用，保存当前Fragment的状态。该方法会自动保存Fragment的状态，比如EditText键入的文本，即使Fragment被回收又重新创建，一样能恢复EditText之前键入的文本。
- `onStop()`
    - Fragment不可见时会调用。
- `onDestroyView()`
    - 销毁与Fragment有关的视图，但未与Activity解除绑定，依然可以通过onCreateView方法重新创建视图。通常在ViewPager+Fragment的方式下会调用此方法。
- `onDestroy()`
    - 销毁Fragment。通常按Back键退出或者Fragment被回收时调用此方法。
- `onDetach()`
    - Fragment和Activity解除关联时调用。
需要关注一下两者生命周期顺序问题，其中onCreate、onStart、onResume都是Activity先调用之后才是Fragment，onPause、onStop、onDestroy（在Fragment中是onDetach），是先Fragment调用之后才是Activity

#### Fragment相关操作对生命周期的影响

动态添加是在代码中FragmentManger使用一系列FragmentTransaction事务操作动态控制，灵活多变。一般都是使用动态添加，下面就讲讲动态添加有关的生命周期。

- add：onAttach->onCreate->onCreateView->onActivityCreated->onStart->onResume
- remove：onPause->onStop->onDestroyView->onDestroy->onDetach
- show：onHiddenChanged(boolean hidden) hidden为false
- hide：onHiddenChanged(boolean hidden) hidden为true
- replace：旧Fragment的remove生命周期->新Fragment的add生命周期
- replace+addToBackStack：onPause->onStop->onDestroyView->新Fragment的add生命周期之后点击back：新Fragment的remove->onCreateView->onViewCreated->onActivityCreated->onStart->onResume 就是第一张图的线
- detach：onPause->onStop->onDestroyView 可以看到只是视图被移除，Fragment关联状态还是不变，还是处于FragmentManger的管理下
- FragmentTransaction.attach(Fragment var1)：onStart->onResume->onCreateView

#### 应用被系统回收对Fragment生命周期的影响

应用被回收一般都是后台应用，所以生命周期是从onDestroyView开始

- 单独一个Fragment 
  - onDestroyView->onDestroy->onDetach->add生命周期
- Fragment A hide，Fragment B show 
  - A.onDestroyView->A.onDestroy->A.onDetach->B.onDestroyView->B.onDestroy->B.onDetach->A.onAttach->A.onCreate->B.onAttach->B.onCreate->A.onCreateView->A.onActivityCreated->B.onCreateView->B.onActivityCreated->A.onStart->B.onStart->A.onResume->B.onResume

---

### 任务栈

- "standard"(默认启动模式)
  - standard是默认的启动模式，即如果不指定launchMode属性，则自动就会使用这种启动模式。这种启动模式表示每次启动该Activity时系统都会为创建一个新的实例，并且总会把它放入到当前的任务当中。声明成这种启动模式的Activity可以被实例化多次，一个任务当中也可以包含多个这种Activity的实例。
- "singleTop"
  - `栈顶复用模式`
  - 这种启动模式表示，如果要启动的这个Activity在当前任务中已经存在了，并且还处于栈顶的位置，那么系统就不会再去创建一个该Activity的实例，而是调用栈顶Activity的onNewIntent()方法。声明成这种启动模式的Activity`也可以被实例化多次`，一个任务当中也可以包含多个这种Activity的实例。
  - 举个例子来讲，一个任务的返回栈中有A、B、C、D四个Activity，其中A在最底端，D在最顶端。这个时候如果我们要求再启动一次D，并且D的启动模式是"standard"，那么系统就会再创建一个D的实例放入到返回栈中，此时栈内元素为：A-B-C-D-D。而如果D的启动模式是"singleTop"的话，由于D已经是在栈顶了，那么系统就不会再创建一个D的实例，而是直接调用D Activity的onNewIntent()方法，此时栈内元素仍然为：A-B-C-D。
- "singleTask"
  - `栈内复用模式`
  - singleTask 模式比较适合应用的主界面activity（频繁使用的主架构），可以用于主架构的activity，（如新闻，侧滑，应用主界面等）
  - 这种启动模式表示，系统会创建一个新的任务，并将启动的Activity放入这个新任务的栈底位置。但是，如果现有任务当中已经存在一个该Activity的实例了，那么系统就不会再创建一次它的实例，而是会直接调用它的onNewIntent()方法 (将该任务栈的该activity之上的所有activity全部弹栈) 。声明成这种启动模式的Activity，在同一个任务当中只会存在一个实例。注意这里我们所说的启动Activity，都指的是启动其它应用程序中的Activity，因为"singleTask"模式在默认情况下只有启动其它程序的Activity才会创建一个新的任务，启动自己程序中的Activity还是会使用相同的任务，具体原因会在下面 处理affinity 部分进行解释。如果启动的是自己程序的activity呢？ 本任务栈中还是会只有一个实例么？？？（答案：系统会去检测要启动的这个Activity的affinity和当前任务的affinity是否相同，如果相同的话就会把它放入到现有任务当中，如果不同则会去创建一个新的任务。而同一个程序中所有Activity的affinity默认都是相同的，这也是前面为什么说，同一个应用程序中即使声明成"singleTask"，也不会为这个Activity再去创建一个新的任务了。）
- "singleInstance"
  - 这种启动模式和"singleTask"有点相似，只不过系统不会向声明成"singleInstance"的Activity所在的任务当中再添加其它Activity。也就是说，这种Activity所在的任务中始终只会有一个Activity，通过这个Activity再打开的其它Activity也会被放入到别的任务当中。

#### 适用场景
`singleTop`   
- 适合接收通知启动的内容显示页面。例如，某个新闻客户端的新闻内容页面，如果收到10个新闻推送，每次都打开一个新闻内容页面是很烦人的。
- 从外界可能多次跳转到一个界面

`singleTask`	
- 适合作为程序入口点。
- 例如浏览器的主界面。不管从多少个应用启动浏览器，只会启动主界面一次，其余情况都会走onNewIntent，并且会清空主界面上面的其他页面。
- 之前打开过的页面，打开之前的页面就ok，不再新建

`singleInstance`	
- 电话,闹钟.   
- 适合需要与程序分离开的页面。例如闹铃提醒，将闹铃提醒与闹铃设置分离。singleInstance不要用于中间页面，如果用于中间页面，跳转会有问题，比如：A -> B (singleInstance) -> C，完全退出后，再次启动，首先打开的是B。
- 某个应用中用到了google地图，当退出该应用的时候，进入google地图，还是刚才的界面


---

### 安卓设计模式 

设计模式遵循的原则：

- 功能单一明确，设计一个类的意图要明确，不能大包大揽什么功能都继承进去
    
- 对于扩展要开放，修改要关闭。软件通常都有需求变化，变化过程中通过扩展的方式来实现需求变化，而不是通过修改原有的方法，因为修改原有的方法会导致原来方法的调用方会出问题，这样层层调用出问题。
    
- 变化的进行抽象，不变的进行具体。设计代码过程中会面对很对可变的东西，比如在实现一个功能的时候，能够运用不同的方式进行实现，这个时候可以将每个具体的实现方法进行抽象，真正不变的是这个方法要实现的目的

1. 适配器模式：ListView或GridView的Adapter
   - 简介：不同的数据提供者使用一个适配器来向一个相同的客户提供服务。
2. 建造者模式：AlertDialog.Builder
   - 简介：可以分步地构造每一部分。
3. 命令模式：Handler.post后Handler.handleMessage
   - 简介：把请求封装成一个对象发送出去，方便定制、排队、取消。
     - 调用对象与作用对象之间分离，由中间件来协调两者之间的工作，如控制器；
4. 享元模式：Message.obtainMessage通过重用Message对象来避免大量的Message对象被频繁的创建和销毁。
   - 简介：运用共享技术有效地支持大量细粒度的对象。
5. 迭代器模式：如通过Hashtable.elements方法可以得到一个Enumeration，然后通过这个Enumeration访问Hashtable中的数据，而不用关心Hashtable中的数据存放方式。
   - 简介：提供一个方法顺序访问数据集合中的所有数据而又不暴露对象的内部表示。
6. 备忘录模式：Activity的onSaveInstanceState和onRestoreInstanceState就是通过Bundle这种序列化的数据结构来存储Activity的状态，至于其中存储的数据结构，这两个方法不用关心
   - 简介：不需要了解对象的内部结构的情况下备份对象的状态，方便以后恢复。
7. 观察者模式：我们可以通过BaseAdapter.registerDataSetObserver和BaseAdapter.unregisterDataSetObserver两方法来向BaseAdater注册、注销一个DataSetObserver。这个过程中，DataSetObserver就是一个观察者，它一旦发现BaseAdapter内部数据有变量，就会通过回调方法DataSetObserver.onChanged和DataSetObserver.onInvalidated来通知DataSetObserver的实现类。事件通知也是观察者模式
   - 简介：一个对象发生改变时，所有信赖于它的对象自动做相应改变。
8. 原型模式：比如我们需要一张Bitmap的几种不同格式：ARGB_8888、RGB_565、ARGB_4444、ALAPHA_8等。那我们就可以先创建一个ARGB_8888的Bitmap作为原型，在它的基础上，通过调用Bitmap.copy(Config)来创建出其它几种格式的Bitmap。另外一个例子就是Java中所有对象都有的一个名字叫clone的方法，已经原型模式的代名词了
   - 简介：在系统中要创建大量的对象，这些对象之间具有几乎完全相同的功能，只是在细节上有一点儿差别。
9. 代理模式：类似于ios开发的delegate委托模式，所有的AIDL都是一个代理模式的例子。假设一个Activity A去绑定一个Service S，那么A调用S中的每一个方法其实都是通过系统的Binder机制的中转，然后调用S中的对应方法来做到的。Binder机制就起到了代理的作用。
   - 简介：为其他对象提供一种代理以控制对这个对象的访问。
10. 状态模式：View.onVisibilityChanged方法，就是提供了一个状态模式的实现，允许在View的visibility发生改变时，引发执行onVisibilityChanged方法中的动作。
    - 简介：状态发生改变时，行为改变。
11. 策略模式：
    - 举例：Java.util.List就是定义了一个增（add）、删（remove）、改（set）、查（indexOf）策略，至于实现这个策略的ArrayList、LinkedList等类，只是在具体实现时采用了不同的算法。但因为它们策略一样，不考虑速度的情况下，使用时完全可以互相替换使用。
      - 简介：定义了一系列封装了算法、行为的对象，他们可以相互替换。
12. 调解者模式
    - 简介：一个对象的某个操作需要调用N个对象的M个方法来完成时，把这些调用过程封装起来，就成了一个调解者
      - 举例：如Resource.getDrawable方法的实现逻辑是这样的：创建一个缓存来存放所有已经加载过的，如果getDrawable中传入的id所对应的Drawable以前没有被加载过，那么它就会根据id所对应的资源类型，分别调用XML解析器生成，或者通过读取包中的图片资源文件来创建Drawable。
    - 而Resource.getDrawable把涉及到多个对象、多个逻辑的操作封装成一个方法，就实现了一个调解者的角色。
13. 抽象工厂模式
    - `使用场景是任何需要生成复杂对象的地方，都可以使用工厂方法模式。复杂对象使用工厂模式，用new就可以完成创建的对象无需使用工厂模式。`
    - 工厂模式：生产固定的一些东西，如抽象类，缺点是产品修改麻烦；如喜欢动作片和爱情片的人分别向服务器发出同一个请求，就可以得到他们想看的影片集，相当于不同对象进行同一请求，需求均得到满足。
      - DAO与Service的使用

- 简单工厂模式：一个抽象产品类，可以派生出多个具体产品类。一个具体工厂类，通过往此工厂的static方法中传入不同参数，产出不同的具体产品类实例。
- 工厂方法模式：一个抽象产品类，可以派生出多个具体产品类。一个抽象工厂类，可以派生出多个具体工厂类。每个具体工厂类只能创建一个具体产品类的实例。
- 抽象工厂模式： 多个抽象产品类，每个抽象产品类可以派生出多个具体产品类。一个抽象工厂类，可以派生出多个具体工厂类。每个具体工厂类可以创建多个具体产品类的实例。

---

### Android Binder

- Android内核是基于Linux系统，而Linux现存多种进程间IPC方式：管道，消息队列，共享内存，套接字，信号量，信号。
- 目前linux支持的IPC包括传统的管道，System V IPC，即消息队列/共享内存/信号量，以及socket中只有socket支持Client-Server的通信方式。当然也可以在这些底层机制上架设一套协议来实现Client-Server通信，但这样增加了系统的复杂性，在手机这种条件复杂，资源稀缺的环境下可靠性也难以保证。
- 另一方面是传输性能。socket作为一款通用接口，其传输效率低，开销大，主要用在跨网络的进程间通信和本机上进程间的低速通信。消息队列和管道采用存储-转发方式，即数据先从发送方缓存区拷贝到内核开辟的缓存区中，然后再从内核缓存区拷贝到接收方缓存区，至少有两次拷贝过程。共享内存虽然无需拷贝，但控制复杂，难以使用。
- Binder基于Client-Server通信模式，传输过程只需一次拷贝，为发送发添加UID/PID身份，既支持实名Binder也支持匿名Binder，安全性高。
- Binder框架定义了四个角色：Server，Client，ServiceManager（以后简称SMgr）以及Binder驱动。其中Server，Client，SMgr运行于用户空间，驱动运行于内核空间。这四个角色的关系和互联网类似：Server是服务器，Client是客户终端，SMgr是域名服务器（DNS），驱动是路由器。

1. Binder 驱动
   - 和路由器一样，Binder驱动虽然默默无闻，却是通信的核心。尽管名叫‘驱动’，实际上和硬件设备没有任何关系，只是实现方式和设备驱动程序是一样的。它工作于内核态，驱动负责进程之间Binder通信的建立，Binder在进程之间的传递，Binder引用计数管理，数据包在进程之间的传递和交互等一系列底层支持。
2. ServiceManager 与实名Binder
   - 和DNS类似，SMgr的作用是将字符形式的Binder名字转化成Client中对该Binder的引用，使得Client能够通过Binder名字获得对Server中Binder实体的引用。注册了名字的Binder叫实名Binder，就象每个网站除了有IP地址外还有自己的网址。Server创建了Binder实体，为其取一个字符形式，可读易记的名字，将这个Binder连同名字以数据包的形式通过Binder驱动发送给SMgr，通知SMgr注册一个名叫张三的Binder，它位于某个Server中。驱动为这个穿过进程边界的Binder创建位于内核中的实体节点以及SMgr对实体的引用，将名字及新建的引用打包传递给SMgr。SMgr收数据包后，从中取出名字和引用填入一张查找表中。
   - 细心的读者可能会发现其中的蹊跷：SMgr是一个进程，Server是另一个进程，Server向SMgr注册Binder必然会涉及进程间通信。当前实现的是进程间通信却又要用到进程间通信，这就好象蛋可以孵出鸡前提却是要找只鸡来孵蛋。Binder的实现比较巧妙：预先创造一只鸡来孵蛋：SMgr和其它进程同样采用Binder通信，SMgr是Server端，有自己的Binder对象（实体），其它进程都是Client，需要通过这个Binder的引用来实现Binder的注册，查询和获取。`SMgr提供的Binder比较特殊，它没有名字也不需要注册，当一个进程使用BINDER_SET_CONTEXT_MGR命令将自己注册成SMgr时Binder驱动会自动为它创建Binder实体（这就是那只预先造好的鸡）。其次这个Binder的引用在所有Client中都固定为0而无须通过其它手段获得。也就是说，一个Server若要向SMgr注册自己Binder就必需通过0这个引用号和SMgr的Binder通信`。类比网络通信，`0号引用就好比域名服务器的地址`，你必须预先手工或动态配置好。要注意这里说的Client是相对SMgr而言的，一个应用程序可能是个提供服务的Server，但对SMgr来说它仍然是个Client。
3. Client 获得实名Binder的引用
   - Server向SMgr注册了Binder实体及其名字后，Client就可以通过名字获得该Binder的引用了。Client也利用保留的0号引用向SMgr请求访问某个Binder：我申请获得名字叫张三的Binder的引用。SMgr收到这个连接请求，从请求数据包里获得Binder的名字，在查找表里找到该名字对应的条目，从条目中取出Binder的引用，将该引用作为回复发送给发起请求的Client。从面向对象的角度，这个Binder对象现在有了两个引用：一个位于SMgr中，一个位于发起请求的Client中。如果接下来有更多的Client请求该Binder，系统中就会有更多的引用指向该Binder，就象java里一个对象存在多个引用一样。而且类似的这些指向Binder的引用是强类型，从而确保只要有引用Binder实体就不会被释放掉。通过以上过程可以看出，SMgr象个火车票代售点，收集了所有火车的车票，可以通过它购买到乘坐各趟火车的票-得到某个Binder的引用。
4. 匿名 Binder
   - 并不是所有Binder都需要注册给SMgr广而告之的。Server端可以通过已经建立的Binder连接将创建的Binder实体传给Client，当然这条已经建立的Binder连接必须是通过实名Binder实现。由于这个Binder没有向SMgr注册名字，所以是个匿名Binder。Client将会收到这个匿名Binder的引用，通过这个引用向位于Server中的实体发送请求。匿名Binder为通信双方建立一条私密通道，只要Server没有把匿名Binder发给别的进程，别的进程就无法通过穷举或猜测等任何方式获得该Binder的引用，向该Binder发送请求。

---

### Android 尽量不要使用static静态变量

使用静态static静态变量潜在性问题：

1. 占用内存，并且内存一般不会释放；
2. 在系统不够内存情况下会自动回收静态内存，这样就会引起访问全局静态错误。
3. 不能将activity作为static静态对象，这样使activity的所有组件对象都存入全局内存中，并且不会被回收；
   （转自：http://blog.csdn.net/ctcwri/article/details/8858414）

#### 静态变量的生命周期：

- 类在什么时候被加载？
  - 当我们启动一个app的时候，系统会创建一个进程，此进程会加载一个Dalvik VM的实例，然后代码就运行在DVM之上，类的加载和卸载，垃圾回收等事情都由DVM负责。也就是说在进程启动的时候，类被加载，静态变量被分配内存。

#### 静态变量在类被卸载的时候销毁。

- 类在什么时候被卸载？
  - 在进程结束的时候。
  - 说明：一般情况下，所有的类都是默认的ClassLoader加载的，只要ClassLoader存在，类就不会被卸载，而默认的ClassLoader生命周期是与进程一致的，本文讨论一般情况。

#### Android中的进程什么时候结束

- 这个是Android对进程和内存管理不同于PC的核心——如果资源足够，Android不会杀掉任何进程，另一个意思就是进程随时可能会被杀掉。而Android会在资源够的时候，重启被杀掉的进程。也就是说静态变量的值，如果不做处理，是不可靠的，可以说内存中的一切都不可靠。如果要可靠，还是得保存到Nand或SD卡中去，在重启的时候恢复回来。
- 另一种情况就是不能把退出所有Activity等同于进程的退出，所以在用户点击图标启动应用的时候，以前存放于静态变量中的值，有可能还存在，因此要视具体情况给予清空操作。

#### Application也是一样不可靠

- Application其实是一个单例对象，也是放在内存中的，当进程被杀掉，就全清空了，只不过Android系统会帮重建Application，而我们存放在Application的数据自然就没有了，还是得自己处理。

#### 静态引用的对象不会被垃圾回收

- 只要静态变量没有被销毁也没有置null，其对象一直被保持引用，也即引用计数不可能是0，因此不会被垃圾回收。因此，单例对象在运行时不会被回收。

---

### Context——上下文

#### Context是什么?

1. Context是一个抽象类，其通用实现在ContextImpl类中。
2. Context：是一个访问application环境全局信息的接口，通过它可以访问application的资源和相关的类

其主要功能如下：

- 启动Activity
- 启动和停止Service
- 发送广播消息(Intent)
- 注册广播消息(Intent)接收者
- 可以访问APK中各种资源(如Resources和AssetManager等)
- 可以访问Package的相关信息
- APK的各种权限管理

#### 何时创建Context?

应用程序在以下几种情况下创建Context实例：

1. 创建Application 对象时， 而且整个App共一个Application对象
2. 创建Service对象时
3. 创建Activity对象时

因此应用程序App共有的Context数目公式为：

**总Context实例个数 = Service个数 + Activity个数 + 1（Application对应的Context实例）**


![](/img/in-post/post-Android/review/context.png)


![](/img/in-post/post-Android/review/context2.png)

- Context是一个场景，代表与操作系统的交互的一种过程。从程序的角度上来理解：Context是个抽象类，而Activity、Service、Application等都是该类的一个实现。
- Context是应用程序环境中全局信息的接口，它整合了许多系统级的服务，可以用来获取应用中的类、资源，以及可以进行应用程序级的调起操作，比如启动Activity、Service等等，而且Context这个类是abstract的，不包含具体的函数实现。
- Activity Context和Application Context的区别是很大的，也就是说，他们的应用场景（你也可以认为是能力）是不同的，并非所有Activity为Context的场景，Application Context都能搞定。

#### Context的应用场景

![](/img/in-post/post-Android/review/context3.png)

- 大家注意看到有一些NO上添加了一些数字，其实这些从能力上来说是YES，但是为什么说是NO呢？下面一个一个解释：
  - 数字1：启动Activity在这些类中是可以的，但是需要创建一个新的task。一般情况不推荐。
  - 数字2：在这些类中去layout inflate是合法的，但是会使用系统默认的主题样式，如果你自定义了某些样式可能不会被使用。
  - 数字3：在receiver为null时允许，在4.2或以上的版本中，用于获取黏性广播的当前值。（可以无视）
- 注：ContentProvider、BroadcastReceiver之所以在上述表格中，是因为在其内部方法中都有一个context用于使用。
- 可以看到，和UI相关的方法基本都不建议或者不可使用Application，并且，前三个操作基本不可能在Application中出现。实际上，只要把握住一点，凡是跟UI相关的，都应该使用Activity做为Context来处理；其他的一些操作，Service,Activity,Application等实例都可以，当然了，注意Context引用的持有，防止内存泄漏。

#### Context的内存泄露

众所周知，Android中的内存泄露，很多数情况都是Context造成的，根据上面对其结构和用途的分析，可以推测一下几点原因：

- 涉及各种系统、APP资源，功能过于强大，导致使用的地方太多，在流程和引用关系上造成混乱，比如：异步的网络请求、View动画等，在流程上处理不当，导致无法释放
- 杂乱无章的传值，让很多对象都Hold一份Context，开辟了过多无法回收的资源
- Drawable对象、Bitmap对象回收不及时，甚至与View死死绑定
- 单例的滥用（单例模式会有静态变量，生命周期与应用周期一致），导致Context长期被引用无法释放
  - 单例对象如果持有Context，很容易引发内存泄露，需要注意传递给单例对象的Context最好是Application Context。

解决方案：

1. 当Application的Context能搞定的情况下，并且生命周期长的对象，优先使用Application的Context。
2. 不要让生命周期长于Activity的对象持有到Activity的引用。
3. 尽量不要在Activity中使用非静态内部类，因为非静态内部类会隐式持有外部类实例的引用， 
   如果使用静态内部类，将外部实例引用作为弱引用持有。

---

### Java的四种引用方式

- java对象的引用包括
  - 强引用，软引用，弱引用，虚引用 
- Java中提供这四种引用类型主要有两个目的：
  - 第一是可以让程序员通过代码的方式决定某些对象的生命周期；
  - 第二是有利于JVM进行垃圾回收。

1. 强引用
   - 是指创建一个对象并把这个对象赋给一个引用变量。（这个对象的实例没有其他对象引用，GC时才会被回收）
   - 比如：
       		Object object = new Object();
       		String str = "hello";
   - 强引用有引用变量指向时永远不会被垃圾回收，JVM宁愿抛出OutOfMemory错误也不会回收这种对象。
   - 如果想中断强引用和某个对象之间的关联，可以显示地将引用赋值为null，这样一来的话，JVM在合适的时间就会回收该对象。
2. 软引用（SoftReference）
   - 如果一个对象具有软引用，内存空间足够，垃圾回收器就不会回收它；
   - 如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。
   - 软引用可用来实现内存敏感的高速缓存,比如网页缓存、图片缓存等。使用软引用能防止内存泄露，增强程序的健壮性。 
   - SoftReference的特点是它的一个实例保存对一个Java对象的软引用， 该软引用的存在不妨碍垃圾收集线程对该Java对象的回收。
   - 也就是说，一旦SoftReference保存了对一个Java对象的软引用后，在垃圾线程对 这个Java对象回收前，SoftReference类所提供的get()方法返回Java对象的强引用。
   - 另外，一旦垃圾线程回收该Java对象之 后，get()方法将返回null。
3. 弱引用（WeakReference）
   - 弱引用也是用来描述非必需对象的，当JVM进行垃圾回收时，无论内存是否充足，都会回收被弱引用关联的对象。在java中，用java.lang.ref.WeakReference类来表示。不过要注意的是，这里所说的被弱引用关联的对象是指只有弱引用与之关联，如果存在强引用同时与之关联，则进行垃圾回收时也不会回收该对象（软引用也是如此）。
4. 虚引用（PhantomReference）
   - 虚引用和前面的软引用、弱引用不同，它并不影响对象的生命周期。在java中用java.lang.ref.PhantomReference类表示。如果一个对象与虚引用关联，则跟没有引用与之关联一样，在任何时候都可能被垃圾回收器回收。
   - 要注意的是，虚引用必须和引用队列关联使用，当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会把这个虚引用加入到与之关联的引用队列中。程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。如果程序发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动。

- 软引用和弱引用
  - 对于强引用，我们平时在编写代码时经常会用到。而对于其他三种类型的引用，使用得最多的就是软引用和弱引用，这2种既有相似之处又有区别。它们都是用来描述非必需对象的，但是被软引用关联的对象只有在内存不足时才会被回收，而被弱引用关联的对象在JVM进行垃圾回收时总会被回收。

#### 如何利用软引用和弱引用解决OOM问题

- 前面讲了关于软引用和弱引用相关的基础知识，那么到底如何利用它们来优化程序性能，从而避免OOM的问题呢？
- 下面举个例子，假如有一个应用需要读取大量的本地图片，如果每次读取图片都从硬盘读取，则会严重影响性能，但是如果全部加载到内存当中，又有可能造成内存溢出，此时使用软引用可以解决这个问题。
- 设计思路是：用一个HashMap来保存图片的路径 和 相应图片对象关联的软引用之间的映射关系，在内存不足时，JVM会自动回收这些缓存图片对象所占用的空间，从而有效地避免了OOM的问题。在Android开发中对于大量图片下载会经常用到。

---

### Android内存泄漏什么情况会发生？

- **Handler**
  - 了解Handler机制的人都明白，但message被Handler send出去的时候，会被加入的MessageQueue中，Looper会不停的从MessageQueue中取出Message并分发执行。但是如果Activity 销毁了，Handler发送的message没有执行完毕。那么Handler就不会被回收，但是由于非静态内部类默认持有外部类的引用。Handler可达，并持有Activity实例那么自然jvm就会错误的认为Activity可达就不行GC。这时我们的Activity就泄漏，Activity作为App的一个活动页面其所占有的内存是不容小视的。那么怎么才能合理的解决这个问题呢
  - 解决方法
    1. 使用弱引用
       - 引用了弱引用就不会打扰到Activity的正常回收。但是在使用之前一定要记得判断弱引用中包含对象是否为空，如果为空则表明表明Activity被回收不再继续防止空指针异常
    2. 使用Handler.removeMessages();
       - 知道原因就很好解决问题，Handler所导致的Activity内存泄漏正是因为Handler发送的Message任务没有完成，所以在onDestory中可以将handler中的message都移除掉，没有延时任务要处理，activity的生命周期就不会被延长，则可以正常销毁。
- **资源对象没关闭造成的内存泄漏**
  - 资源性对象比如(Cursor，File文件等)往往都用了一些缓冲，我们在不使用的时候，应该及时关闭它们，以便它们的缓冲及时回收内存。它们的缓冲不仅存在于 java虚拟机内，还存在于java虚拟机外。如果我们仅仅是把它的引用设置为null,而不关闭它们，往往会造成内存泄漏。因为有些资源性对象，比如 SQLiteCursor(在析构函数finalize(),如果我们没有关闭它，它自己会调close()关闭)，如果我们没有关闭它，系统在回收它时也会关闭它，但是这样的效率太低了。因此对于资源性对象在不使用的时候，应该调用它的close()函数，将其关闭掉，然后才置为null.在我们的程序退出时一定要确保我们的资源性对象已经关闭。
- **构造Adapter时，没有使用缓存的convertView**
  - 以构造ListView的BaseAdapter为例，在BaseAdapter中提供了方法：
        public View getView(int position, ViewconvertView, ViewGroup parent)
    来向ListView提供每一个item所需要的view对象。初始时ListView会从BaseAdapter中根据当前的屏幕布局实例化一定数量的 view对象，同时ListView会将这些view对象缓存起来。当向上滚动ListView时，原先位于最上面的list item的view对象会被回收，然后被用来构造新出现的最下面的list item。这个构造过程就是由getView()方法完成的，getView()的第二个形参View convertView就是被缓存起来的list item的view对象(初始化时缓存中没有view对象则convertView是null)。由此可以看出，如果我们不去使用 convertView，而是每次都在getView()中重新实例化一个View对象的话，即浪费资源也浪费时间，也会使得内存占用越来越大。 
- **试着使用关于application的context来替代和activity相关的context**
- **单例所导致的内存泄漏**
  - 在Android中单例模式中经常会需要Context对象进行初始化，如下简单的一段单例代码示例
        public class MyHelper {
        
          private static MyHelper myHelper;
        
          private Context context;
        
          private MyHelper(Context context){
            this.context = context;
          }
        
          public static synchronized MyHelper getInstance(Context context){
            if (myHelper == null){
              myHelper = new MyHelper(context);
            }
            return myHelper;
          }
        
          public void doSomeThing(){
        
          }
        
        }
  - 这样的写法看起来好像没啥问题，但是一旦如下调用就会产生内存溢出

        public void singleInstanceLeakcanary(){
            MyHelper.getInstance(this).doSomeThing();
          }
  - 首先单例中有一个static实例，实例持有Activity，但是static变量的生命周期是整个应用的生命周期，肯定是会比单个Activity的生命周期长的，所以，当Activity finish时，activity实例被static变量持有不能释放内存，导致内存泄漏。
  - 解决办法：
    1. 使用`getApplicationContext()`

            private void singleInstanceResolve() {
               MyHelper.getInstance(getApplicationContext()).doSomeThing();
             }
    2. 改写单例写法，在Application里面进行初始化。
    
- **（动态）注册没取消造成的内存泄漏**
  - 一些Android程序可能引用我们的Anroid程序的对象(比如注册机制)。即使我们的Android程序已经结束了，但是别的引用程序仍然还有对我们的Android程序的某个对象的引用，泄漏的内存依然不能被垃圾回收。调用registerReceiver后未调用unregisterReceiver。
  - 比如:假设我们希望在锁屏界面(LockScreen)中，监听系统中的电话服务以获取一些信息(如信号强度等)，则可以在LockScreen中定义一个 PhoneStateListener的对象，同时将它注册到TelephonyManager服务中。对于LockScreen对象，当需要显示锁屏界面的时候就会创建一个LockScreen对象，而当锁屏界面消失的时候LockScreen对象就会被释放掉。
  - 但是如果在释放 LockScreen对象的时候忘记取消我们之前注册的PhoneStateListener对象，则会导致LockScreen无法被垃圾回收。如果不断的使锁屏界面显示和消失，则最终会由于大量的LockScreen对象没有办法被回收而引起OutOfMemory,使得system_process 进程挂掉。
  - 虽然有些系统程序，它本身好像是可以自动取消注册的(当然不及时)，但是我们还是应该在我们的程序中明确的取消注册，程序结束时应该把所有的注册都取消掉。

---

### 安卓 final关键字的作用
- 被final修饰的方法不能被重写。
- 当final修饰一个基本数据类型时，表示该基本数据类型的值一旦在初始化后便不能发生变化；如果final修饰一个引用类型时，则在对其初始化之后便不能再让其指向其他对象了，但该引用所指向的对象的内容是可以发生变化的。本质上是一回事，因为引用的值是一个地址，final要求值，即地址的值不发生变化。

---

### 什么是栈内存（heap）和栈内存（heap）

1. 栈：为编译器自动分配和释放，如函数参数、局部变量、临时变量等等
2. 堆：为成员分配和释放，由程序员自己申请、自己释放。否则发生内存泄露。典型为使用new申请的堆内容。
   除了这两部分，还有一部分是：
3. 静态存储区：内存在程序编译的时候就已经分配好，这块内存在程序的整个运行期间都存在。它主要存放静态数据、全局数据和常量。

---

### HTTPS HTTP

- HTTP协议以明文方式发送内容，不提供任何方式的数据加密，如果攻击者截取了Web浏览器和网站服务器之间的传输报文，就可以直接读懂其中的信息，因此，HTTP协议不适合传输一些敏感信息，比如：信用卡号、密码等支付信息。
- HTTPS在HTTP的基础上加入了SSL协议，SSL依靠证书来验证服务器的身份，并为浏览器和服务器之间的通信加密。
- SSL是Netscape开发的专门用户保护Web通讯的，目前版本为3.0。最新版本的TLS 1.0是IETF(工程任务组)制定的一种新的协议，它建立在SSL 3.0协议规范之上，是SSL 3.0的后续版本。两者差别极小，可以理解为SSL 3.1，它是写入了RFC的。

- `HTTPS工作步骤`
  - client向server发送请求https://baidu.com，然后连接到server的443端口。
        - 服务端必须要有一套数字证书，可以自己制作，也可以向组织申请。区别就是自己颁发的证书需要客户端验证通过，才可以继续访问，而使用受信任的公司申请的证书则不会弹出提示页面，这套证书其实就是一对公钥和私钥。
  - 传送证书 
        - 这个证书其实就是公钥，只是包含了很多信息，如证书的颁发机构，过期时间、服务端的公钥，第三方证书认证机构(CA)的签名，服务端的域名信息等内容。
  - 客户端解析证书 
        - 这部分工作是由客户端的TLS来完成的，首先会验证公钥是否有效，比如颁发机构，过期时间等等，如果发现异常，则会弹出一个警告框，提示证书存在问题。如果证书没有问题，那么就生成一个随即值（秘钥）。然后用证书对该随机值进行加密。
  - 传送加密信息 
        - 这部分传送的是用证书加密后的秘钥，目的就是让服务端得到这个秘钥，以后客户端和服务端的通信就可以通过这个随机值来进行加密解密了。
  - 服务段加密信息 
        - 服务端用私钥解密秘密秘钥，得到了客户端传过来的私钥，然后把内容通过该值进行对称加密。
  - 传输加密后的信息 
        - 这部分信息是服务端用私钥加密后的信息，可以在客户端被还原。
  - 客户端解密信息 
        - 客户端用之前生成的私钥解密服务端传过来的信息，于是获取了解密后的内容
- 区别：
  - https协议需要到ca申请证书，一般免费证书很少，需要交费。
  - http是超文本传输协议，信息是明文传输，https 则是具有安全性的ssl加密传输协议。
  - http和https使用的是完全不同的连接方式用的端口也不一样，前者是80，后者是443。
  - http的连接很简单，是无状态的。
  - HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，要比http协议安全。
- 针对无状态的一些解决策略：
  - 通过Cookie/Session技术
  - HTTP/1.1持久连接（HTTP keep-alive）方法，只要任意一端没有明确提出断开连接，则保持TCP连接状态，在请求首部字段中的Connection: keep-alive即为表明使用了持久连接

#### HTTPS 采用混合加密：

- 结合非对称加密和对称加密技术。客户端使用对称加密生成密钥对传输数据进行加密，然后使用非对称加密的公钥再对秘钥进行加密，所以网络上传输的数据是被秘钥加密的密文和用公钥加密后的秘密秘钥，因此即使被黑客截取，由于没有私钥，无法获取到加密明文的秘钥，便无法获取到明文数据。 
- 非对称加密过程需要用到公钥进行加密，那么公钥从何而来？其实公钥就被包含在数字证书中，数字证书通常来说是由受信任的数字证书颁发机构CA，在验证服务器身份后颁发，证书中包含了一个密钥对（公钥和私钥）和所有者识别信息。数字证书被放到服务端，具有服务器身份验证和数据传输加密功能。

---

### 浏览器输入一个网址经历了什么样的过程？

HTTP是一个应用层协议，在这个层的协议，只是一种通讯规范，也就是因为双方要进行通讯，大家要事先约定一个规范。

1. **连接** 当我们输入这样一个请求时，首先要建立一个socket连接，因为socket是通过ip和端口建立的，所以之前还有一个DNS解析过程，把www.mycompany.com变成ip，如果url里不包含端口号，则会使用该协议的默认端口号。 
- **DNS 的过程是这样的**：首先我们知道我们本地的机器上在配置网络时都会填写DNS，这样本机就会把这个url发给这个配置的DNS服务器，如果能够 找到相应的url则返回其ip，否则该DNS将继续将该解析请求发送给上级DNS，整个DNS可以看做是一个树状结构，该请求将一直发送到根直到得到结 果。现在已经拥有了目标ip和端口号，这样我们就可以打开socket连接了。


2. **请求** 连接成功建立后，开始向web服务器发送请求，这个请求一般是GET或POST命令（POST用于FORM参数的传递）。
GET命令的格式为：GET 路径/文件名 HTTP/1.0 文件名指出所访问的文件，HTTP/1.0指出Web浏览器使用的HTTP版本。现在可以发送GET命令：

        GET /mydir/index.html HTTP/1.0，

 
3. **应答** web服务器收到这个请求，进行处理。从它的文档空间中搜索子目录mydir的文件index.html。如果找到该文件，Web服务器把该文件内容传送给相应的Web浏览器。

为了告知浏览器，，Web服务器首先传送一些HTTP头信息，然后传送具体内容（即HTTP体信息），HTTP头信息和HTTP体信息之间用一个空行分开。
常用的HTTP头信息有：
- HTTP 1.0 200 OK 　这是Web服务器应答的第一行，列出服务器正在运行的HTTP版本号和应答代码。代码"200 OK"表示请求完成。
- MIME_Version:1.0　它指示MIME类型的版本。
- content_type:类型　这个头信息非常重要，它指示HTTP体信息的MIME类型。如：content_type:text/html指示传送的数据是HTML文档。
- content_length:长度值　它指示HTTP体信息的长度（字节）。

4. **关闭连接** 当应答结束后，Web浏览器与Web服务器必须断开，以保证其它Web浏览器能够与Web服务器建立连接。


**下面我们具体分析其中的数据包在网络中漫游的经历**

在网络分层结构中，各层之间是严格单向依赖的。“服务”是描述各层之间关系的抽象概念，即网络中各层向紧邻上层提供的一组操作。下层是服务提供者，上层是请求服务的用户。服务的表现形式是原语（primitive），如系统调用或库函数。系统调用是操作系统内核向网络应用程序或高层协议提供的服务原语。网络中的n层总要向n+1层提供比n-1层更完备的服务，否则n层就没有存在的价值。

 
传输层实现的是“端到端”通信，引进网 间进程通信概念，同时也要解决差错控制，流量控制，数据排序（报文排序），连接管理等问题，为此提供不同的服 务方式。通常传输层的服务通过系统调用的方式提供，以socket的方式。对于客户端，要想建立一个socket连接，需要调用这样一些函数 socket() bind() connect(),然后就可以通过send()进行数据发送。


**现在看数据包在网络中的穿行过程：** 

`应用层`

首先我们可以看到在应用层，根据当前的需求和动作，结合应用层的协议，有我们确定发送的数据内容，我们把这些数据放到一个缓冲区内，然后形成了应用层的报文data。


`传输层` 

这些数据通过传输层发送，比如tcp协议。所以它们会被送到传输层处理，在这里报文打上了传输头的包头，主要包含端口号，以及tcp的各种制信息，这些信息是直接得到的，因为接口中需要指定端口。这样就组成了tcp的数据传送单位segment。 tcp是一种端到端的协议，利用这些信息，比如tcp首部中的序号确认序号，根据这些数字，发送的一方不断的进行发送等待确认，发送一个数据段后，会开启 一个计数器，只有当收到确认后才会发送下一个，如果超过计数时间仍未收到确认则进行重发，在接受端如果收到错误数据，则将其丢弃，这将导致发送端超时重 发。通过tcp协议，控制了数据包的发送序列的产生，不断的调整发送序列，实现流控和数据完整。

 
`网络层`

然后待发送的数据段送到网络层，在网络层被打包，这样封装上了网络层的包头，包头内部含有源及目的的ip地址，该层数据发送单位被称为packet。网络层开始负责将这样的数据包在网络上传输，如何穿过路由器，最终到达目的地址。在这里，根据目的ip地址，就需要查找下一跳路由的地址。首先在本机，要查找本机的路由表，在windows上运行route print就可以看到当前路由表内容，有如下几项：
Active Routes              Default Route              Persistent Route.

 
整个查找过程是这样的:
1. 根据目的地址，得到目的网络号，如果处在同一个内网，则可以直接发送。
2. 如果不是，则查询路由表，找到一个路由。
3. 如果找不到明确的路由，此时在路由表中还会有默认网关，也可称为缺省网关，IP用缺省的网关地址将一个数据传送给下一个指定的路由器，所以网关也可能是路由器，也可能只是内网向特定路由器传输数据的网关。
4. 路由器收到数据后，它再次为远程主机或网络查询路由，若还未找到路由，该数据包将发送到该路由器的缺省网关地址。而数据包中包含一个最大路由跳数， 如果超过这个跳数，就会丢弃数据包，这样可以防止无限传递。路由器收到数据包后，只会查看网络层的包裹数据，目的ip。所以说它是工作在网络层，传输层的 数据对它来说则是透明的。

 如果上面这些步骤都没有成功，那么该数据报就不能被传送。如果不能传送的数据报来自本机，那么一般会向生成数据报的应用程序返回一个`“主机不可达”或 “网络不可达”`的错误。



以windows下主机的路由表为例，看路由的查找过程

        Active Routes:
        Network Destination            Netmask                      Gateway              Interface                  Metric
        0.0.0.0                        0.0.0.0                    192.168.1.2            192.168.1.101           10
        127.0.0.0                     255.0.0.0                   127.0.0.1              127.0.0.1                   1
        192.168.1.0                  255.255.255.0                192.168.1.101          192.168.1.101           10
        192.168.1.101                255.255.255.255              127.0.0.1              127.0.0.1                   10
        192.168.1.255                255.255.255.255              192.168.1.101          192.168.1.101           10
        224.0.0.0                     240.0.0.0                  192.168.1.101          192.168.1.101           10
        255.255.255.255              255.255.255.255              192.168.1.101          192.168.1.101           1
        Default Gateway:              192.168.1.2

`Network Destination 目的网段`  
`Netmask 子网掩码`
`Gateway 下一跳路由器入口的ip`，路由器通过interface和gateway定义一调到下一个路由器的链路，通常情况下，interface和gateway是同一网段的。
`Interface 到达该目的地的本路由器的出口ip`（对于我们的个人pc来说，通常由机算机A的网卡，用该网卡的IP地址标识，当然一个pc也可以有多个网卡）。 

网关这个概念，主要用于不同子网间的交互，当两个子网内主机A,B要进行通讯时，首先A要将数据发送到它的本地网关，然后网关再将数据发送给B所在的网关，然后网关再发送给B。
默认网关，当一个数据包的目的网段不在你的路由记录中，那么，你的路由器该把那个数据包发送到哪里！缺省路由的网关是由你的连接上的default gateway决定的，也就是我们通常在网络连接里配置的那个值。

通常interface和gateway处在一个子网内，对于路由器来说，因为可能具有不同的interface,当数据包到达时，根据 Network Destination寻找匹配的条目，如果找到，interface则指明了应当从该路由器的那个接口出去，gateway则代表了那个子网的网关地 址。

第一条      0.0.0.0   0.0.0.0   192.168.1.2    192.168.1.101   10
0.0.0.0代表了缺省路由。该路由记录的意思是：当我接收到一个数据包的目的网段不在我的路由记录中，我会将该数据包通过192.168.1.101 这个接口发送到192.168.1.2这个地址，这个地址是下一个路由器的一个接口，这样这个数据包就可以交付给下一个路由器处理，与我无关。该路由记录 的线路质量 10。当有多个条目匹配时，会选择具有较小Metric值的那个。

第三条      192.168.1.0   255.255.255.0  192.168.1.101   192.168.1.101  10
直联网段的路由记录：当路由器收到发往直联网段的数据包时该如何处理，这种情况，路由记录的interface和gateway是同一个。当我接收到一个 数据包的目的网段是192.168.1.0时，我会将该数据包通过192.168.1.101这个接口直接发送出去，因为这个端口直接连接着 192.168.1.0这个网段，该路由记录的线路质量 10 （因interface和gateway是同一个，表示数据包直接传送给目的地址，不需要再转给路由器）。


一般就分这两种情况，目的地址与当前路由器接口是否在同一子网。如果是则直接发送，不需再转给路由器，否则还需要转发给下一个路由器继续进行处理。

查找到下一跳ip地址后，还需要知道它的mac地址，这个地址要作为链路层数据装进链路层头部。这时需要arp协议，具体过程是这样的，查找arp 缓冲，windows下运行arp -a可以查看当前arp缓冲内容。如果里面含有对应ip的mac地址，则直接返回。否则需要发生arp请求，该请求包含源的ip和mac地址，还有目的地 的ip地址，在网内进行广播，所有的主机会检查自己的ip与该请求中的目的ip是否一样，如果刚好对应则返回自己的mac地址，同时将请求者的ip mac保存。这样就得到了目标ip的mac地址。

`链路层`

将mac地址及链路层控制信息加到数据包里，形成Frame，Frame在链路层协议下，完成了相邻的节点间的数据传输，完成连接建立，控制传输速度，数据完整。

`物理层`

物理线路则只负责该数据以bit为单位从主机传输到下一个目的地。

下一个目的地接受到数据后，从物理层得到数据然后经过逐层的解包 到 链路层 到 网络层，然后开始上述的处理，在经网络层 链路层 物理层将数据封装好继续传往下一个地址。

在上面的过程中，可以看到有一个路由表查询过程，而这个路由表的建立则依赖于路由算法。也就是说路由算法实际上只是用来路由器之间更新维护路由表， 真正的数据传输过程并不执行这个算法，只查看路由表。这个概念也很重要，需要理解常用的路由算法。而整个tcp协议比较复杂，跟链路层的协议有些相似，其 中有很重要的一些机制或者概念需要认真理解，比如编号与确认，流量控制，重发机制，发送接受窗口。

---

### Activity 与 Window 与 View 之间的关系  
 
https://blog.csdn.net/zane402075316/article/details/69822438

![](/img/in-post/post-Android/review/activityWindowView.png)

1. 为什么要设计Activity、View、Window？
    - 答：Activity就像工匠，Window就像是窗户，View就像是窗花，LayoutInflater像剪刀，Xml配置像窗花图纸。 
    - Android根据他们不同的职能让他们各斯其活，同时也相互配合展示给我们灵活、精致的界面。为啥这样设计？因为这样的结构更好管理。就像为啥需要使用MVP、MVVM、各种设计模式一样。

2. Activity工作过程是什么样的？ 
    - 答：以Activity启动过程为例，Activity启动时是通过Binder向AMS(ActivityManagerService)发请求，通过PIC启动Activity的。

3. Window是什么？它的职能是什么？ 
    - 答：Activity要管理View需要通过Window来间接管理的。Window通过addView()、removeView()、updateViewLayout()这三个方法来管理View的。

4. View跟Window有什么联系？ 
    - 答：View需要通过Window来展示在Activity上。

5. Activity、View、Window三者如何关联？ 
    - 答：Activity包含了一个PhoneWindow，而PhoneWindow就是继承于Window的，Activity通过setContentView将View设置到了PhoneWindow上。Window的添加过程以及Activity的启动流程都是一次IPC的过程。Activity的启动需要通过AMS完成；Window的添加过程需要通过WindowSession完成。

---
### View实现等待加载的转圈动画效果

```
private void init() {
        mPaint = new Paint();
        mPaint.setAntiAlias(true);
        mPaint.setStyle(Paint.Style.FILL);
        mPaint.setColor(color);
 
        mDensity = getResources().getDisplayMetrics().density;
    }
 
    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mStartX = getWidth()/2;
        mEndY = getHeight()/2;
        mStartY = mEndY*5/6;
        playAnimator();
    }
 
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        canvas.drawCircle(mStartX,mCurrentY,mDensity*radius,mPaint);
    }
 
    private void playAnimator() {
 
        ValueAnimator animator = ValueAnimator.ofInt(mStartY,mEndY);
        animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator valueAnimator) {
                mCurrentY= (int) valueAnimator.getAnimatedValue();
               invalidate();
            }
        });
        animator.setRepeatMode(ValueAnimator.REVERSE);
        animator.setRepeatCount(-1);
        animator.setDuration(1000);
        animator.start();
    }
```
---
### scaleType各属性效果

scaleType的属性有`matrix（默认）、center、centerCrop、centerInside、fitCenter、fitEnd、fitStart、fitXY`。

![](/img/in-post/post-Android/review/scaletype.png)

- android:scaleType="`center`"
    - 保持原图的大小，显示在ImageView的中心。当原图的size大于ImageView的size，超过部分裁剪处理。

- android:scaleType="`centerCrop`"
    - 以填满整个ImageView为目的，将原图的中心对准ImageView的中心，等比例放大原图，直到填满ImageView为止（指的是ImageView的`宽和高都要填满`），原图超过ImageView的部分作裁剪处理。

- android:scaleType="`centerInside`"
    - 以原图完全显示为目的，将图片的内容完整居中显示，通过按比例缩小原图的size宽(高)等于或小于ImageView的宽(高)。如果原图的size本身就小于ImageView的size，则原图的size不作任何处理，居中显示在ImageView。

- android:scaleType="`matrix`"
    - 不改变原图的大小，从ImageView的左上角开始绘制原图，原图超过ImageView的部分作裁剪处理。

- android:scaleType="`fitCenter`"
    - 把原图按比例扩大或缩小到ImageView的ImageView的高度/宽度，居中显示

- android:scaleType="`fitEnd`"
    - 把原图按比例扩大(缩小)到ImageView的高度/宽度，显示在ImageView的下部分位置

- android:scaleType="`fitStart`"
    - 把原图按比例扩大(缩小)到ImageView的高度/宽度，显示在ImageView的上部分位置

- android:scaleType="`fitXY`"
    - 把原图按照指定的大小在View中显示，拉伸显示图片，不保持原比例，填满ImageView

注：
`Center`: 图片大小＝原始图片大小；
`fitCenter`： 图片大小＝View大小；
`CenterInside`： 图片大小 <= View大小 && 图片大小 <= 原始图片大小。

---
### 实现双指放大缩小图片
使用canvas或者简单使用Matrix + imageView 即可实现。
为imageView添加触摸事件监听器，并监听双指触摸事件。

**关于触摸事件的各个属性值：**
- `ACTION_DOWN` is for the first finger that touches the screen. This starts the gesture. The pointer data for this finger is always at index 0 in the MotionEvent.
- `ACTION_POINTER_DOWN` is for extra fingers that enter the screen beyond the first. The pointer data for this finger is at the index returned by getActionIndex().
- `ACTION_POINTER_UP` is sent when a finger leaves the screen but at least one finger is still touching it. The last data sample about the finger that went up is at the index returned by getActionIndex().
- `ACTION_UP` is sent when the last finger leaves the screen. The last data sample about the finger that went up is at index 0. This ends the gesture.
- `ACTION_CANCEL` means the entire gesture was aborted for some reason. This ends the gesture.


```java
imageView.setImageMatrix(matrix);
imageView.setOnTouchListener(new View.OnTouchListener() {
    @Override
    public boolean onTouch(View v, MotionEvent event) {
        ImageView view = (ImageView) v;
        switch (event.getAction() & MotionEvent.ACTION_MASK) {
            //单个手指触摸
            case MotionEvent.ACTION_DOWN:
                mode = 1;
                break;
            //两指触摸
            case MotionEvent.ACTION_POINTER_DOWN:
                preDistance = getDistance(event);
                //当两指间距大于10时，计算两指中心点
                if (preDistance > 10f) {
                    mid = getMid(event);
                    savedMatrix.set(matrix);
                    mode = 2;
                }
                break;
            case MotionEvent.ACTION_UP:
                mode = 0;
                break;
            case MotionEvent.ACTION_POINTER_UP:
                mode = 0;
                break;
            case MotionEvent.ACTION_MOVE:
                //当两指缩放，计算缩放比例
                if (mode == 2) {
                    distance = getDistance(event);
                    if (distance > 10f) {
                        matrix.set(savedMatrix);
                        float scale = distance / preDistance;
                        matrix.postScale(scale, scale, mid.x, mid.y);//缩放比例和中心点坐标
                    }
                }
                break;
        }
        view.setImageMatrix(matrix);
```

通过`event.getPointerCount()`：
```
mImage.setOnTouchListener(new OnTouchListener() {
    float baseValue;
    @Override
    public boolean onTouch(View v, MotionEvent event) {
    // TODO Auto-generated method stub
    // return ArtFilterActivity.this.mGestureDetector.onTouchEvent(event);
    if (event.getAction() == MotionEvent.ACTION_DOWN) {
        baseValue = 0;
        float x = last_x = event.getRawX();
        float y = last_y = event.getRawY();
    } 
    else if (event.getAction() == MotionEvent.ACTION_MOVE) {
        if (event.getPointerCount() == 2) {
            float x = event.getX(0) - event.getX(1);
            float y = event.getY(0) - event.getY(1);
            float value = (float) Math.sqrt(x * x + y * y);// 计算两点的距离
            if (baseValue == 0) {
                baseValue = value;
            } 
            else {
                if (value - baseValue >= 10 || value - baseValue <= -10) {
                    float scale = value / baseValue;// 当前两点间的距离除以手指落下时两点间的距离就是需要缩放的比例。
                    img_scale(scale);  //缩放图片
                }
            }
        } 
        else if (event.getPointerCount() == 1) {
            float x = event.getRawX();
            float y = event.getRawY();
            x -= last_x;
            y -= last_y;
            if (x >= 10 || y >= 10 || x <= -10 || y <= -10)
                img_transport(x, y); //移动图片位置
                last_x = event.getRawX();
                last_y = event.getRawY();
            }
        } 
        else if (event.getAction() == MotionEvent.ACTION_UP) {
 
        }
        return true;
    }
});
```
