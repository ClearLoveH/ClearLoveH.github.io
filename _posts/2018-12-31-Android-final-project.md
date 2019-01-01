---
layout:     post
title:      "Yue Ba 项目设计"
subtitle:   "Android期末项目"
date:       2018-12-31 23:59
author:     "Heng"
header-img: "img/恕瑞玛.jpg"
catalog: true
tags:
    - Android
---

>新年快乐

---

坑：
- `listview的item回收问题`，这也是我去面试被提问到的问题，现在做到这部分豁然开朗。
    - listview为了保证item的条目不至于太多导致OOM（OutOfMemory），会自动回收，而这个自动回收机制`让我在滑动list的时候，已经加载过的item如果被滑上去屏幕之外，重新滑下来的时候又会重新加载，这个加载对于大图就显得有些卡顿`；而对于recyclerView，也有同样的自动回收机制，但不同于listview的是，recyclerView允许我们自己设置缓存的item数：
        recyclerView.setItemViewCacheSize(20);
    让我们保证少数的item不至于每次滑动之后都要销毁，但是这样的机制也是不可避免的，为了保证内存不会OOM，所以我需要考虑的是另外一个问题，就是加载图片减小其大小即可提升加载速度。

- 提升加载图片的速度，就需要我去思考怎么缩小加载图片的大小。解决思路，使用Glide加载图片时加上`fitCenter()`或者修改imageView的scaletype为fitCenter或者centerCrop即可。（缩放显示图，分辨率再大也按imageview的size来加载）
    - 对比：
        - 一开始不作处理直接加载大图，加载时间很长，用户体验不是很好：
            ![](/img/in-post/post-Android/final_project/2_2.png)
        - 改用fitCenter加载大图
            ![](/img/in-post/post-Android/final_project/2_1.png)
        - 明显看到分析器的Energy列的差距，使用fitCenter让加载所消耗的资源减少了许多。

    - [解决方法参考](https://blog.csdn.net/myth13141314/article/details/78248267)

- 一个非常坑的东西，在fragment中不能直接startActivityForResult，因为这个的requestcode是随机生成的，切记应该需要先获取activity，再使用该指令，如下：这样才能在MainActivity中获取到正确的requestCode
        getActivity().startActivityForResult(intent,FINISH_POST);

- 又一个非常非常坑的东西，应用刚启动时，便会调用所有fragment的setUserVisibleHint()函数一次，关键是，fragment创建的时机并不是在activity的oncreate方法之后。
    - 针对性的，我出现这个问题是因为我想在fragment创建的时候便初始话一次postList，如果我们在setUserVisibleHint函数里一直调用刷新postList的话，会让用户体验不是很好，做一些多余的刷新操作，那么我这里就很巧妙地利用了上面提到的fragment的生命周期超前于activity的特点：判断`getActivity()!=null`，同时设一个boolean值，这样就完成了只有一次的初始化操作。
        ```java
        @Override
        public void setUserVisibleHint(boolean isVisibleToUser) {
            super.setUserVisibleHint(isVisibleToUser);
            if (init && getActivity()!=null){
                init = false;
                refresh();
            }
        }
        ```
    - 关于`setUserVisibleHint`函数的非常好的[参考博客](https://blog.csdn.net/czhpxl007/article/details/51277319)
    - 关于fragment的生命周期，非常重要的几个结论：
        - fragment生命周期图：

            ![](/img/in-post/post-Android/final_project/4_1.png)
            ![](/img/in-post/post-Android/final_project/4_1.png)

        问：
        1. fragment是如何知道自己时候用户可见？
        2. setUserVisibleHint() 在上图所示fragment的生命周期的什么位置?

        答：
        1. viewpager监听切换tab事件，tab切换一次，执行一次setUserVisibleHint()方法
        2. setUserVisibleHint() 在 上图所示fragment所有生命周期之前，无论viewpager是在activity哪个生命周期里初始化。
        3. activity生命周期 和 fragment生命周期 时序并不是按序来的，也就是说fragment的oncreate方法时序并不一定在activity的oncreate方法之后。


- 单独刷新recyclerView中某个item：adapter.notifyItemChanged

- 给toolbar 上的item添加长按事件，不能直接set监听器，需要在线程中set：
    ```java
    @Override
    public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
        super.onCreateOptionsMenu(menu, inflater);
        inflater.inflate(R.menu.post_toolbar, menu);
        new Handler().post(new Runnable(){
            @Override
            public void run(){
                View menu_more=getActivity().findViewById(R.id.pick_image);
                menu_more.setOnLongClickListener(new View.OnLongClickListener(){
                    @Override
                    public boolean onLongClick(View v){
                        Toast.makeText(getActivity(),"123123",Toast.LENGTH_SHORT).show();
                        return true;
                    }
                });
            }
        });
    }
    ```

- R爆红的问题，这是经常容易在开发过程中突然发生的现象，刚刚解决了，是xml文件中有引入了一个图片，但是我直接把这个图片删了，就导致了R命名文件的错乱，以后遇到这问题排查一下找到删掉文件之前是否被使用过即可。

- retrofit使用[相关参数详解](https://blog.csdn.net/qq_40543575/article/details/79183601)
    - 网络请求方法

        ![](/img/in-post/post-Android/final_project/8_1.png)
    - 网络请求规则 

        ![](/img/in-post/post-Android/final_project/8_2.png)
    
    - 标记

        ![](/img/in-post/post-Android/final_project/8_3.png)

    - 网络请求参数

        ![](/img/in-post/post-Android/final_project/8_4.png)

- recyclerView的item[局部刷新](https://blog.csdn.net/OneDeveloper/article/details/79721284)
    - `notifyItemChanged(int position, @Nullable Object payload)` 回调 `onBindViewHolder(@NonNull VH holder, int position,@NonNull List<Object> payloads)` 方法

- 下拉刷新：安卓自带的下拉刷新布局：SwipeRefreshLayout
    - [SwipeRefreshLayout使用详解](https://blog.csdn.net/etwge/article/details/80136513)
    - [几种强大的下拉刷新库](https://www.cnblogs.com/foxy/p/7825073.html)
    - 使用SwipeRefreshLayout记得里面只能有单一的listView或者recyclerView，有其他组件的话会导致下拉布局不能正确监控list的下滑动作。

- [安卓开发中非常炫的效果集合](https://www.cnblogs.com/ldq2016/p/5217590.html)


- adater适配器的问题，在做评论的删除回复部分时，我发现回复删除操作之后，adapter都无法立即更新，但是postman获取下来的post信息中，是更新了的，说明 `commentAdapter.notifyDataSetChanged();` 这条指令没有起作用。
    - 一开始为了方便，我是把commentAdapter作为一个私有成员变量来使用的，当我把删除回复评论的函数进行修改，改为传入一个commentAdapter 参数来使用，而不是直接使用私有成员变量，这样不论在哪里创建的adapter都能被正确传入对应的触发函数中，保证了适配器的准确性。
        ```java
        private void showCommentDeleteDialog(final CommentAdapter commentAdapter, final int commentId, final int postIndex, final int commentIndex)
            ······
        ```
    - 后来发现，私有成员变量的方式其实并没有问题，是我`setAdapter(commentAdapter);`的时机不对，在点击事件发生之后，才把适配器set上去，这样的适配器肯定无法与其绑定的数据保证实时更新。

- [Android 布局平铺展开效果的属性动画](https://blog.csdn.net/debbytang/article/details/68496728)
    - 修改成我们需要的从左至右平铺的一个动画之后，有个bug，平铺之前会闪烁一下，用户体验不是很好。经过排查找到罪魁祸首，就是setVisibility为Visible的时候，会先让要显示的两个按钮完整的显示出来，然后才开始执行动画。

    - 琢磨了一下，终于找出问题所在：`v.setLayoutParams(layoutParams);`，原来的实现方式如下
        ```java
        ValueAnimator animator = createDropAnimator(view, 0,origHeight);

        private ValueAnimator createDropAnimator(final View v, int start, int end) {
        ValueAnimator animator = ValueAnimator.ofInt(start, end);
            animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
                @Override
                public void onAnimationUpdate(ValueAnimator arg0) {
                    int value = (int) arg0.getAnimatedValue();
                    ViewGroup.LayoutParams layoutParams = v.getLayoutParams();
                    layoutParams.width = value;
                    v.setLayoutParams(layoutParams);
                }
            });
            return animator;
        }
        ```
    - setLayoutParams只是一种临时修改layout的长款的方法，其实对于要被显示的内容，只是一种临时的修改，是没有被存下来的。所以我思考之后，把布局伸展的动画改用padding来实现，这样就不会有突然闪烁的糟糕体验了。
        ```java
        ValueAnimator animator = createDropAnimator(view, 0,origHeight);
        private ValueAnimator createDropAnimator(final View v, int start, int end) {
            ValueAnimator animator = ValueAnimator.ofInt(start, end);
            animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
                @Override
                public void onAnimationUpdate(ValueAnimator arg0) {
                    int value = (int) arg0.getAnimatedValue();
                    v.setPadding(value,0,0,0);
                }
            });
            return animator;
        }
        ```