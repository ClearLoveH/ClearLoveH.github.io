---
layot:     post
title:      "中山大学智慧健康服务平台应用开发"
subtitle:   "Broadcast 使用"
date:       2018-10-22 14:15
author:     "Heng"
header-img: "img/弗雷尔卓德2.jpg"
catalog: true
tags:
    - Android
---

>Royal may give up now..

---

[项目仓库](https://gitee.com/ClearLoveH/PersonalProject2)

---


## 一、实验题目 

### Broadcast以及EventBus的使用

---

## 二、实现内容

在第六周任务的基础上，实现静态广播、动态广播两种改变Notification 内容的方法。
- 实验要求
    - 在启动应用时，会有通知产生，随机推荐一个食品。
    - 点击通知跳转到所推荐食品的详情界面。
    - 点击收藏图标，会有对应通知产生，并通过Eventbus在收藏列表更新数据。
    - 点击通知返回收藏列表。
实现方式要求:启动页面的通知由静态广播产生，点击收藏图标的通知由动态广播产生。
- 实验目的
    - 掌握 Broadcast 编程基础。
    - 掌握动态注册 Broadcast 和静态注册 Broadcast。
    - 掌握Notification 编程基础。
    - 掌握 EventBus 编程基础。


---

## 三、课堂实验结果
### (1)实验截图

>本次实验所用虚拟机系统为安卓7.0，所以在通知的样式上略有不同，但是在6.0安卓下运行样式是没有差别的。

- 启动应用时，会有通知产生，随机推荐一个食品。
![](/img/in-post/post-Android/lab2/截图/lab2_week7_1.png)

- 点击通知跳转到所推荐食品的详情界面。
![](/img/in-post/post-Android/lab2/截图/lab2_week7_2.png)

- 点击收藏图标，会有对应通知产生，并通过Eventbus在收藏列表更新数据。
![](/img/in-post/post-Android/lab2/截图/lab2_week7_3.png)
![](/img/in-post/post-Android/lab2/截图/lab2_week7_4.png)

- 点击通知返回收藏列表。
![](/img/in-post/post-Android/lab2/截图/lab2_week7_5.png)

### (2)实验步骤以及关键代码

1. 启动应用时产生的通知，使用的是静态广播。
    - 创建一个静态广播类。
    - 主页在进入时就产生一个随机的推荐，我们需要在`OnCreate()`函数中发送广播，添加产生的随机的一个Collection，然后把这个实参传给接收器`StaticReceiver`类。
        ```java
        //broadcastBundle 静态广播
        Random random = new Random();  //随机数生成
        int randomInt=random.nextInt(data.size());
        Bundle broadcastBundle=new Bundle();            
        broadcastBundle.putInt("index",randomInt);
        broadcastBundle.putSerializable("foodItem",  data.get(randomInt));
        Intent intentBroadcast = new Intent("android.intent.action.StaticReceiver"); //定义Intent
        intentBroadcast.putExtras(broadcastBundle);
        sendBroadcast(intentBroadcast);
        ```

    - 实现静态广播接收器类`StaticReceiver`，分为两个部分
        - 接受发送的广播及其传来实参，然后显示在通知中。
            ```java
            public class StaticReceiver extends BroadcastReceiver {
                private static final String STATICACTION = "android.intent.action.StaticReceiver";
                @Override
                public void onReceive(Context context, Intent intent) {
                    if (intent.getAction().equals(STATICACTION)){
                        Bundle bundle = intent.getExtras();
                        //TODO:添加Notification部分
                        Collection temp= (Collection) intent.getSerializableExtra("foodItem");
                        NotificationManager manager=(NotificationManager)context.getSystemService(Context.NOTIFICATION_SERVICE);
                        Notification.Builder builder=new Notification.Builder(context);
                        builder.setContentTitle("今日推荐")
                                .setContentText(temp.getName())
                                .setSmallIcon(R.drawable.empty_star)
                                .setWhen(System.currentTimeMillis())
                                .setAutoCancel(true);
                        ···
                        ···
            ```
            - 在接收器类中，同时还需要实现点击通知跳转至对应的详情界面，`foodItem`是详情页中接受的Intent的一个参数，该参数是前面实验使用过的，因为这里使用的是与前面实验的相同的处理参数的方式。
                ```java
                        //点击转到详情界面
                        Intent clickIntent=new Intent(context,detailActivity.class);
                        Bundle clickBundle = new Bundle();
                        clickBundle.putSerializable("foodItem",  temp);
                        clickIntent.putExtras(clickBundle);
                        PendingIntent clickPendingIntent =PendingIntent.getActivity(context,0,clickIntent,PendingIntent.FLAG_UPDATE_CURRENT);
                        builder.setContentIntent(clickPendingIntent);
                        //绑定Notification，发送通知请求
                        Notification notify=builder.build();
                        manager.notify(0,notify);
                ```
        - 在AndroidMainfest.xml 中进行注册。
            ```xml
            <receiver android:name=".StaticReceiver">
            <intent-filter>
                <action android:name="android.intent.action.StaticReceiver" />
            </intent-filter>
            ```

2. 点击详情页的收藏时产生的通知的实现，使用的是动态广播。
    - 与静态广播不同，我们使用动态广播时不需要在Manifest对其进行注册，而是在发送广播的位置对其进行相应的注册。
    - 发送广播，在这里发送广播的位置是详情页点击收藏按钮的事件中。
        ```java
        //动态广播
            IntentFilter dynamic_filter = new IntentFilter();
            //添加动态广播的Action
            dynamic_filter.addAction("android.intent.action.DynamicReceiver");

            //注册自定义动态广播消息
            registerReceiver(dynamicReceiver, dynamic_filter);

            //定义Intent
            Intent dynamic_intent = new Intent();
            dynamic_intent.setAction("android.intent.action.DynamicReceiver");
            //注意此处不能使用   putExtras  也是前面使用静态通知时的错误
            dynamic_intent.putExtra("name",backFoodItem);
            sendBroadcast(dynamic_intent);
        ```
    - 在退出详情页时，对广播一并进行注销
        ```java
            //注销广播
            @Override
            protected void onDestroy() {
                super.onDestroy();
                unregisterReceiver(dynamicReceiver);
            }
            ```
    - 动态广播接收器，这里接收器的实现与前面静态广播接收器无差，也是分为两个部分：1.接收广播并显示，2.点击通知页面跳转：
        - 接收广播
            ```java
            public void onReceive(Context context, Intent intent) {
            if (intent.getAction().equals(DYNAMICACTION)) {    //动作检测
                Bundle bundle = intent.getExtras();
                //TODO:添加Notification部分
                NotificationManager manager=(NotificationManager)context.getSystemService(Context.NOTIFICATION_SERVICE);
                Notification.Builder builder=new Notification.Builder(context);
                builder.setContentTitle("已收藏")
                        .setContentText(intent.getStringExtra("name"))
                        .setSmallIcon(R.drawable.empty_star)
                        .setWhen(System.currentTimeMillis())
                        .setAutoCancel(true);
            ```
        - 点击通知页面跳转，与静态广播不同，我没有将一整个Collection作为参数传给主页，而且仅传递了一个bool变量`returnFromBroadcast`，来声明该次切换至主页的收藏夹是从广播切换过去的，而对于具体的哪个Item我们使用EventBus来传递。
            ```java
            //点击转到详情界面
            Intent clickIntent=new Intent(context,MainActivity.class);
            Bundle clickBundle = new Bundle();
            clickBundle.putBoolean("returnFromBroadcast", true);
            clickIntent.putExtras(clickBundle);
            PendingIntent clickPendingIntent =PendingIntent.getActivity(context,0,clickIntent,PendingIntent.FLAG_UPDATE_CURRENT);
            builder.setContentIntent(clickPendingIntent);
            //绑定Notification，发送通知请求
            Notification notify=builder.build();
            manager.notify(0,notify);
        - 对比可以发现，接收器的实现上动静态广播两者差别是不大的，主要差别是在发送广播端的处理，最后与前面静态广播切换页面的方式不同，这里我们需要使用EventBus来进行页面切换时的参数传递。

3. 使用EventBus来进行参数的传递，每点击通知一次，添加对应的一个食品到收藏列表并产生一条通知。EventBus于此取代Intent与Bundle来进行页面切换时的传参问题。
    - EventBus添加依赖
        ```
        compile 'org.greenrobot:eventbus:3.0.0'
        ```
    - 声明一个事件类，用于传递食品信息
        ```java
        public class collectEvent {
            public String name;
            public collectEvent(String name) {
                this.name = name;
            }
            public String getName() {
                return name;
            }
            public void setName(String name) {
                this.name = name;
            }
        }
        ```
    - 在主页`MainActivity`中，准备订阅者
        1. 在收藏列表所在的MainActivity中声明并注释订阅方法，可选地指定线程模式，接收到发布者发布的事件后，进行相应的处理操作——把该Collection添加进收藏夹。

            ```java
            @Subscribe(threadMode = ThreadMode.MAIN)
            public void onEventMainThread(collectEvent temp){
                //接收到发布者发布的事件后，进行相应的处理操作
                for(int i=0;i < this.data.size();i++){
                    if(this.data.get(i).getName().equals(temp.getName())){
                        this.data2.add(this.data.get(i));
                        myListViewAdapter.notifyDataSetChanged();
                        break;
                    }
                }
            }
            ```

        2. 注册收藏列表所在的MainActivity为订阅者

            ```java
            //注册订阅者
            EventBus.getDefault().register(this);
            ```

        3. 注销订阅者，退出时要注销订阅者，重写详情页的`OnDestroy`函数，在主页MainActivity退出时，订阅一并取消
            ```java
            @Override
            protected void onDestroy() {
                super.onDestroy();
                EventBus.getDefault().unregister(this);
            }
            ```
            
    - 在详情页`detailActivity`中，点击收藏按钮时，发布事件并传递食品信息。
        ```java
        backFoodItem= ((TextView)findViewById(R.id.foodName)).getText().toString();
        ···
        ···
        //发布事件
        EventBus.getDefault().post(new collectEvent(backFoodItem));
        ```

    

### (3)实验遇到的困难以及解决思路

这次广播实验遇到的主要问题就一个：
- intent的`putExtra`与`putExtras`
    - 动态广播传递参数的过程中，在详情页处只传递了一个String参数作为"name"的变量传回给主页，然后主页接收了name再进行处理，添加到收藏夹中。
        ```java
        dynamic_intent.putExtra("name",backFoodItem);
        ```
    这里由于只传了一个参数，所以我没有用到bundle，但是一开始我在这里依旧使用了`putExtras`函数，有无s这个非常细微的差别却导致了我没有把name传给主页，在debug到是此处出错的时候，我查了一下两个函数的区别，终于发现问题——`public Intent putExtra (String name, double[] value)`与`public Intent putExtras (Bundle extras)`——putExtras中的参数是bundle而不能直接使用某些变量传递过去，传具体的变量时使用的应该是petExtra函数。

---


## 四、实验思考及感想

实验中的思考与感悟：

- 这次实验是在上周任务基础上来完成的，在点击静态广播页面切换是参数的传递过程中，我使用的依旧是Intent与Bundle，而传递过去的参数之后，由于对参数的使用（把Collection的具体信息呈现在详情页中）与之前呈现方式是无差的，所以我这里Intent传递过去的参数名与前面主页点击Item进入详情页时是一样的，目的就是为了避免冗余的代码，使用已有代码来完成相同或者相差无几的任务，这样清晰的思路是非常有利于项目的完成的。   

- 而对于动态博客的参数传递，这次实验要求我们实使用EventBus来实现，使用EventBus的步骤相对Intent与Bundle是比较多的——`声明信息类，声明订阅方法，注册订阅者，发布事件，订阅完成事件，最后注销订阅。`使用了EventBus之后，我便取消掉`onActivityResult`方法，取而代之是在订阅方法`onEventMainThread`中把该传的数据传过去。
- 虽然步骤较多，但是整体使用起来的思路是比较清楚的，因为EventBus把在不同的Actibity中传递数据的这个过程分为给不同的角色去完成，每个角色都有其各自的任务，分工清楚且代码简单，理解了之后使用起来是非常方便的，这个用于取代Intent与Bundle也是非常可行的，希望后面的学习中我能够学以致用，活学活用，做出更好的项目。


---

