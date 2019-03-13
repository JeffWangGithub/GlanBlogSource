---
date: 2015-07-27 18:59
status: public
title: Android屏幕适配全攻略(官方指导翻译)
categories: Android
---

翻译详情：
[http://blog.csdn.net/zhaokaiqiang1992/article/details/45419023](http://blog.csdn.net/zhaokaiqiang1992/article/details/45419023)

从网络获取的图片进行显示的时候，如果有需求不能写死要按照，可以根据不同dpi的手机进行代码适配
```java
 //适配不同手机的首页图标 临时解决方案 by GlanWang
float density = mContext.getResources().getDisplayMetrics().density;
if (density > 1.0 && density <= 2.0) {
    //xhdpi 2*mdpi
    size = 120;
} else if (density > 2.0 && density <= 3.0) {
    //xxhdpi 3*mdpi
    size = 180;
} else if (density <= 1.0) {
    //mdpi
    size = 60;
} else if (density > 3.0 && density == 4.0) {
    //xxxhdpi  4*mdpi
    size = 240;
}

```