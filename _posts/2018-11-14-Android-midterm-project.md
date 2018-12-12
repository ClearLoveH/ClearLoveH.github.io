---
layot:     post
title:      "王者荣耀英雄大全"
subtitle:   "Cooperate with wilson and lim"
date:       2018-11-14 23:10
author:     "Heng"
header-img: "img/弗雷尔卓德2.jpg"
catalog: true
tags:
    - Android
---

>Invictus Gaming——We are the champion！.

---


## 一、期中项目

### 王者荣耀英雄大全

---

#### 项目预览：

![](/img/in-post/post-Android/midterm_pro/images/preview.gif)
![](/img/in-post/post-Android/midterm_pro/images/preview2.gif)
![](/img/in-post/post-Android/midterm_pro/images/preview3.gif)
![](/img/in-post/post-Android/midterm_pro/images/preview4.gif)
![](/img/in-post/post-Android/midterm_pro/images/preview5.gif)
![](/img/in-post/post-Android/midterm_pro/images/preview6.gif)


- [项目展示视频](http://v.youku.com/v_show/id_XMzk2MTcxOTM2NA==.html?spm=a2h3j.8428770.3416059.1)

---

## 二、本人负责部分

* 主页中“社区”页面的完成
* 主页中“我”页面的完成
* 应用登录页面的完成
* 产品设计
* 应用添加入场动画、修改应用图标

---

## 三、课堂实验结果

### (1)实验截图

#### 登录界面

- 登录：
    ![](/img/in-post/post-Android/midterm_pro/images/1.png)
- 注册
    ![](/img/in-post/post-Android/midterm_pro/images/7.png)
- 如果输入已注册用户，自动显示该用户头像
    ![](/img/in-post/post-Android/midterm_pro/images/2.png)
- 不能注册已存在用户
    ![](/img/in-post/post-Android/midterm_pro/images/3.png)
- 选择头像
    ![](/img/in-post/post-Android/midterm_pro/images/4.png)
- 注册6用户
    ![](/img/in-post/post-Android/midterm_pro/images/5.png)
- 登录6用户
    ![](/img/in-post/post-Android/midterm_pro/images/6.png)


#### “社区”界面

- 社区主页：
    ![](/img/in-post/post-Android/midterm_pro/images/8.png)
- 点赞
    ![](/img/in-post/post-Android/midterm_pro/images/9.png)
- 发表新的社区内容：
    - 选择图片
    ![](/img/in-post/post-Android/midterm_pro/images/10.png)
    - 输入想法
    ![](/img/in-post/post-Android/midterm_pro/images/11.png)
    - 点击“发表”发布，社区主页同时更新：
    ![](/img/in-post/post-Android/midterm_pro/images/12.png)
- 对于属于当前登录用户的post，可以进行编辑与删除操作：
    - 编辑post：
        ![](/img/in-post/post-Android/midterm_pro/images/13.png)
    - 编辑后效果：
        ![](/img/in-post/post-Android/midterm_pro/images/14.png)
    - 删除post：
        ![](/img/in-post/post-Android/midterm_pro/images/15.png)
        ![](/img/in-post/post-Android/midterm_pro/images/16.png)
    - 删除后社区显示：
        ![](/img/in-post/post-Android/midterm_pro/images/17.png)

#### “我”的界面
- “我”——用户主页
 ![](/img/in-post/post-Android/midterm_pro/images/18.png)
- 在英雄主页为英雄评分，评分大于3的添加到个人收藏夹中
    ![](/img/in-post/post-Android/midterm_pro/images/19.png)
    ![](/img/in-post/post-Android/midterm_pro/images/20.png)
- 在用户主页收藏列表中的英雄，我们点击进去可以进行重新评分，若个人重新评分低于3，则会在该用户的收藏列表中自动移除该英雄，同时你的评分会影响着该英雄的总评分。
    ![](/img/in-post/post-Android/midterm_pro/images/22.png)
    ![](/img/in-post/post-Android/midterm_pro/images/23.png)    
- 点击用户icon提供换头像的功能
    ![](/img/in-post/post-Android/midterm_pro/images/24.png)
    ![](/img/in-post/post-Android/midterm_pro/images/25.png)
- 登出：
    ![](/img/in-post/post-Android/midterm_pro/images/21.png)

### (2)实验步骤以及关键代码

>注：这里我只提供了关键操作或者难点重点操作的代码

#### 登录界面的代码实现 —— `loginActivity.java`
- 登录页面的操作有：
    - 登录与注册页面的互相切换
    - 头像选择并呈现
    - `若输入了已存在的用户名，自动显示该用户的头像：`
        ```java
        public void loginHit(){
            if(MainActivity.database.getImage(usernameEdit.getText().toString())!=null){
                Glide.with(getApplicationContext()).load(MainActivity.database.getImage(usernameEdit.getText().toString())).into(icon);

            }else {
                Glide.with(getApplicationContext()).load(R.drawable.default_icon).into(icon);
            }
        }
        ```
    - 登录注册的相关hints（用户重复、密码错误）

#### “我”——用户个人主页代码实现 —— `user_page.java`
- 用户个人主页的操作有：
    - 点击个人主页的icon提供换头像功能
    - 为英雄评分自动添加评分大于3的英雄在个人的收藏列中（这部分实现调用的是钟同学实现的数据库的接口）
    - 登出
        ```java
        floatingActionButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                AlertDialog dialog = new AlertDialog.Builder(getContext())
                        .setTitle("确定登出？")
                        .setPositiveButton("退出登录",new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialog, int which) {
                                clear();
                                MainActivity.postPage.clear();
                                MainActivity.loginstate = false;
                                MainActivity.viewPager.setCurrentItem(0);
                                MainActivity.currentUser = null;
                            }
                        })
                        .setNegativeButton("取消",null)
                        .create();
                dialog.show();
                dialog.getButton(AlertDialog.BUTTON_POSITIVE).setTextColor(Color.RED);
                dialog.getButton(DialogInterface.BUTTON_NEGATIVE).setTextColor(Color.BLUE);
            }
        });
        ```
    - 这里提供这个使用频率比较高的一个用于更新收藏列的函数：
        ```java
        public void updateCollect() {
            // update List

            listView.setVisibility(View.VISIBLE);
            collectionHeroes.clear();
            collectionHeroes.addAll(MainActivity.database.getCollectHero(MainActivity.currentUser.name));
            collectionListAdapter.notifyDataSetChanged();

            if(collectionHeroes.size()==0){
                noCollectionHint.setVisibility(View.VISIBLE);
            }
            else {
                noCollectionHint.setVisibility(View.GONE);
            }
        }
        ```
#### 社区页面代码实现 —— `post_page.java`
- 社区页面的操作有：
    - 发布属于当前用户的post
    - 对于其他用户的post，当前用户可以点赞
        ```java
        postListAdapter.setOnItemLikeClickListener(new postListAdapter.onItemLikeListener() {
            @Override
            public void onLikeClick(int i) {
                MainActivity.database.toggleFavouritePost(posts.get(i).id, MainActivity.currentUser.name);
                refresh();
            }
        });
        ```
    - 对于当前用户自己发布的post，可以点赞、编辑以及删除
        - 编辑post：
            ```java
            postListAdapter.setOnItemEditPostClickListener(new postListAdapter.onItemEditPostListener() {
                @Override
                public void onEditPostClick(int i) {
                    ImageSelectorUtils.openPhoto(getActivity(), MainActivity.EDIT_POST, false, 9, listToArrayList(posts.get(i).imgList));
                    editPost = posts.get(i);
                }
            });
            ```
        - 删除post：
            ```java
            postListAdapter.setOnItemDeleteClickListener(new postListAdapter.onItemDeleteListener() {
                @Override
                public void onDeleteClick(int i) {
                    final int postId = posts.get(i).id;
                    AlertDialog dialog = new AlertDialog.Builder(getContext())
                            .setTitle("确定删除？")
                            .setPositiveButton("删除", new DialogInterface.OnClickListener() {
                                @Override
                                public void onClick(DialogInterface dialog, int which) {
                                    MainActivity.database.deletePost(postId);
                                    refresh();
                                }
                            })
                            .setNegativeButton("取消", null)
                            .create();
                    dialog.show();
                    dialog.getButton(AlertDialog.BUTTON_POSITIVE).setTextColor(Color.RED);
                    dialog.getButton(DialogInterface.BUTTON_NEGATIVE).setTextColor(Color.BLUE);
                }
            });
            ```
    - 当对postList中的post进行删改或点赞是，使用refresh()函数用于刷新该页面
        ```java
        //刷新当前activity
        //先把data中清空，再重新加载
        public void refresh() {
            posts.clear();
            posts.addAll(MainActivity.database.getAllPost());
            postListAdapter.notifyDataSetChanged();
            if (posts.size() > 0) {
                noPostHint.setVisibility(View.GONE);
            } else noPostHint.setVisibility(View.VISIBLE);
        }
        ```

#### 应用入场动画的实现 —— splashActivity.java
- 实现三个图片依次渐入渐出的效果，在重载的`public void onAnimationEnd(Animation animation)`函数中（该函数在当前动画结束时触发），套用下一个动画的完整过程，三轮套用即实现三个动画依次呈现的效果。
    ```java
    public class splashActivity extends Activity
    {
        private ImageView welcomeImg = null;

        @Override
        protected void onCreate(Bundle savedInstanceState)
        {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.splash);
            welcomeImg = this.findViewById(R.id.cover);
            AlphaAnimation anima = new AlphaAnimation(0,1);

            MainActivity.database = new Database(this, Database.DB_NAME);
            MainActivity.sharedPreferences = getSharedPreferences("HeroStatus", MODE_PRIVATE);

            //非阻塞加载英雄list
            Thread getHero = new Thread(){
                public void run(){
                    try {
                        GetData.getHerosFromNet_Online();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            };

            getHero.start();

            anima.setDuration(2000);// 设置动画显示时间
            anima.setAnimationListener(new AnimationImpl());
            //第一个入场动画
            welcomeImg.startAnimation(anima);
        }
    ```
    ```java
    private class AnimationImpl implements AnimationListener
        {

            @Override
            public void onAnimationStart(Animation animation)
            {
                welcomeImg.setBackgroundResource(R.drawable.cover1);
            }

            @Override
            public void onAnimationEnd(Animation animation)
            {
                AlphaAnimation anima2 = new AlphaAnimation(1, 0);
                anima2.setDuration(1000);
                welcomeImg.startAnimation(anima2);
                anima2.setFillAfter(true);
                anima2.setAnimationListener(new Animation.AnimationListener() {
                    @Override
                    public void onAnimationStart(Animation animation) {

                    }

                    //第二个入场动画
                    @Override
                    public void onAnimationEnd(Animation animation) {
                    ······
                    ······    
    ```

### (3)实验遇到的困难以及解决思路
- 朋友圈效果的实现
    - 解决思路：listView嵌套两个RecycleView实现，当然我们也需要对应的三个adapter。
    - 每个post，我们需要实现选择的图片（不超过9张）的呈现以及所有点赞用户头像的呈现，加上post本身需要listview来呈现，所以我们于此通过listView嵌套两个RecycleView来最终实现朋友圈的效果。
        ```xml
        <ListView
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:padding="20dp"
            android:visibility="gone"
            android:id="@+id/post_list"
            android:divider="#d1d0d0"
            android:dividerHeight="1dp"
            app:layout_constraintTop_toBottomOf="@id/toolbar"
            android:layout_marginBottom="50dp"
            android:layout_marginTop="50dp"/>
        ```
        ```xml
        <android.support.v7.widget.RecyclerView
            android:id="@+id/post_images"
            android:layout_width="250dp"
            android:layout_height="wrap_content"
            android:layout_marginTop="10dp"
            android:layout_gravity="center"
            android:background="#00000000"
            android:scaleType="centerCrop"
            app:layout_constraintLeft_toLeftOf="@id/post_AllInfo"
            app:layout_constraintTop_toBottomOf="@id/post_AllInfo"/>
        
        <android.support.v7.widget.RecyclerView
            android:id="@+id/likedBy_images"
            android:layout_width="250dp"
            android:layout_height="30dp"
            android:layout_marginTop="2dp"
            android:layout_marginLeft="3dp"
            android:layout_gravity="center"
            android:background="#00000000"
            android:scaleType="centerCrop" />
        ```
- 登录状态的对主应用的影响——未登录、登录和登出三种状态下社区页和“我”用户页都要正确的符合逻辑的呈现
    - 为了保证社区页以及“我”页面能根据登录状态正确的呈现，我们通过维护一个静态变量`loginState`以及`currentUser`来随时获取或改变当前登录用户的状态。
    - 在MainActivity中，viewpage的切换函数（如下）中，我们不直接切换页面，而是通过`loginState`以及`currentUser`来判断是否正确呈现post_list和collection_list中内容。
        ```java
        public void onPageSelected(int i) {
            switch (i) {
                    case 0:
                        bottomNavigationView.getMenu().getItem(0).setChecked(true);
                        break;
                    case 1:
                        if(!loginstate){
                            Toast.makeText(getApplicationContext(), "Please login firstly", Toast.LENGTH_SHORT).show();
                            Intent intent=new Intent(getApplicationContext(),loginActivity.class);
                            startActivityForResult(intent,LOGIN);
                        } else {
                            userPage.updateCollect();
                            bottomNavigationView.getMenu().getItem(1).setChecked(true);
                        }
                        break;
                    case 2:
                        if(!loginstate){
                            Toast.makeText(getApplicationContext(), "Please login firstly", Toast.LENGTH_SHORT).show();
                            Intent intent=new Intent(getApplicationContext(),loginActivity.class);
                            startActivityForResult(intent,LOGIN);
                        } else {
                            userPage.updateCollect();
                            bottomNavigationView.getMenu().getItem(2).setChecked(true);
                        }
                        break;
                    default:
                        break;
                }
            }
        ```

- viewPage在切换时fragment数据刷新的问题
    - 因为我们这个项目是基于`BottomNavigationView + ViewPager + Fragment`的框架，而`setUserVisibleHint`函数就是viewPage在切换时调用的函数，一开始在切换viewPage时，每个页面的初始化时在`onCreate`调用时就完成了，但是我们在登录与不登录时，社区页面和“我”页面所呈现的内容是不一样的，所以我们在登陆后通过`setUserVisibleHint`函数来实现fragment的数据刷新问题，相关数据刷新函数都在该函数实现即可。

## 四、实验思考及感想
- 本次项目经验真的是非常丰富且实用，虽然过程真的很累，代码量也比较多，但是最终我们小组齐心协力的项目呈现的效果还是非常好的。
- 我们小组项目的完成是先搭好了架构，然后我们再一起定好需求，根据需求来各自分工。框架定好了再来进行各自的编程，并且基于github合并我们大家的进度，这两周下来，我们项目的完成过程中可以说是思路一直非常的清晰了，即使代码量较大，也各自相对独立，有条不紊。就像我需要使用到数据库的某些增删操作时，我无需自己写数据库的查询操作，而是负责数据库的同学写好接口，我们来调用即可。所以我们这次小组的配合上来说是非常成功的。
- 本次实验我们花了都很多的心思，为了让我们自己定的需求实现的更好，我们都各自搜索了一些比较好的封装好的包来满足我们的想法，像图片选择这种功能，放在很多的app中都是有需要的一个功能，至少我以后遇到类似问题，也可以重新利用到本次项目所获得的经验和想法来解决。
- 社区的实现一开始我们还是模糊的需求，后来确定要实现，我决定自己弄一个仿朋友圈的实现，最后虽然相对来说要比微信的差一些，但效果呈现出来还是很不错的，感到比较满意，这也是我第一次使用listview嵌套RecycleView来实现朋友圈的这样一个功能，让我对adapter已经listview、recycleView的理解更加的深刻了。
- 我们的项目使用的是framework框架，把各自的页面分开实现，有交集的部分都放在MainActivity中（主要是OnActivityResult的使用），项目结构非常明晰，没有错乱，代码的可读性也是非常高的。
- 总而言之，这是一次愉快的项目经验，确实是在安卓方面让我学到了很多新的东西，我们小组三人都有不同程度的提升，真的收获颇丰。

