---
title: AppBarLayout的五中ScrollFlags
date: 2017-07-04 13:13:44
tags: design, appBarLayout
categories: Android
---
最近使用CoordinatorLayout和AppBarLayout，总结一下AppBarLayout的scrollFlags属性；
scrollFlags共有五中取值提供给AppBarLayout的ChildView使用， 在布局中直接使用app:layout_scroolFlags设置， 对应的职位scroll, enterAlways, enterAlwaysCollapsed, snap, exitUntilCollapsed; 代码中通过'setScrollFlags(int)'来进行设置。
### scroll
child view 伴随这滚动事件儿滚出或滚进屏幕。 注意: a. 如果使用其他值，必定要使用这个值才能起作用；b. 如果再这个chid view前面的任何Chid view没有设置这个值，那么这个child View的设置不起作用
![]( AppBarLayout的五中ScrollFlags/scroll.gif)
### enterAlways
快速返回。 其实就是向下滚动时Scrolling view和child view之前的滚动优先级的问题。 对比scroll 和scroll | enterAlways设置，发生向下滚动式时，scroll优先滚动scrolling View， 后者优先滚动child view，当滚动的一方已经全部滚进屏幕之后，另一方才开始滚动。
![]( AppBarLayout的五中ScrollFlags/enterAlways.gif)

###  enterAlwaysCollapsed
enterAlways的附加值。这里涉及到Child View的高度和最小高度，向下滚动时，Child View先向下滚动最小高度值，然后Scrolling View开始滚动，到达边界时，Child View再向下滚动，直至显示完全。
![]( AppBarLayout的五中ScrollFlags/enterAlwaysCollapsed.gif)
### exitUntilCollapsed
这里也涉及到最小高度。发生向上滚动事件时，Child View向上滚动退出直至最小高度，然后Scrolling View开始滚动。`也就是，Child View不会完全退出屏幕``。
![]( AppBarLayout的五中ScrollFlags/exitUntilCollapsed.gif)

### snap
类似于viewPager的吸附效果
简单理解，就是Child View滚动比例的一个吸附效果。也就是说，Child View不会存在局部显示的情况，滚动Child View的部分高度，当我们松开手指时，Child View要么向上全部滚出屏幕，要么向下全部滚进屏幕，有点类似ViewPager的左右滑动。
![]( AppBarLayout的五中ScrollFlags/snap.gif)

原文连接: [http://www.jianshu.com/p/7caa5f4f49bd](http://www.jianshu.com/p/7caa5f4f49bd)