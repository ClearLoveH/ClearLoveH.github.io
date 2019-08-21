---
layout:     post
title:      "RecyclerView 与 子 View 的滑动冲突处理"
date:       2019-8-21 18:05
author:     "Heng"
header-img: "img/比尔吉沃特1.jpg"
catalog: true
tags:
    - Android
---
### 1.RecyclerView 的 onInterceptTouchEvent
在实际工作中，我们经常会遇到需要处理 **RecyclerView 和子 View 的滑动事件冲突**的情景，而 RecyclerView 去拦截滑动事件的代码在 **onInterceptTouchEvent** 方法里：
```java
case MotionEvent.ACTION_MOVE: {
    final int index = e.findPointerIndex(mScrollPointerId);
	if (index < 0) {
        Log.e(TAG, "Error processing scroll; pointer index for id "
            + mScrollPointerId + " not found. Did any MotionEvents get skipped?");
    	return false;
	}

    final int x = (int) (e.getX(index) + 0.5f);
    final int y = (int) (e.getY(index) + 0.5f);
    int dx = mLastTouchX - x;
    int dy = mLastTouchY - y;

    if (dispatchNestedPreScroll(dx, dy, mScrollConsumed, mScrollOffset, TYPE_TOUCH)) {
        dx -= mScrollConsumed[0];
        dy -= mScrollConsumed[1];
        vtev.offsetLocation(mScrollOffset[0], mScrollOffset[1]);
        // Updated the nested offsets
        mNestedOffsets[0] += mScrollOffset[0];
        mNestedOffsets[1] += mScrollOffset[1];
    }

    if (mScrollState != SCROLL_STATE_DRAGGING) {
        boolean startScroll = false;
        if (canScrollHorizontally && Math.abs(dx) > mTouchSlop) {
            if (dx > 0) {
                dx -= mTouchSlop;
            } else {
                dx += mTouchSlop;
            }
            startScroll = true;
        }
        if (canScrollVertically && Math.abs(dy) > mTouchSlop) {
            if (dy > 0) {
                dy -= mTouchSlop;
            } else {
                dy += mTouchSlop;
            }
            startScroll = true;
        }
        if (startScroll) {
            setScrollState(SCROLL_STATE_DRAGGING);
        }
    }
```

通过源码可以看到，RecyclerView 通过判断滑动的 deltaX / Y 值中有没有一个大于 **mTouchSlop** （`这个值会根据手机屏幕宽高调整，不同手机值不一样`），如果有，就会改变滑动状态，通过 `setScrollState(SCROLL_STATE_DRAGGING);` 方法来消费滑动事件，这样子View就获取不到了滑动事件了，就不存在冲突了。

---
### 2.冲突出现的场景

我们在子View中去处理 onTouch 事件时，一定要**注意与 RecyclerView 的逻辑保持统一**，从RecyclerView的源码中我们可以看到，它在计算ACTION_MOVE事件中的坐标时，使用的 x，y，dx，dy都是 int 类型，而且都有通过 **+ 0.5f** 的方式来四舍五入

若我们子View中**去计算相应的坐标没有使用 int 类型，比如 float 类型来存取坐标**，这样子View onTouch 事件中获取到的坐标会相对更加精细，而我们为了让 RecyclerView 可以正确的拦截滑动事件，我们一般也会添加一个如下的逻辑，**让子View在滑动距离小于 mTouchSlop 范围内的滑动不去响应事件**：
```java
if (absDeltaX <= mTouchSlop && absDeltaY <= mTouchSlop ) {
    return true
}
```
- 这里的 **mTouchSlop** 我们也是和RecyclerView中的slop保持一致，但是结果依旧在这里出现了滑动冲突的问题，滑动事件的冲突并没有解决干净，这是为什么？
    - 因为我们子 View 这里使用的 absDeltaX 是 float 类型的，而 RecyclerView 使用的 dx 是 int 类型的
    - 现在设 `mTouchSlop = 20`，若此时实际的水平滑动距离 `absDeltaX = 20.1`，在这样的滑动距离下，在 RecyclerView 中，我们按照他的计算方式，dx = (int) 20.1 得到的值肯定是 20，因为不满足 `Math.abs(dx) > mTouchSlop`，所以 RecyclerView 并不会去拦截这个滑动事件，那么就会被传递到子 View 中；另一方面，我们在子 View 中，因为子 View 中的 absDeltaX 因为超过了 mTouchSlop，不会 return true，所以传递下来的事件会被子View 去消费，这样就没有达到我们想要的一个效果。

---
### 3.解决方法
所以这种情况下，我们的解决方式就是通过统一子View的 x，y，deltaX，deltaY 为 int类型，来与 RecyclerView 的逻辑保持一致；或者简单的在 slop 基础上 +1 也可以解决：
```java
MotionEvent.ACTION_MOVE -> run {
    ···
    // +1是为了解决滑动冲突，RecyclerView中的deltaY是Int型，这边使用的是float，所以通过+1让RecyclerView可以正确拦截滑动事件
    if (absDeltaX <= mTouchSlop + 1 && absDeltaY <= mTouchSlop + 1) {
        return true
    }
    ···      
```

---
### 4.mTouchSlop
在解决了问题之后，其实也可以理解 RecyclerView 中为什么会有一个 **mTouchSlop** 这样的东西存在，slop就是避免 RecyclerView **去消费用户的很小幅度的误滑动作**，这样可以让用户的体验更好一些。

这种思路在我们处理许多 onTouch 事件都可以去使用，通过设定一个滑动的阈值，避免 View 对于用户非常小幅度的拖动太过敏感，进行过多的计算，从而影响性能。

像常用的调节音量、屏幕亮度的需求，我们都可以去设定恰当的 slop 值来提升性能。

