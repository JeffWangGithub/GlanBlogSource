---
date: 2015-05-26 14:02
status: public
title: ImageView的scaleType详解
categories: Android
---

使用ImageView时经常会用到scaleType属性，如：
<ImageView android:layout_width="50dp"
	android:layout_height="50dp" 
	android:scaleType="matrix"
	android:src="@drawable/sample_small" />
scaleType属性的各个值总是记不住之间的区别。今天找点时间总结了一下：
scaleType的属性值有：matrix   fitXY  fitStart   fitCenter  fitEnd  center   centerCrop  centerInside 
它们之间的区别如下：
matrix 用矩阵来绘制(从左上角起始的矩阵区域)
fitXY  把图片不按比例扩大/缩小到View的大小显示（确保图片会完整显示，并充满View）
fitStart  把图片按比例扩大/缩小到View的宽度，显示在View的上部分位置（图片会完整显示）
fitCenter  把图片按比例扩大/缩小到View的宽度，居中显示（图片会完整显示）
fitEnd   把图片按比例扩大/缩小到View的宽度，显示在View的下部分位置（图片会完整显示）
center  按图片的原来size居中显示，当图片宽超过View的宽，则截取图片的居中部分显示，当图片宽小于View的宽，则图片居中显示 
centerCrop  按比例扩大/缩小图片的size居中显示，使得图片的高等于View的高，使得图片宽等于或大于View的宽 
centerInside  将图片的内容完整居中显示，使得图片按比例缩小或原来的大小（图片比View小时）使得图片宽等于或小于View的宽 （图片会完整显示）
附上两张实验的截图:
![图片比ImageView大的截图](http://hi.csdn.net/attachment/201107/23/0_1311430700s5uH.gif)
](ImageView的scaleType详解/14-03-40.jpg)
 图1：　图片比ImageView大的截图
 
![图比ImageView小](http://hi.csdn.net/attachment/201107/23/0_13114307082z1x.gif)
图2： 图比ImageView小 实验截图 

----
原始地址： http://blog.csdn.net/xilibi2003/article/details/6628668