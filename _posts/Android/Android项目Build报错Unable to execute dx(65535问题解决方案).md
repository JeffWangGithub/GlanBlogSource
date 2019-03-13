---
date: 2015-04-17 12:26
status: public
title: 'Android项目Build报错Unable to execute dx(65535问题解决方案)'
categories: Android
---

问题描述：Android项目引入过多的第三方包时，在编译成dex文件的时候，单个dex文件中的方法总数超过了65535个，此时就报错。直白了说就是单个dex文件最多能接受65535个方法。
## 解决方法：
Google已经描述了这一问题，并且颁布了对应的解决方案。使用`Android-support-multidex.jar`兼容包来解决。`注意：此包只能兼容到api 14，一次4.0一下的系统会有问题`。
方案原理：编译时发现方法总数过多时，将生成多个dex文件，这样单个文件的方法总数就不会产生65535的问题。
## # 一，非gradle构建的项目（eclipse开发项目）解决方案：
通常使用eclipse开发，都是使用的adt构建的项目，出现此问题解决方案就是
```
1 下载`Android-support-multidex.jar`兼容包。
2 将此兼容包放到libs目录下，并将jar引入过程配置。
3 如果代码有实现了Application的类，则需要将此类继承MultiDexApplication类。如果没有则在AndroidMainfest.xml中加入如下配置。
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.android.multidex.myapplication">
    <application
        ...
        android:name="android.support.multidex.MultiDexApplication">
        ...
    </application>
</manifest>
```
## # 二、gradle构建的项目(Android studio默认的构建方式)解决方案
```
思路基本相同：
1，引入兼容包Android-support-multidex.jar
    dependencies {
      compile 'com.android.support:multidex:1.0.0'
    }
2，配置Application类，或者配置清单文件
```
`详情见：`[android 官网相关介绍](https://developer.android.com/tools/building/multidex.html#mdex-gradle)
