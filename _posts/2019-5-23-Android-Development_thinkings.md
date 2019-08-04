---
layout:     post
title:      "Android 拓展思考"
date:       2019-5-23 11:47
author:     "Heng"
header-img: "img/比尔吉沃特1.jpg"
catalog: true
tags:
    - Android
---


### 微信朋友圈小红点实现


### QQ移动端查看了新信息后，PC端便取消提醒


---
### Android 10.0
1. 更好的隐私控制
    - 在Android Q 中，Google 加入更多新的隐私权限，有近50 项新功能和动项目与安全性与隐私权保护机制相关，让你能更容易控管与应用程序分享的信息，像是应用程序必须获得你的授权，才能在背景追踪你的位置，或是只能在使用期间侦测位置。另外也能设定应用程序，对照片、影片与音乐的访问权限。
2. 暗黑模式
    - 跟 iOS 一样，在今年的版本中终于加入原生的暗黑模式了。这功能似乎可以在快速控制介面中快速切换，而Google 也建立一个新的API，让开发人员可以设计当应用程序开启时，可改变成暗黑主题，也就是App界面也会变成深色配色。
3. 即时转录功能
    - Android Q 将新增一个Live Caption 功能，使用者按音量键的上或下，待出现音量调整杆，就会看到这功能图示，这功能将能实现即使转录效能，当你在播放任何应用程序的影片、Podcasts、音讯、甚至是个人录制内容，打开这功能，只要播放的内容被侦测到，就会自动产生字幕，而且不需要另外开启应用程序，或使用Wi-Fi 网路或手机的行动数据。
4. Smart Reply
    - 通过机器学习技术，Smart Replay 将能实现自动显示建议回覆的效果，而且是所有通讯应用程序都支持，另外也会聪明预测使用者要执行的动作，如下方的GIF 档，当有人传送地址给你时，下方就会出现使用Google 地图开启的功能选项，让你快速查看实际位置。
5. 自带屏幕录制功能
    - Android Q也要支持内置屏幕录制功能了!虽然现在已经有很多App可以实现，但既然 Android 直接内置，应该没有人不喜欢吧?
6. 原生桌面模式
    - Android Q 手机连接外接屏幕时，会自动切换到桌面模式，实现多应用程序视窗开启、操作的情境，底部也有保留导航栏，以及顶部的状态栏位。不过目前还处于测试版阶段，因此预计正式版推出后，应该会有更丰富的功能。
7. 折叠手机与 5G 的支持性
    - 今年Samsung、华为等都推出自家的折叠手机，即便Google 还没有，也要赶在其他品牌手机正式推出前，加入折叠手机的支持性，Google 表示不论是多工运作，还是让屏幕呈现的内容可在使用者折叠或打开手机时做出调整，借此因应不同尺寸的屏幕。此外，Android Q 也是第一个支持5G 的作业系统，为开发人员提供必要工具，确保他们开发的应用程序可拥有更快速强大的连线能力，以及提升使用者在游戏与扩增实境中的体验。
8. 更聪明的系统助手
    - 使用Chrome 上网时，如果侦测到你的WIFI 没有连结，则会自动跳出一个设定选单，提供WIFI、行动数据与飞航模式三个选项，让你能快速调整设定，这侦测功能似乎也支持其他应用程序，意旨在用户无需离开应用程序，就能完成设定。
9. 通过二维码分享WIFI
    - Android Q中，使用者将能通过二维码来分享WIFI 的相关信息。
10. 专注模式
    - 在专注模式中，使用者将能选取要保持执行、停用的应用程序，像是信息、相机、备忘录之类，简单来说就是让你能更专注于当下的工作，不被其他应用程序打扰，直到退出专注模式为止。

---
### Android系统结构

---
### Android安装过程

#### 一、复制

安卓的程序目录是/data/app/，所以安装的第一步就是把apk文件复制到这个目录下。这里有四个问题：
- 安卓机有内部存储和SD卡两部分，很多安卓机的内存并不大，需要把apk安装到SD卡上节省内存空间，所以程序目录/data/app/实际上也是在内部存储和SD卡上各一个。
- 系统自带的App是安装在/system/app/目录下的，这个目录只有root权限才能访问，所以系统App在root之前是无法删除和修改的，也就是说，系统App升级时，实际上是在/data/app/里重新安装了一个App，这个路径会重新注册到系统那里，系统再打开App时，就会指向新App的地址。当然，这个新的App是可以卸载的，不过新的App卸载后，系统会把 /system/app/里那个旧的App提供给你，所以是卸掉新的，还你旧的。
- 还是系统App，在root后，我们可以操作/system/app/目录，但是系统安装Apk仍然会装到/data/app/里，所以如果想修改/system/app/目录里的app，必须自己手动push新的apk文件进去，这个新的apk文件不会自动被安装，需要重启设备，系统在重启时检查到apk被更新，才会去安装apk。
- 系统目录有个/system/priv-app/目录，这里面放的是权限更高的系统核心应用，如开机launcher、系统UI、系统设置等，这个目录我们最好不要动，保持系统干净简洁。

#### 二、安装

安卓系统开机启动时，会启动一个超级管理服务SystemServer，这个SystemServer会启动所有的系统核心服务，其中就包括PackageManagerService，简称PMS，具体的apk安装过程，就是由这个PMS操作的。
PMS会监控/data/app/这个目录，在上一步中，系统安装程序向这个目录复制了一个apk，PMS自己就会定期扫描这个目录，找到后缀为apk的文件，如果这个apk没有被安装过，它就会自动开始安装，安装时会做这么几件事：

1. 创建应用目录，路径为/data/data/your package(你的应用包名)，App中使用的数据库、so库、xml文件等，都会放在这个目录下。
2. 提取dex文件，dex是App的可执行文件，系统解压apk就能得到dex文件，然后把dex文件放到/data/dalvik-cache，这样可以提前缓存dex到内存中，能加快启动速度。系统还会把dex优化为odex，进一步加快启动速度。
3. 判断是否可以安装apk，如检查apk签名等。
4. 为应用分配并保存一个UID，UID是来自Linux的用户账户体系，不过在Android这种单用户系统里，UID被用来与App对应，这也是安全机制的一部分，每个App都有自己对应的UID，这种对应关系是持久化保存的，App更新或卸载重装后，系统还会给它分配原来那个UID。用adb pull /data/system/packages.list可以查看所有App的UID。GID（用户组）一般等于UID。
5. 利用AndroidManifest文件，注册Apk的各项信息，包括但不限于：
    - 根据installLocation属性（internalOnly、auto、preferExternal），选择安装在内部存储器还是SD卡上。
    - 根据sharedUserId属性，为App分配UID，如果两个App使用同一个UID，打包时又使用了相同的签名，它们就被视为同一个用户，可以共享数据，甚至运行在同一个进程上。
    - 向/data/system/packages.xml文件中，记录App的包名、权限、版本号、安装路径等；同时在/data/system/packages.list中，更新所有已安装的app列表。
    - 注册App中的的四大组件（Activity、Service、Broadcast Receiver和Content Provider），包括组件的intent-filter和permission等。
    - 在桌面上添加App的快捷方式，如果AndroidManifest文件中有多个Activity被标注为`<action android:name="android.intent.action.MAIN" />`和`<category android:name="android.intent.category.LAUNCHER" />`，系统就会向桌面添加多个App快捷方式，所以有时候在安装一个App后，用户可能会感觉安装了多个App。

#### 三、通知

apk安装完成后，PMS会发一个ACTION_PACKAGE_ADDED广播，如果是卸载，会发ACTION_PACKAGE_REMOVED广播。

整个安装过程大概是这样的：

![](/img/in-post/post-Android/Reorganization/1.png)

---
### Android启动过程
启动一个App，首先需要触发启动过程，然后分配系统资源，最后才启动要打开的App组件。


#### 一、触发启动过程
- 在安卓系统开机启动时，启动的超级管理服务SystemServer会启动所有的系统核心服务，其中就包括ActivityManagerService，简称AMS，启动App具体都是AMS来负责的。
- 不过，一般Java程序都有个main函数入口，启动Java程序其实就是执行main函数去了。但是，安卓App不是这样设计的，App并没有统一的程序入口，一个App其实更像是一群组件的集合，启动App其实就是启动了某个组件，即便是从桌面点击应用图标打开某个App，也是系统桌面Home根据安装时注册的组件信息，找到这个图标对应的Activity信息，再由AMS去启动Activity组件。

![](/img/in-post/post-Android/Reorganization/2.png)

#### 二、分配系统资源
在安卓系统里，除非人为设置为多进程`（Activity的android:process属性）`，否则默认每个App都有`1个独立的进程和虚拟机`，所以在系统启动时，系统会建立一个Linux进程(Process)，在这个进程里放一个虚拟机VM，在这个VM里，运行你的App。
在系统层面，它其实要做这么几件事：
1. 分配UID，App要有UID才能有自己的系统资源，UID是在安装App时由系统分配的，一般每个App都有自己的UID，App的资源不能共享，因为它们不属于同一个用户。
2. 分配进程Process，系统会给App一个进程，每个App的都有自己的进程，进程的PID是系统即时生成的，用完销毁。
如果要让两个App共用进程，除了需要设置同一个进程（android:process），还需要分配同一个UID（android:sharedUserId）来共享系统资源，使用同一个应用签名（同一个签名证书才可以视为同一个程序）。
有时候，如果某些业务特别消耗内存或特别耗时，还可以把1个App分成多个进程，让某些组件在独立的进程中工作，销毁该组建时，把整个进程一起用system.exit来销毁掉。
3. 提供虚拟机VM，安卓App是java程序，需要在java虚拟机上运行，这个虚拟机需要由系统在分配进程时，和进程一起提供。
4. 除非做了跨进程跨用户的配置，否则App之间是隔离的，不能直接互相访问，也不能直接共享资源。
5. AMS管理启动过程，启动App的工作都是AMS统一负责的，AMS里保存了App对应的系统进程ID（PID），在启动App时，AMS会去找App对应的PID，如果找不到PID，说明需要创建进程，就会要求系统为App提供进程和VM。
6. 提供进程和VM，创建VM是非常耗时的，为了加快App启动速度，安卓系统采用了复制的方式：系统开机启动时会启动一个Zygote进程，这个进程会初始化系统的第一个VM, 并预加载framework等app通用资源，当安卓要启动某个App时，Zygote通过fork复制Zygote的VM，就可以快速创建一个带VM的进程，为App运行提供载体。系统开机启动时的第一个进程一般是桌面Home进程。

提供系统资源的过程大概如下图：

![](/img/in-post/post-Android/Reorganization/3.png)


#### 三、启动要打开的App组件
App本身没有main函数入口，但是系统在启动进程时，会创建一个主线程ActivityThread对象（Process.start("android.app.ActivityThread",...)），这个ActivityThread是一个final类，虽然不是线程，但是管理着主线程，它是有main函数入口的（Java终于找到组织了），ActivityThread有这么几个作用：

1. 管理App的主线程，也就是UI线程，启动主线程的Looper，只有主线程可以调用View的onDraw函数来刷新界面(为了线程安全)。
至于视频类控件，都不是View而是SurfaceView，所以可以用子线程刷新，而且SurfaceView是直接绘制到屏幕上的，和View是分开管理的。
另外，BroadCast消息也是主线程处理的，主线程创建BroadCastReceiver对象，并调用其onReceive()函数，处理完就销毁，所以它的生命周期很短（10秒，超时就ANR）。
2. 负责管理调度进程资源、Application和App四大组件中的三个（Activity，Service，ContentProvider），列表中的组件用Token区分，至于BroadCastReceiver，因为是随用随造，用完销毁，所以不需要保存和管理。
3. 构建Context和Application，这个任务包括检查和加载LoadedApk对象、设置分辨率密度、是否高耗内存、获取package和component等启动信息、获取ClassLoader把类加载到内存等，最后，先创建一个context对象contextimpl.createAppContext(this,getSystemContext().mPackageInfo;)，再用context去创建Application对象context.mPackageInfo.makeApplication(true,null)。
4. 对接AMS，AMS自己有专门的系统进程，ActivityThread把一个ApplicationThread（一个Bindler对象）作为自己的Proxy交给AMS，以便由AMS来调度管理ActivityThread中的Activity。
5. 处理消息，ActivityThread是通过消息机制来启动App组件的，ActivityThread有Message队列、Handler和Looper，在AMS启动Activity时，AMS会向ActivityThread发送LAUNCH_ACTIVITY消息启动Activity，ActivityThread收到这个消息后启动Activity，然后，就进入我们熟悉的组件onCreate生命周期了。
6. 重要系统服务如SystemServer也是App，也有ActivityThread，也可能出现ANR之类的异常，为了避免系统“跑飞”，这些应用都有Watchdog看护，出现问题会重启设备。

启动过程大概是这样的：

![](/img/in-post/post-Android/Reorganization/4.png)



#### AMS是通过`IPC`向ActivityThread传递消息的。

另外，在创建组件时，组件之间有这样几个区别：

1. Application和四大组件的启动时机：
    - Application是主线程启动时创建的，这是应用程序运行的第一个类、
    - ContentProvider是主线程启动时创建的（并发布到AMS）、
    - BroadCastReceiver是主线程收到广播时创建的（前台10秒/后台60秒ANR）、
    - Activity是AMS发消息让主线程创建的（5秒ANR）、
    - Service是AMS通过ApplicationThread接口，让主线程创建的（并运行在主线程上，前台20秒/后台200秒ANR）。
2. 关于Context：
    - App里依靠context来提供资源和上下文，所以Application、Activity和Service有context（它们都extends Context）、
    - BroadCastReceiver没有context，主线程在调用onReceive时会把Application的context作为参数传进去、
    - ContentProvider也没有context、
    - 虽然组件有各自的context，但它们指向同一块资源，因为实现ContextImpl时，获取资源的ResourcesManager采用单例模式，所以同一个App的不同context都指向同一个Resource对象
    - Activity的context多了主题Theme，而Application的context生命周期最长。

    这些context之间的关系如下：

    ![](/img/in-post/post-Android/Reorganization/5.png)

3. 关于对Application的共享：
    - App中的Application是一个单例，整个App是共享同一个Application对象的、
    - Activity和Service都有getApplication函数、这个Application是在创建组件时赋给组件的，比如Activity就是ActivityThread在performLaunchActivity时，把Application实体赋给Application的。
    - 组件有getApplication和getApplicationContext两个函数，这两个函数一个是组件本身的，一个contextwrapper要求实现的，很多情况下他们返回的是一个对象，但是官方并不建议把两者混淆。

