---
layot:     post
title:      "中山大学智慧健康服务平台应用开发"
subtitle:   "AppWidget 使用"
date:       2018-10-28 10:36
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

### AppWidget 使用

---

## 二、实现内容

在第七周任务的基础上，实现静态广播、动态广播两种改变widget内容的方法。
- 要求
    - widget初始情况如下：
        ![](/img/in-post/post-Android/lab2/截图/lab2_week8_样式1.png)
    - 点击widget可以启动应用，并在widget随机推荐一个食品。

    - 点击widget跳转到所推荐食品的详情界面。

    - 点击收藏图标，widget相应更新。

    - 点击widget跳转到收藏列表。

    - 实现方式要求:启动时的widget更新通过`静态广播`实现，点击收藏图标时的widget更新通过`动态广播`实现。

- 实验目的
    - 复习 Broadcast 编程基础。
    - 复习动态注册 Broadcast 和静态注册 Broadcast 。
    - 掌握 AppWidget 编程基础。

---

## 三、课堂实验结果
### (1)实验截图

>本次实验widget的静态广播我没有放在OnCreate()函数中，而是放在了`OnRestart()`函数里，即启动应用后点击home键再返回应用即可发送静态广播，便于TA们的检查。

- widget初始情况如下：
    ![](/img/in-post/post-Android/lab2/截图/lab2_week8_1.png)
    - 点击Home键之后，再返回应用：
        ![](/img/in-post/post-Android/lab2/截图/lab2_week8_2.png)
    - widget更新，随机推荐了一个食品：
        ![](/img/in-post/post-Android/lab2/截图/lab2_week8_3.png)
    - 且与上一周任务所做的静态通知相符合：
    ![](/img/in-post/post-Android/lab2/截图/lab2_week8_4.png)

- 点击widget可以启动应用，跳转到所推荐食品的详情界面。
    ![](/img/in-post/post-Android/lab2/截图/lab2_week8_5.png)

- 点击收藏图标，widget相应更新。
    ![](/img/in-post/post-Android/lab2/截图/lab2_week8_6.png)
    ![](/img/in-post/post-Android/lab2/截图/lab2_week8_7.png)
    - 且与上周所做的动态广播保持一致
        ![](/img/in-post/post-Android/lab2/截图/lab2_week8_8.png)
- 点击widget跳转到收藏列表。
    ![](/img/in-post/post-Android/lab2/截图/lab2_week8_9.png)

### (2)实验步骤以及关键代码

1. Widget静态广播——用于启动应用时显示今日推荐

    - 静态广播的发送端，我由OnCreate()函数移至了`OnRestart()`函数中，这也算是本次实验一个我自己的想法，这样做的目的是实现应用刚打开并不直接更新Widget，等到再次进入时才发送静态广播(这里涉及到了应用的生命周期的知识点，在最后感悟部分进行总结)：
        ```java
            //widget在按了home键后重新进入应用时才发送静态广播
            @Override
            protected void onRestart() {
                super.onRestart();  // Always call the superclass method first
                //action的设置出现过问题——android.appwidget.action.APPWIDGET_UPDATE不能用于此使用，是系统自用的action
                //这个广播要记得需要静态的注册android.appwidget.action.APPWIDGET_UPDATE广播，如果你不添加这个就无法在挂件栏里面看到这个挂件，也就无法添加新的挂件
                Intent widgetIntentBroadcast = new Intent("com.example.lab2.myWidget");
                Bundle widgetBundle=new Bundle();
                widgetBundle.putSerializable("foodItemForWidget",  data.get(randomInt));
                widgetIntentBroadcast.putExtras(widgetBundle);
                sendBroadcast(widgetIntentBroadcast);
                // Activity being restarted from stopped state
            }
            ```
    - 静态广播的接收端，是通过重写`myWidget.java`文件中的`OnReceive()`函数来实现的，此次先给Widget添加一个默认的点击的PendingIntent，目的是实现第一次点击Widget只是打开应用，然后才更新Widget为今日推荐，冗余部分代码于此省去，Intent与Bundle的使用还有页面跳转都与上周任务无差，主要是`RemoteViews`的新知识点`（主要RequestCode的设置）`。
        ```java
        public void onReceive(Context context, Intent intent) {
            super.onReceive(context, intent);
            AppWidgetManager appWidgetManager = AppWidgetManager.getInstance(context);
            RemoteViews view = new RemoteViews(context.getPackageName(),R.layout.my_widget);
            //应用刚启动时，点击widget只用于启动应用，不进行跳转
            //点击转到详情界面
            Intent clickIntent=new Intent(context,MainActivity.class);
            Bundle clickBundle=new Bundle();
            clickBundle.putBoolean("returnFromWidget", false);
            clickIntent.putExtras(clickBundle);
            //requestCode !!! 此处requestCode不能与DynamicReceiver中的相同，否则MainActivity在返回判断这件事情时会把两个不同的动作当作同一个事件。
            PendingIntent pendingIntent = PendingIntent.getActivity(context, 1, clickIntent, 0);
            view.setOnClickPendingIntent(R.id.myWidget, pendingIntent);
            appWidgetManager.updateAppWidget(new ComponentName(context, myWidget.class), view);
            if (intent.getAction().equals("com.example.lab2.myWidget")) {
                Collection temp= (Collection) intent.getSerializableExtra("foodItemForWidget");
                String widgetName ="今日推荐 "+ temp.getName();
                view.setTextViewText( R.id.appwidget_text, widgetName);

                //点击转到详情界面
                ···
                ···
                ···
            }
        }
        ```

2. Widget动态广播——用于点击收藏时自动更新Widget
    - 动态广播的注册：
        ```java
        //widget的动态广播注册
        IntentFilter widget_dynamic_filter = new IntentFilter();
        widget_dynamic_filter.addAction("com.example.lab2.WidgetDynamicReceiver");
        registerReceiver(dynamicReceiverWidget, widget_dynamic_filter); //注册自定义动态广播信息
        ```
    - 动态广播的发送端，依旧是设置在详情页的点击收藏按钮的事件中，发送动态广播与上周的也是一模一样，注意新action的注册即可:
        ```java
        //发送widget动态广播
            Intent widgetDynamicIntent=new Intent("com.example.lab2.WidgetDynamicReceiver");
            widgetDynamicIntent.putExtra("name",backFoodItem);
            sendBroadcast(widgetDynamicIntent);
        ```
    - 动态广播的接收端，我们在`DynamicReceiver.java`，文件中进行修改，`OnReceive()`函数添加新的判断条件，用于获取Widget的动态广播，区别于上周的动态广播通知，设置上与静态广播相似，页面跳转使用的依旧是上周的相同事件，无需太多其他处理。
        ```java
        private static final String WIDGETDYNAMICACTION="com.example.lab2.WidgetDynamicReceiver";
        ```
        ```java
        else if(intent.getAction().equals(WIDGETDYNAMICACTION)){ //检测widget动态广播动作
            AppWidgetManager appWidgetManager = AppWidgetManager.getInstance(context);
            RemoteViews view = new RemoteViews(context.getPackageName(),R.layout.my_widget);
            String widgetCollectName ="已收藏 "+ intent.getStringExtra("name");
            //点击跳转到收藏夹
            Intent clickIntent = new Intent(context, MainActivity.class);
            Bundle clickBundle = new Bundle();
            clickBundle.putBoolean("returnFromBroadcast", true);
            clickIntent.putExtras(clickBundle);
            PendingIntent pendingIntent = PendingIntent.getActivity(context, 0, clickIntent, 0);
            view.setOnClickPendingIntent(R.id.myWidget, pendingIntent);
            view.setTextViewText( R.id.appwidget_text, widgetCollectName);

            appWidgetManager.updateAppWidget(new ComponentName(context, myWidget.class), view);
        }
        ```
- 本次实验大体过程就是如此，相对上周较为轻松，基于上周的大部分的代码，完成难度不大，但是做项目的过程中我却遇到几个问题。

### (3)实验遇到的困难以及解决思路

这次广播实验遇到的主要问题有两个：
- `android.appwidget.action.APPWIDGET_UPDATE`问题
    - 在发送广播的时候，action的设置出现过问题，我使用了生成Widget时，自动注册的`android.appwidget.action.APPWIDGET_UPDATE`，后来发现他不能给自己定义的action来使用，这个是系统自用的action，系统对其自有用途。
    - android.appwidget.action.APPWIDGET_UPDATE这个是广播静态的注册，如果你不添加这个就无法在挂件栏里面看到这个挂件，也就无法添加新的挂件

- `requestCode`问题
    - 在Widget的java文件中，重写OnReceive()函数时，我为了使第一次点击Widget只启动应用，直接预先给Widget添加了一个PendingIntent来实现此效果，为了区分前一周点击动态广播通知返回主页并显示收藏夹，我在这个默认的PendingIntent中特意设置了一个Bool值:
        ```java
        clickBundle.putBoolean("returnFromWidget", false);
        ```
    这个Bool值是我在之前点击动态广播通知返回主页时，让MainActivity判断出是从哪里返回过来主页的，这里很明显应该设置为false，但是我发现第一次点击Widget打开主页时，却直接显示了收藏夹，也就意味着此Bool值并没有用，经过检查发现，是`getActivity()`这里出现了问题，对比`DynamicReceiver.java`与`myWidget.java`发现，两者的`getActivity()`函数是一样的：
    ```java
        PendingIntent pendingIntent = PendingIntent.getActivity(context, 0, clickIntent, 0);
    ```
    第二个参数是`requestCode`，用于标识请求来源，我把两者都设为0，返回主页时，主页时无法判断两者的区别的，尽管我把返回的Bool值设为了false，依旧无效，改为其他的任意未被占用的requestCode，此问题都可以解决。

---


## 四、实验思考及感想

实验中的思考与感悟：

- 这次实验依旧是在上周任务基础上来完成的，主要知识点还是建立在上周的Intent与Bundle之上，只是添加了一个新的知识点`RemoteView`——RemoteViews是用来描述一个视图的，它描述的这个视图将显示在另外一个进程中。但是对于PendingIntent的使用还是非常熟悉的。所以 任务相对来说比较轻松。
- 但是由于自己的“懒”，我反而给自己增加了工作量，由于这次实验与上次实验代码有许多相似之处，所以在Intent的设置上，Widget的动静态广播都是修改自上周代码而来，在requestCode的设置上也没有改变导致最终debug了一会。但是我也因此对requestCode的了解更多了，后续对其的应用也会更加的灵活，新的动作我们还是需要赋予新的requestCode。
- 另外的一处“懒”，便是我懒得注册新的action，我直接使用了Widget自动生成的静态注册，导致应用启动时就直接崩溃，这里我也花了不少时间来debug，也是给我自己好好的上了一课，以后写代码一定要更加规范合理，按步骤来，不要盲目搞自己的想法，才能打出更好的项目。
- 在使用到`OnRestart()`函数时，我对安卓中应用程序的生命周期也有了许多的了解，所以现在在看OnCreate()、OnRestart()这些函数也是豁然开朗：

    对于应用程序简单的点击Home与返回应用的操作，其后台的执行情况是这样的：
    1. 启动这个应用执行如1；onCreate()--->onStart()--->onResume()这三个方法
    2. 按下home键执行如2；onPause()--->onStop()这两个方法
    3. 再次打开这个应用执行如3；onRestart()--->onStart()--->onResume()三个方法。
- 在了解了程序生命周期之后，再进行相应的编程就变得非常简单，onRestart()函数的使用也得心应手，后面再遇到程序生命周期的问题时，我还可以更加灵活，需要学习的知识点依旧还有很多，继续努力。

---

