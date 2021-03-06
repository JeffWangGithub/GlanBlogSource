---
title: 使用嵌套滚动实现viewpager和header的联动
date: 2018-06-11 16:22:48
categories: Android
tags: 嵌套滚动, viewpager
---
要实现ViewPager 添加一个头部，并和 viewPager 中子 view 的联动效果；可能大家第一印象都是又得处理touch 事件了。不错，处理事件是万能的，也是所有这一些列问题的基础。不过处理事件对于很多人来说确实还是一个复杂而有难度的过程。其实 google 也意识到了这一点，之后出的一些框架和技术都是尽量做了封装，尽量避免让开发者去做这些复杂的事件。
今天我们就使用嵌套滚动实现 viewPager 添加一个 header 并实现和其子 View 的联动效果。
先看一下效果图:
![](使用嵌套滚动实现viewpager和header的联动/vph.gif)
首先不了解嵌套滚动的基本用法的同学，先看一下嵌套滚动的基本用法和几个回掉方法的含义，然后再向下看。
先说一下实现思路，然后再看代码吧
- layout 高度的重新测量
- 嵌套滚动，拦截垂直方向的滚动
- 在onNestedPreScroll()中判断是否需要滚动自己，进行 y 方向上的距离消费
- 为了能够在 stickyView 到达顶部时更顺畅的滚动子 view，还需要处理一下onNestedPreFling()方法

1 重新测量高度
```java
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int newHeight = MeasureSpec.getSize(heightMeasureSpec) - mStickyViewMarginTop;
        int newHeightSpec = MeasureSpec.makeMeasureSpec(newHeight, MeasureSpec.getMode(heightMeasureSpec));
        super.onMeasure(widthMeasureSpec, newHeightSpec);
        if (mTopView != null) {
            //不限制顶部 view 的高度
            mTopView.measure(widthMeasureSpec, MeasureSpec.makeMeasureSpec(0, MeasureSpec.UNSPECIFIED));
            ViewGroup.LayoutParams params = mViewPager.getLayoutParams();
            mMaxViewPagerHeight = Math.max(mMaxViewPagerHeight, (getMeasuredHeight() - mStickyView.getMeasuredHeight()));
            params.height = mMaxViewPagerHeight;
            setMeasuredDimension(getMeasuredWidth(), mTopView.getMeasuredHeight() + mStickyView.getMeasuredHeight() + mViewPager.getMeasuredHeight());
        }
    }
```
2 滚动方向的拦截
```java
    @Override
    public boolean onStartNestedScroll(View child, View target, int nestedScrollAxes) {
        NTLog.d(TAG, "onStartNestedScroll--" + "ViewCompat.SCROLL_AXIS_VERTICAL = " + ViewCompat.SCROLL_AXIS_VERTICAL + "; nestedScrollAxes= " + nestedScrollAxes);
        //拦截垂直滚动
        return  (nestedScrollAxes & getNestedScrollAxes()) != 0;
    }    
    @Override
    public int getNestedScrollAxes() {
        return ViewCompat.SCROLL_AXIS_VERTICAL;
    }
```
3 在onNestedPreScroll()中处理滚动
```java
    @Override
    public void onNestedPreScroll(View target, int dx, int dy, int[] consumed) {
        NTLog.d(TAG, "onNestedPreScroll");
        //上滑
        boolean hiddenTop = dy > 0 && getScrollY() < mMaxScrollY;
        //下滑动，且子 view 不可滑动了
        boolean showTop = dy < 0 && getScrollY() >= 0 && !ViewCompat.canScrollVertically(target, -1);
        if (hiddenTop || showTop) {
            // 滚动自己
            scrollBy(0, dy);
            //消费掉 dy
            consumed[1] = dy;
        }
    }
```
4 在onNestedPreFling()方法中处理 fling 的状态
```java
    @Override
    public boolean onNestedPreFling(View target, float velocityX, float velocityY) {
        NTLog.d(TAG, "onNestedPreFling");
        if (velocityY == 0) {
            return false;
        }
        mCurDirection = velocityY < 0 ? SCROLL_DOWN : SCROLL_UP;
        if (mScrollY > 0) {
            mScroller.fling(0, getScrollY(), (int)velocityX, (int)velocityY, 0, 0,
                    -Integer.MAX_VALUE, Integer.MAX_VALUE);
            invalidate(); //执行此代码是为了能够 fling 后执行一下computeScroll()方法
            return true;
        }
        return false;
    }
```
上边的在mScroller.fling()之后掉用了 invalidate 是为了能后执行一下computeScroll()方法；在computeScroll()方法中去判断是否需要子 view 或者自己滚动。computeScroll()的实现如下:
```java
@Override
    public void computeScroll() {
        if (mScroller.computeScrollOffset()) {
            final int currY = mScroller.getCurrY();
            if (mCurDirection == SCROLL_UP) {
                //手势向上
                if (isSticked()) {
                    //已经sticky,直接让子 view 滚动
                    int dy = mScroller.getFinalY() - currY;
                    if (dy > 0) {
                        int duration = mScroller.getDuration() - mScroller.timePassed();
                        mScrollable.smoothScrollBy((int) mScroller.getCurrVelocity(), dy, duration);
                        mScroller.abortAnimation();
                    }
                } else {
                    //滚动自己
                    int dy = mScroller.getCurrY() - mScrollY;
                    int toY = getScrollY() + dy;
                    scrollTo(0, toY);
                    invalidate();
                }
            } else {
                //手势向下
                if (mScrollable.isTop()) {
                    //子 view 滚动到了顶部
                    int dy = currY - mScrollY;
                    int toY = getScrollY() + dy;
                    scrollTo(0, toY);
                } else {
                    int dy = mScroller.getFinalY() - mScrollY;
                    int duration = mScroller.getDuration() - mScroller.timePassed();
                    mScrollable.smoothScrollBy(-(int)mScroller.getCurrVelocity(), dy, duration);
                }
                //刷新调用computeScroll() 方法,时时判断是否滚动 top 状态. 
                invalidate();
            }
        }
    }
```
OK, 大家可以看到使用嵌套滚动，我们可以省略一些对事件的处理，因为嵌套滚动给我们封装到了这几个回掉方法里。
源码实现:[https://github.com/JeffWangGithub/StickLayout](https://github.com/JeffWangGithub/StickLayout)
以上源码中实现了处理是事件和嵌套滚动两种方式达到我们需要的效果。原生事件处理的方案，copy 了我司一位美女程序员(单身妹子)的代码。
