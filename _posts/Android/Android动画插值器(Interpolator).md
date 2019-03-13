---
date: 2015-12-29 18:05
status: public
title: Android动画插值器(Interpolator)
categories: Android
---

## 强大的Interpolator动画插值器
Interpolator 被用来修饰动画效果，定义动画的变化率。
安卓系统默认实现了以下几种插值器：
  - `AccelerateDecelerateInterpolator` 在动画开始与结束的地方速率改变比较慢，在中间的时候加速
  - `AccelerateInterpolator`  在动画开始的地方速率改变比较慢，然后开始加速
  - `AnticipateInterpolator` 开始的时候向后然后向前甩
  - `AnticipateOvershootInterpolator` 开始的时候向后然后向前甩一定值后返回最后的值
  - `BounceInterpolator`   动画结束的时候弹起
  - `CycleInterpolator` 动画循环播放特定的次数，速率改变沿着正弦曲线
  - `DecelerateInterpolator` 在动画开始的地方快然后慢
  - `LinearInterpolator`   以常量速率改变
  - `OvershootInterpolator`    向前甩一定值后再回到原来位置
  
简单使用可以参考android提供的APIDemo，运行效果如下：
![](Android动画插值器(Interpolator)/interpolator.gif)

## OvershootInterpolator指定张力值使用
`张力值tension`可以简单的理解为向外甩出去的幅度
首先来看OvershootInterpolator类的源码
```java
    private final float mTension; //张力值
    public OvershootInterpolator() {
        mTension = 2.0f;
    }
    public OvershootInterpolator(float tension) {
        mTension = tension;
    }
```
通过构造可以发现，我们可以通过自己来指定tension的值，从而来调节向外甩出的幅度。
举个栗子：
实现如下动画效果：
![](Android动画插值器(Interpolator)/interpolator1.gif)
```java
/**
     * 启动动画
     */
    private void startAnimation() {
        if(enterAnimation1 == null){
            enterAnimation1 = new TranslateAnimation(TranslateAnimation.RELATIVE_TO_PARENT, 0, TranslateAnimation.RELATIVE_TO_PARENT, 0, TranslateAnimation.RELATIVE_TO_PARENT, 1.0f, TranslateAnimation.RELATIVE_TO_PARENT, 0);
        }
        enterAnimation1.setDuration(350);
        enterAnimation1.setStartOffset(50);
        if(enterAnimation2 == null){
            enterAnimation2 = new TranslateAnimation(TranslateAnimation.RELATIVE_TO_PARENT, 0, TranslateAnimation.RELATIVE_TO_PARENT, 0, TranslateAnimation.RELATIVE_TO_PARENT, 1.0f, TranslateAnimation.RELATIVE_TO_PARENT, 0);
        }
        enterAnimation2.setDuration(350);
        enterAnimation1.setInterpolator(new OvershootInterpolator(0.8f));//设置Overshoot插值器，并指定张力值为0.8f
        enterAnimation2.setInterpolator(new OvershootInterpolator(0.5f));

        for (int i = 0; i < mShareLayout.getChildCount(); i++) {
            View child = mShareLayout.getChildAt(i);
            if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB){
                child.setLayerType(View.LAYER_TYPE_HARDWARE,null);//使用硬件层加速动画
            }
            if(child.getVisibility() == View.VISIBLE){
                if(i % LINE_ITEM_COUNT == 0 || i % LINE_ITEM_COUNT == LINE_ITEM_COUNT - 1){
                    child.startAnimation(enterAnimation1);
                } else {
                    child.startAnimation(enterAnimation2);
                }
            }
        }
    }
```

```java
 /**
     * 移出动画
     */
    private void exitAnimation() {
        Animation exitAnimation1 = new TranslateAnimation(TranslateAnimation.RELATIVE_TO_PARENT, 0, TranslateAnimation.RELATIVE_TO_PARENT, 0, TranslateAnimation.RELATIVE_TO_PARENT, 0, TranslateAnimation.RELATIVE_TO_PARENT, 1.0f);
        exitAnimation1.setDuration(350);
        exitAnimation1.setStartOffset(50);
        exitAnimation1.setInterpolator(new AccelerateInterpolator());//设置加速插值器
        Animation exitAnimation2 = new TranslateAnimation(TranslateAnimation.RELATIVE_TO_PARENT, 0, TranslateAnimation.RELATIVE_TO_PARENT, 0, TranslateAnimation.RELATIVE_TO_PARENT, 0f, TranslateAnimation.RELATIVE_TO_PARENT, 1.0f);
        exitAnimation2.setDuration(350);
        exitAnimation2.setInterpolator(new AccelerateInterpolator());//设置加速插值器

        int childCount = mShareLayout.getChildCount();
        for (int i = 0; i < childCount; i++) {
            View child = mShareLayout.getChildAt(i);
            if(child.getVisibility() == View.VISIBLE){
                if (i % 4 == 0 || i % 4 == 3) {
                    child.startAnimation(exitAnimation1);
                } else {
                    child.startAnimation(exitAnimation2);
                }
            }
        }
    }
```