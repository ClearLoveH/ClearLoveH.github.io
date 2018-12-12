---
layot:     post
title:      "简单音乐播放器"
date:       2018-11-26 10:50
author:     "Heng"
header-img: "img/弗雷尔卓德2.jpg"
catalog: true
tags:
    - Android
---

>Invictus Gaming——We are the champion！.

---

[项目仓库](https://gitee.com/ClearLoveH/PersonalProject4)

---

## 一、实验题目 

### 简单音乐播放器

---

## 二、实现内容

- 实现一个简单的播放器，要求功能有：

    - 播放、暂停、停止、退出功能，按停止键会重置封面转角，进度条和播放按钮；按退出键将停止播放并退出程序
    - 后台播放功能，按手机的返回键和home键都不会停止播放，而是转入后台进行播放
    - 进度条显示播放进度、拖动进度条改变进度功能
    - 播放时图片旋转，显示当前播放时间功能，圆形图片的实现使用的是一个开源控件CircleImageView

    - 附加内容（加分项，加分项每项占10分）
        - 选歌：
        用户可以点击选歌按钮自己选择歌曲进行播放，要求换歌后不仅能正常实现上述的全部功能，还要求选歌成功后不自动播放，重置播放按钮，重置进度条，重置歌曲封面转动角度，最重要的一点：需要解析mp3文件，并更新封面图片。

- 实验目的
    - 学会使用MediaPlayer
    - 学会简单的多线程编程，使用Handler更新UI
    - 学会使用Service进行后台工作
    - 学会使用Service与Activity进行通信
    - `学习rxJava，使用rxJava更新UI`

---

## 三、课堂实验结果
### (1)实验截图

>注意本次实验使用的是虚拟机，而不是真机测试，所以在mp3存放路径的选择上会有所不同。`初始文件路径为：storage/emulated/0/Music/山高水长.mp3`

- 正常播放初始路径下文件

    ![](/img/in-post/post-Android/lab4/截图/1.png)

- 封面会随着音乐一起转动，同时显示当前播放时间

    ![](/img/in-post/post-Android/lab4/截图/2.png)

- 暂停

    ![](/img/in-post/post-Android/lab4/截图/8.png)

- 拖动进度条音乐播放进度也随着改变

    ![](/img/in-post/post-Android/lab4/截图/3.png)

- 停止

    ![](/img/in-post/post-Android/lab4/截图/4.png)

- 点击home键回到主页，音乐在后台继续播放，且重新进入应用时继续播放

    ![](/img/in-post/post-Android/lab4/截图/5.png)


加分项：

- 点击左下按钮自选择音频文件进行播放，歌名及歌手都进行切换

    ![](/img/in-post/post-Android/lab4/截图/9.png)

    - 播放正常

        ![](/img/in-post/post-Android/lab4/截图/10.png)


### (2)实验步骤以及关键代码

第一周我们在Activity中直接获取service中的静态变量mediaplayer并直接对其进行操作，虽然用这种方法实现对应的需求比较简单，但是却违背了我们想把后台service与前端Activity分离的想法，这样是不严谨的。

所以第二周我们就进行播放器的修改与升级，我们把service中mediaplayer不再设为静态变量，Activity中需要使用到mediaplayer的所有需求都改为用transact和ontransact的方法来进行通信，从而达到service的一种封装，这样的后台与前台的沟通才是比较严谨的。

1. 音乐的播放、暂停和停止。
    - 第一周的实现：因为是直接获取service中的mediaPlayer静态变量，我们可以在Activity中直接设置播放暂停及停止。
    - 第二周，我们改用transact和ontransact的方法来进行Activity与service的通信，所以对音乐的播放、暂停和停止不再在Activity中直接改变，而是通过transect传信息给服务器让服务器来进行相应操作。
        ```java
        startOrPause.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Parcel data = Parcel.obtain();
                Parcel reply = Parcel.obtain();
                try {
                    mBinder.transact(PLAY_CODE, data, reply, 0);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
                if(reply.readInt() == 1){
                    if(playState == 0) {
                        objectAnimator.start();
                        playState = 1;
                    }
                    else objectAnimator.resume();
                }
                else objectAnimator.pause();
            }
        });


        stop.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Parcel data = Parcel.obtain();
                Parcel reply = Parcel.obtain();
                try {
                    mBinder.transact(STOP_CODE, data, reply, 0);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
                objectAnimator.end();
                seekBar.setProgress(0);
                currentTime.setText("00:00");
                playState = 0;
            }

        });

        exit.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                android.os.Process.killProcess(android.os.Process.myPid());   //获取PID
                System.exit(0);
            }
        });

        ```

2. 旋转动画：
    ```java
    //设置旋转动画
        objectAnimator = ObjectAnimator.ofFloat(imageView, "rotation", 0f, 360f);//添加旋转动画
        objectAnimator.setDuration(30000);//设置动画时间
        objectAnimator.setRepeatCount(ObjectAnimator.INFINITE);
        objectAnimator.setRepeatMode(ObjectAnimator.RESTART);
    ```
- 第二周此部分的修改就是旋转动画的暂停与开始都是通过transact和ontransact判断service的播放状态，从而随着暂停与播放。
    ```java
    if(reply.readInt() == 1){
        if(playState == 0) {
            objectAnimator.start();
            playState = 1;
        }
        else objectAnimator.resume();
    }
    else objectAnimator.pause();
    ```


3. 解析MP3文件信息并在主界面显示出来：
    - 这部分需求主要通过`MediaMetadataRetriever`来实现，MediaMetadataRetriever可以通过文件路径解析对应媒体文件的基本信息，文件名、文件的创作着以及文件的专辑图都可以解析出来。
        ```java
        //调整播放器显示歌曲信息
        mmr.release();
        mmr=new MediaMetadataRetriever();
        mmr.setDataSource(path);
        title.setText(mmr.extractMetadata(MediaMetadataRetriever.METADATA_KEY_TITLE));
        author.setText(mmr.extractMetadata(MediaMetadataRetriever.METADATA_KEY_ARTIST));
        byte[] image = mmr.getEmbeddedPicture();
        if(image == null){
            return;
        }
        Bitmap bitmap = BitmapFactory.decodeByteArray(image, 0, image.length);
        imageView.setImageBitmap(bitmap);
        ```
    - 对播放歌曲的文件信息的解析，和mediaPlayer的播放状态并没有直接关系，所以没有进行修改。

4. 关键点——`根据歌曲播放状态进行主界面UI的更新（播放进度及进度条的更新）`

    - 第一周，使用的是Handler来进行UI的更新，因为我们可以对service中mp（mediaplayer）进行直接的访问，所以在Handler中我们就直接通过判断mp当前的播放进度来更新进度条即可，播放按钮随播放状态的变化而改变也可以放在此处。Handler这里相当于时刻监听着mp的状态并进行对应的UI更新。
        ```java
            //Handler 更新UI
            //在Runnable中更新SeekBar的状态，并且拖动SeekBar滑动条使歌曲跳动到指定位置
            public android.os.Handler handler = new android.os.Handler();
            public Runnable runnable = new Runnable() {
                @Override
                public void run() {
                    if(mediaPlayer.isPlaying()) {
                        startOrPause.setBackgroundResource(R.drawable.pause);
                    } else {
                        startOrPause.setBackgroundResource(R.drawable.play);
                    }

                    currentTime.setText(timeParse(mediaPlayer.getCurrentPosition()));
                    endTime.setText(timeParse(mediaPlayer.getDuration()));
                    seekBar.setProgress(mediaPlayer.getCurrentPosition());

                    handler.post(runnable);
                }
            };
        ```
    - 第二周，我们改用Rxjava替代handler来进行UI更新，这也是这周比较难的部分，使用Rxjava来进行UI更新的思路主要是，`把耗时操作放在后台，实时的通知主页面UI的更新`。
        - 这里耗时操作指的就是于此每隔500ms（不可太低，频率太快会出错），使用transact和ontransact，对service进行一次通信，让service实时返回后台的mp的当前状态（当前播放进度，是否正在播放），从而在主页进行进度条以及播放按钮的即时更新。
        - 而使用Rxjava的观察者模式实现UI实时更新，主要是三个步骤：
            -  创建被观察者，我们所需要的耗时操作就放在被观察者中（向service实时发送更新信息的请求）
                ```java
                //实时更新UI
                final Observable<Integer> observable = Observable.create(new ObservableOnSubscribe<Integer>() {
                    @Override
                    public void subscribe(ObservableEmitter<Integer> e) throws Exception {
                        while (true){

                            Parcel data = Parcel.obtain();
                            Parcel reply = Parcel.obtain();
                            try {
                                Thread.sleep(500);
                                mBinder.transact(GET_BAR, data, reply, 0);
                                e.onNext(reply.readInt());
                            } catch (InterruptedException exception) {
                                if (!e.isDisposed()) {
                                    e.onError(exception);
                                }
                            }
                        }
                    }
                });
                ```
            - 创建观察者，它决定事件触发的时候将有怎样的行为，即获取到service返回给我们的信息之后，该怎么更新UI。（主要在`onNext()`函数中更新UI）
                ```java
                DisposableObserver<Integer> disposableObserver = new DisposableObserver<Integer>() {

                    @Override
                    public void onNext(Integer o) {
                        Log.i("state", o.toString());
                        if(o == -1){
                            startOrPause.setBackgroundResource(R.drawable.play);

                        }
                        else {
                            startOrPause.setBackgroundResource(R.drawable.pause);
                            seekBar.setProgress(o);
                            currentTime.setText(timeParse(o));
                        }
                    }

                    @Override
                    public void onError(@NonNull Throwable e) {
                    }
                    @Override
                    public void onComplete() {
                    }
                };
                ```
            - 建立订阅关系：把前面创建的观察者与被观察者进行连接。
                ```java
                //建立订阅关系
                observable.subscribeOn(Schedulers.io()).observeOn(AndroidSchedulers.mainThread()).subscribe(disposableObserver);
                mCompositeDisposable.add(disposableObserver);
                ```

5. service中对Activity发来的信息进行对应的操作，第一周不使用transact和ontransact，所以service中关于mp的操作都是在Activity中直接实现，没有做到很好的封装性。这一周改为transact和ontransact的通信方式，service中关于mp的操作都放在了service中（通过`onTransact()`函数获取Acitvity发送过来的操作码及data，进行对应操作或者把相关数据写回reply中）：
    ```java
    @Override
        protected boolean onTransact(int code, @NonNull Parcel data, @Nullable Parcel reply, int flags) throws RemoteException {

            switch (code) {
                //service solve
                case PLAY_CODE:
                    if(mp.isPlaying()){
                        mp.pause();
                        reply.writeInt(-1);
                    }
                    else {
                        reply.writeInt(1);
                        mp.start();
                    }
                    break;
                case STOP_CODE:
                    stop();
                    break;
                case DRAG_SEEK_BAR :
                    mp.seekTo(data.readInt());
                    break;
                case SELECT_NEW_SONG :
                    updateInfo(data.readString());
                    reply.writeInt(mp.getDuration());
                    break;
                case GET_BAR :
                    Log.i("IsPlaying", "" + mp.isPlaying());
                    if(mp.isPlaying())
                        reply.writeInt(mp.getCurrentPosition());
                    else reply.writeInt(-1);

            }
            return super.onTransact(code, data, reply, flags);
        }
    ```

6. 加分项，自助选歌并进行播放，对应歌曲的名字，创作者及专辑图也进行更新。
    - 第一二周的差别主要在选歌后，mp重新设置`setDataSource()`的位置由Activity改到了service中。记住一定要reset()之后才能重新setDataSource()。
        ```java
        //换歌
        public void updateInfo(String path){
            try {
                mp.reset();
                mp.setDataSource(path);
                mp.prepare();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        ```
    - 第二周在Activity换歌只需要把选取文件后的路径发送给service来处理即可。

### (3)实验遇到的困难以及解决思路

第一周：

这次音乐播放器实验遇到的主要问题有一个！！！：

- 选歌问题：
    - 选歌功能的实现是遇到了很刁钻的bug，我的思考思路是，把mediaplayer给release掉，再重新setData即可，但是我发现这种做法是有问题的；后面我决定new一个新的mediaplayer在重新setData，这种新的解决方法，表面上让我解决了这个问题，实际上却导致了另外的error——点击home键再进入应用此时就会秒退，我一直思考了很久为什么会秒退，在想可能是service的问题，后来我取消新建一个mediaplayer，直接使用mediaplayer的reset方法，所有问题都迎刃而解了。
    - 这个问题提醒了我以后万不可想当然去按自己的想法来使用函数，表面上解决了问题实际上却只是把问题转移了，这样最终让我的debug更加的困难。
    - 那现在思考一下，为什么新建了一个mediaplayer，再测试后台播放的功能，却会出现秒退的问题呢？
        - 后台播放的功能是结合了service的使用的，而我们mediaplayer的创建就是在service中完成的，在MainActivity中我们的mediaplayer变量调用的是service中创建的，而自己并没有创建一个新的，


---

第二周：

改用transact和ontransact进行Acitivity与service的通信并使用Rxjava取代Handler来进行UI的更新之后，重新实验过程中，遇到的问题有三个：

- `bindService`异步问题，因为在service中我有设置了换新歌的操作，service在接受到换新歌操作码`SELECT_NEW_SONG`之后，会返回该歌曲的duration，从而更新进度条的长度，以及显示该歌曲的时长，为了让充分复用自己写的代码，减少代码冗余，所以一开始我们初始化歌曲（山高水长）时，我会调用一次换歌的操作，给service发送换歌信息，同时把初始化的歌曲的路径发送过去。
    - 但是我把这个部分操作放在Activity中bindService之后发送，却报错提示我，此时mBinder为null，即并没有与service绑定上，明明不是放在bindService函数之后了吗？——后来才知道，bindService绑定service是一个异步操作，并不是说我们执行这一行绑定的代码就会直接进行service绑定。
    - 所以我直接把第一次初始化的操作放在了 ServiceConnection的`onServiceConnected()`函数中，等到service真正绑定连接上了，才给service发送第一次信息。
        ```java
        private ServiceConnection sc = new ServiceConnection() {
            @Override
            public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
                mBinder = iBinder;
                //初始化，调用一次选取默认歌曲
                Parcel data = Parcel.obtain();
                Parcel reply = Parcel.obtain();
                data.writeString(path);
                try {
                    mBinder.transact(SELECT_NEW_SONG, data, reply, 0);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
                int endtime=reply.readInt();
                seekBar.setMax(endtime);
                endTime.setText(timeParse(endtime));
            }

            @Override
            public void onServiceDisconnected(ComponentName componentName) {
                mBinder = null;
            }
        };
        ```

- 使用Rxjava进行UI更新时，我犯了一个小错误，我把`Thread.sleep(500);`放在了transact与onNext之间，导致onNext函数一直没有正确执行，与service通信，然后进行通信后的操作应该是要连贯的，中间添加了一个耗时操作可能把transact获得的返回的reply中的数据给丢失掉了，从而导致onNext无法正确执行。
    ```java
    while (true){
        Parcel data = Parcel.obtain();
        Parcel reply = Parcel.obtain();
        try {
            Thread.sleep(300);
            mBinder.transact(GET_BAR, data, reply, 0);
            e.onNext(reply.readInt());
        } catch (InterruptedException exception) {
            if (!e.isDisposed()) {
                e.onError(exception);
            }
        }
    }
    ```
- 当我们把耗时操作的时间降低——`Thread.sleep(300)`，即把Activity发送请求service更新UI的信息的频率提高时，系统报错——`Skipped 39 frames!  The application may be doing too much work on its main thread.`，正如报错所告知我们的，发送请求频率太高可能会导致Activity对返回的reply处理还没有做完就进入了下一个新的请求循环。`因为获取返回值我们要实时更新播放按钮的图片，（是播放状态还是暂停状态），对图片的加载是需要时间的，消耗的时间根据图片的大小而定`，而我们对进度条进行更新时，在显示歌曲当前的播放进度，本来就是以秒为单位的，若我们把耗时操作设为100ms一次，其实有很多发送的请求是多余的，所以提高这个请求的频率这里也只能做到减少播放按钮切换图片时的响应时间，所以我们需要适当提高Thraead的sleep时间，让Activit有足够的时间进行图片的切换即可。
    ```java
    @Override
    public void onNext(Integer o) {
        Log.i("state", o.toString());
        if(o == -1){
            startOrPause.setBackgroundResource(R.drawable.play);
        }
        else {
            startOrPause.setBackgroundResource(R.drawable.pause);
            seekBar.setProgress(o);
            currentTime.setText(timeParse(o));
        }
    }

    ```

- 最后一个问题，我在使用service给我Activity的返回值reply时，我使用其中的数据两次，所以一开始就read了两次，但是却发现，第二次read时失效了的，查阅[资料](https://blog.csdn.net/qinjuning/article/details/6785517)发现，
    - 我们在使用Parcel的writeXXX()和readXXX()时，所导致的偏移量是共用的，例如，我们在writeInt(23)后，此时的datapostion=4，如果我们想读取5，简单的通过readInt()是不行的，只能得到0。这时我们只能通过setDataPosition(0)设置为起始偏移量，从起始位置读取四个字节，即23。因此，在(多次)读取某个值时，可能需要使用setDataPostion(int postion)使偏移量装换到我们的值处。
    - 所以在使用service给我们的返回值reply时或者service使用我们传过去的data，不能readXXX()两次，因为read一次使用完就会偏移，所以我们可以新建一个临时变量来存储返回值，或者把一次readXXX()或writeXXX()之后偏移量设为0即可。

## 四、实验思考及感想

实验中的思考与感悟：

- 首先，TA这次需求的改版升级，也是在提醒我们做项目一定要严谨，我们经常在做项目时，也竟然容易为了方便，设置一些静态的全局变量来直接访问，我自己也经常弄一些public的变量，为了访问方便，但是这些却破坏了类的封装性，其实对整个项目来说是很容易把项目的结构给弄得非常混乱，这样是很不好的习惯，特别是出去做项目了，所以这次多一周做了一个新的版本，其实比第一周的任务量还要多一些，但是收获却是很大的，也感受到一个好的编程习惯的重要性。
- 其次，便是这次任务之后，在前后台的通信的学习上，我有了很大的收获，通过`transact和ontransact`取代我们平时喜欢的静态全局变量，让我们的代码结构更好，同时也让前台与后台的分工更加明晰，不会出现代码冗余。而在此基础上，使用Rxjava来实现后台完成耗时操作，手动实现前台对后台的监听机制，通过这种架构来实现另一种以前没有接触过的线程操作，也是收获挺大的，以后在遇到需要新开线程，不能阻塞主线程的操作时，我也可以有更多的选择。
- 学习了解到Rxjava之后，对比普通的`创建一个新的子线程`或者`使用AsyncTask异步处理任务`，虽然本次实验用Rxjava来实时更新UI好像没有什么差别或者优点，但是这种订阅者和观察者的模式，在面对`多个观察者需要同时监听一个后台（被观察者）`的情况时，优点是非常大的：
    - 我们可以使用 CompositeDisposable 对下游进行管理：如果需要添加观察者，通过 `CompositeDisposable.add(disposableObserver)`即可添加；
    - 如果Activity要被销毁时，我们的后台任务没有执行完，那么就会导致Activity不能正常回收，而对于每一个Observer，都会有一个Disposable对象用于管理，而RxJava提供了一个CompositeDisposable类用于管理这些Disposable，我们只需要将其将入到该集合当中，在Activity的onDestroy方法中，调用它的clear方法，就能`避免内存泄漏的发生`。
    - 所以我们在Activity销毁时，同时也要记得要清空CompositeDisposable，避免内存泄漏的发生。
        ```java
         //在onDestroy()中解除绑定
        @Override
        public void onDestroy() {
            super.onDestroy();
            unbindService(sc);
            mCompositeDisposable.clear();
            mmr.release();
        }
        ```
    - Rxjava也提供了给我们可以使用的线程：
        ```
        observable.subscribeOn(Schedulers.io()).observeOn(AndroidSchedulers.mainThread()).subscribe(disposableObserver);
        ```
        绑定观察者与被观察者的同时，也可以指定观察者与被观察者都在什么线程上运行，使用起来非常明了。

---

