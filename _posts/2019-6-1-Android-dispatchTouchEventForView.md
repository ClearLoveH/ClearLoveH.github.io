---
layout:     post
title:      "View 的事件分发机制"
date:       2019-6-1 10:29
author:     "Heng"
header-img: "img/比尔吉沃特1.jpg"
catalog: true
tags:
    - Android
---

>儿童节快乐

---
### 概念
所谓`点击事件(Touch)的事件分发`，其实就是对`MotionEvent(Touch的封装)`事件的分发过程，即当一个MotionEvent产生以后，系统需要把这这个事件传递给那个具体的View。这个传递的过程就是事件分发过程。

---

### MotionEvent事件

我们对屏幕的点击，滑动，抬起等一系的动作都是由一个一个MotionEvent对象组成的。根据不同动作，主要有以下三种事件类型：
1. `ACTION_DOWN`：手指刚接触屏幕，按下去的那一瞬间产生该事件
2. `ACTION_MOVE`：手指在屏幕上移动时候产生该事件
3. `ACTION_UP`：手指从屏幕上松开的瞬间产生该事件
4. `ACTION_CANCEL`：事件被上层拦截时触发。

从ACTION_DOWN开始到ACTION_UP结束我们称为一个事件序列

正常情况下，无论你手指在屏幕上有多么骚的操作，最终呈现在MotionEvent上来讲无外乎下面两种。
1. 点击后抬起，也就是`单击操作`：**ACTION_DOWN -> ACTION_UP**
2. 点击后再风骚的滑动一段距离，再抬起：**ACTION_DOWN -> ACTION_MOVE -> ... -> ACTION_MOVE -> ACTION_UP**

---

### 为什么要有事件分发机制？

安卓上面的View是`树形结构`的，View可能会重叠在一起，当我们点击的地方有多个View都可以响应的时候，这个点击事件应该给谁呢？为了解决这一个问题，就有了事件分发机制。

如下图，View是一层一层嵌套的，当手指点击 View1 的时候，下面的ViewGroupA、 RootView 等也是能够响应的，为了`确定到底应该是哪个View处理这次点击事件，就需要事件分发机制来帮忙`。

![](/img/in-post/post-Android/touchEventDispatch/View1.jpg)


---
### View 的结构

**layout 文件：**

```xml
<com.gcssloop.touchevent.test.RootView
	xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:tools="http://schemas.android.com/tools"
	android:layout_width="match_parent"
	android:layout_height="300dp"
	android:background="#4E5268"
	android:layout_margin="20dp"
	tools:context="com.gcssloop.touchevent.MainActivity">

	<com.gcssloop.touchevent.test.ViewGroupA
		android:background="#95C3FA"
		android:layout_width="200dp"
		android:layout_height="200dp">

		<com.gcssloop.touchevent.test.View1
			android:background="#BDDA66"
			android:layout_width="130dp"
			android:layout_height="130dp"/>

	</com.gcssloop.touchevent.test.ViewGroupA>

	<com.gcssloop.touchevent.test.View2
		android:layout_alignParentRight="true"
		android:background="#BDDA66"
		android:layout_width="80dp"
		android:layout_height="80dp"/>

</com.gcssloop.touchevent.test.RootView>
```

**View结构图：**

![](/img/in-post/post-Android/touchEventDispatch/View2.jpg)

由上结构图可以看到，`PhoneWindow` 和 `DecorView` ，这两个我们并没有在Layout文件中定义过，但是为什么会存在呢？
- 仔细观察上面的 layout 文件，你会发现一个问题，我在 layout 文件中的最顶层 View(Group) 的大小并不是填满父窗体的，留下了大量的空白区域，由于我们的手机屏幕不能透明，所以这些空白区域肯定要显示一些东西，那么应该显示什么呢？
- 屏幕上没有View遮挡的部分会显示主题的颜色。不仅如此，最上面的一个标题栏也没有在 layout 文件中，这个标题栏又是显示在哪里的呢？
- 这个**主题颜色和标题栏等内容就是显示在DecorView中**的。

现在知道 DecorView 是干什么的了，那么PhoneWindow 又有什么作用？
- 要了解 PhoneWindow 是干啥的，首先要了解啥是 Window ，看官方说明：
>Abstract base class for a top-level window look and behavior policy. An instance of this class should be used as the top-level view added to the window manager. It provides standard UI policies such as a background, title area, default key processing, etc.
- 简单来说，Window是一个**抽象类**，是**所有视图的最顶层容器**，视图的外观和行为都归他管，不论是背景显示，标题栏还是事件处理都是他管理的范畴，它其实就像是View界的太上皇(虽然能管的事情看似很多，但是没实权，因为抽象类不能直接使用)。
- 而 PhoneWindow 作为 Window 的**唯一实现类**，自然就是 View 界的皇帝了， **Activity 的成员变量 mWindow 就是一个 PhoneWindow 对象**。但是 PhoneWindow 终究是 Window，它并不具备多少 View 相关的能力。不过 PhoneWindow 中持有一个 Android 中非常重要的一个 View 对象 DecorView.
- 而上面说的**DecorView 是 PhoneWindow 的一个内部类**，其职位相当于小太监，就是跟在 PhoneWindow 身边专业为 PhoneWindow 服务的，除了自己要干活之外，也负责消息的传递，`PhoneWindow 的指示通过 DecorView 传递给下面的 View，而下面 View 的信息也通过 DecorView 回传给 PhoneWindow。`

---
### 事件分发、拦截与消费

#### 三个重要方法

`public boolean dispatchTouchEvent(MotionEvent ev)`

>该方法用来进行事件的分发，即无论ViewGroup或者View的事件，都是从这个方法开始的。

`public boolean onInterceptTouchEvent(MotionEvent ev)`

>在上一个方法内部调用，表示是否拦截当前事件，返回`true表示拦截，如果拦截了事件，那么将不会分发给子View`。比如说：ViewGroup拦截了这个事件，那么所有事件都由该ViewGroup处理，它内部的子View将不会获得事件的传递。（但是ViewGroup是默认不拦截事件的，这个下面会解释。）注意：View是没有这个方法的，也即是说，继承自View的一个子View不能重写该方法，也无需拦截事件，因为它下面没有View了，它要么处理事件要么不处理事件，所以最底层的子View不能拦截事件。

`public boolean onTouchEvent(MotionEvent ev)`

>这个方法表示对事件进行处理，在dispatchTouchEvent方法内部调用，如果返回`true表示消耗当前事件`，如果返回false表示不消耗当前事件。

以上三个方法非常重要，贯穿整个View事件分发的流程，它们的关系可以用如下伪代码呈现：
```java
public boolean dispatchTouchEvent(MotionEvent ev){
    boolean handle = false;
    if(onInterceptTouchEvent(ev)){
        handle = onTouchEvent(ev);
    }else{
        handle = child.dispatchTouchEvent(ev);
    }
    return handle;
}
```

**结论**：如果一个事件传递到了ViewGroup处，首先会判断当前ViewGroup是否要拦截事件，即调用onInterceptTouchEvent()方法；如果返回true，则表示ViewGroup拦截事件，那么ViewGroup就会调用自身的onTouchEvent来处理事件；如果返回false，表示ViewGroup不拦截事件，此时事件会分发到它的子View处，即调用子View的dispatchTouchEvent方法，如此反复直到事件被消耗掉。 



类型	|相关方法	|Activity	|ViewGroup	|View
-|-|-|-|-
事件分发	|dispatchTouchEvent	|√	|√	|√
事件拦截	|onInterceptTouchEvent	|X	|√	|X
事件消费	|onTouchEvent	|√	|√	|√

这个三个方法均有一个 boolean(布尔) 类型的返回值，`通过返回 true 和 false 来控制事件传递的流程`。

从上表可以看到 Activity 和 View 都是没有事件拦截的，这是因为：
- Activity 作为原始的事件分发者，如果 Activity 拦截了事件会导致整个屏幕都无法响应事件，这肯定不是我们想要的效果。
- 而View最为事件传递的最末端，要么消费掉事件，要么不处理进行回传，根本没必要进行事件拦截。

---
### 事件分发流程
前面我们了解到了我们的View是树形结构的，基于这样的结构，我们的事件可以进行有序的分发。

事件收集之后最先传递给 Activity， 然后依次向下传递，大致如下：

>Activity －> PhoneWindow －> DecorView －> ViewGroup －> ... －> View

这样的事件分发机制逻辑非常清晰，可是，你是否注意到一个问题？如果最后分发到View，如果这个View也没有处理事件怎么办，就这样让事件浪费掉？

当然不会啦，如果`没有任何View消费掉事件，那么这个事件会按照反方向回传`，最终传回给Activity，如果最后 Activity 也没有处理，本次事件才会被抛弃:

>Activity <－ PhoneWindow <－ DecorView <－ ViewGroup <－ ... <－ View

这就是一个非常经典的**责任链模式**，如果自己能处理就拦截下来自己干，如果自己不能处理或者不确定就交给责任链中下一个对象。

这种设计是非常精巧的，**上层View既可以直接拦截该事件，自己处理，也可以先询问(分发给)子View，如果子View需要就交给子View处理，如果子View不需要还能继续交给上层View处理**。既保证了事件的有序性，又非常的灵活。

**一个使用事件分发机制的重要场景，就是scrollView 嵌套 RecyclerView时，解决滑动冲突的问题，这种时候就是在RecyclerView的父GroupView中对滑动事件进行拦截就行了。**

---
**对比实际生活场景：**

**角色：**
- Activity － 公司大老板
- RootView － 项目经理
- ViewGroupA － 技术小组长
- View1 － 码农小王(公司里唯一的码农)
- View2 － 跑龙套的路人甲，无视即可

#### 事件1：点击 View1 区域但没有任何 View 消费事件

![](/img/in-post/post-Android/touchEventDispatch/View3.jpg)

当手指在 View1 区域点击了一下之后，`如果所有View都不消耗事件，你就能看到一个完整的事件分发流程`，大致如下：

>红色箭头方向表示事件分发方向。
>绿色箭头方向表示事件回传方向。

![](/img/in-post/post-Android/touchEventDispatch/流程.jpg)
上面的流程中存在部分不合理内容，请选择性接受。

**事件顺序，老板(MainActivity)要做淘宝，这个事件通过各个部门(ViewGroup)一层一层的往下传，传到最底层的时候，码农小王(View1)发现做不了，于是消息又一层一层的回传到老板那里。**

可以看到整个事件传递路线非常有序。从Activity开始，最后回传给Activity结束(由于我们无法操作Phone Window和DecorView，所以没有它们的信息)。
```xml
MainActivity [老板]: dispatchTouchEvent     经理,我准备发展一下电商业务,下周之前做一个淘宝出来.
RootView     [经理]: dispatchTouchEvent     呼叫技术部,老板要做淘宝,下周上线.
RootView     [经理]: onInterceptTouchEvent  (老板可能疯了,但又不是我做.)
ViewGroupA   [组长]: dispatchTouchEvent     老板要做淘宝,下周上线?
ViewGroupA   [组长]: onInterceptTouchEvent  (看着不太靠谱,先问问小王怎么看)
View1        [码农]: dispatchTouchEvent     做淘宝???
View1        [码农]: onTouchEvent           这个真心做不了啊.
ViewGroupA   [组长]: onTouchEvent           小王说做不了.
RootView     [经理]: onTouchEvent           报告老板, 技术部说做不了.
MainActivity [老板]: onTouchEvent           这么简单都做不了,你们都是干啥的(愤怒).
```


#### 事件2：点击 View1 区域且事件被 View1 消费
如果事件被View1消费掉了则事件会回传告诉上层View这个事件已经被我解决了，上层View就无需再响应了。

![](/img/in-post/post-Android/touchEventDispatch/流程2.jpg)

**事件顺序，老板(MainActivity)要做改界面，这个事件通过各个部门(ViewGroup)一层一层的往下传，传到最底层的时候，码农小王(View1)就在按钮上添加了一道光(为啥是小王呢？因为公司没有设计师)。**

可以看出，`事件一旦被消费就意味着消息传递的结束，上层View知道了事件已经被消费掉，就不再处理`了。
```xml
MainActivity [老板]: dispatchTouchEvent     把按钮做的好看一点,要有光泽,给人一种点击的欲望.
RootView     [经理]: dispatchTouchEvent     技术部,老板说按钮不好看,要加一道光.
RootView     [经理]: onInterceptTouchEvent  
ViewGroupA   [组长]: dispatchTouchEvent     给按钮加上一道光.
ViewGroupA   [组长]: onInterceptTouchEvent  
View1        [码农]: dispatchTouchEvent     加一道光.
View1        [码农]: onTouchEvent           做好了.
```

#### 事件3：点击 View1 区域但事件被 ViewGroupA 拦截

上层的View有权拦截事件，不传递给下层View，例如`ListView 滑动的时候，就不会将事件传递给下层的子 View`。

![](/img/in-post/post-Android/touchEventDispatch/流程3.jpg)

可以看到，如果上层拦截了事件，下层View将接收不到事件信息。

**事件顺序，老板(MainActivity)要知道项目进度，这个事件通过各个部门(ViewGroup)一层一层的往下传，传到技术组组长(ViewGroupA)的时候，组长(ViewGroupA)上报任务即可。无需告知码农小王(View1)。**
```java
MainActivity [老板]: dispatchTouchEvent     现在项目做到什么程度了?
RootView     [经理]: dispatchTouchEvent     技术部,你们的app快做完了么?
RootView     [经理]: onInterceptTouchEvent  
ViewGroupA   [组长]: dispatchTouchEvent     项目进度?
ViewGroupA   [组长]: onInterceptTouchEvent  
ViewGroupA   [组长]: onTouchEvent           正在测试,明天就测试完了
```

#### 总结
1. 如果事件被消费，就意味着事件信息传递终止。
2. 如果事件一直没有被消费，最后会传给Activity，如果Activity也不需要就被抛弃。
3. 判断事件是否被消费是根据`返回值`，而不是根据你是否使用了事件。


---
**参考博客**：
- [安卓自定义View进阶-事件分发机制原理](https://www.gcssloop.com/customview/dispatch-touchevent-theory)
- [Android View 事件分发机制源码详解(ViewGroup篇)](https://blog.csdn.net/suyimin2010/article/details/80958205)
- [Android事件传递机制分析](http://wuxiaolong.me/2015/12/19/MotionEvent/)
