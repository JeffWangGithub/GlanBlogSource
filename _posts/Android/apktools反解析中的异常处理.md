---
title: apktools反解析中的异常处理
date: 2016-07-20 16:42:45
categories: Android
tags: apktools
---
最近在改打包脚本，有一个需求是想根据渠道来修改AndroidManifest.xml文件，查了一些文档，基本上是没有什么好的办法，最后只能上apktools进行反解析，然后进行回编译。然而使用的apktools进行反编译时，遇到了以下异常。google了一下，并从apktools官网查了一下文档了解到次异常是由于反编译的apk文件中使用了系统的资源，导致了反编译时找不到此资源。
apktools反编译工具反解析的时报如下异常
![](apktools反解析中的异常处理/Snip20160720_5.png)
想要解决此异常，其实只需要把系统的资源的导出来用就行, 需要的资源在framework-res.apk中，`framework-res.apk一般存放于手机的/system目录下`。下边晒官网的解决方式。
[apktools官网](https://ibotpeaches.github.io/Apktool/)
![](apktools反解析中的异常处理/Snip20160720_7.png)

