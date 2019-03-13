---
date: 2016-03-30 16:43
status: public
title: android手机修改hosts
categories: Android
---

方法一：ADB 命令行替换法
将手机中的hosts文件先pull到电脑上，然后修改，最后push进手机
在Android下，/etc是link到/system/etc的，我们需要修改/system/etc/hosts来实现。但是这个文件是只读，不能通过shell直接修改

为方便操作，可以将压缩包中的adb1程序连文件夹解压缩到C盘。
步骤如下：
1、获得root权限：adb root
2、设置/system为可读写：adb remount
3、将hosts文件复制到PC：adb pull /system/etc/hosts （此时adb文件夹下已经有了复制到PC上的hosts文件）
4、修改PC机上文件
5、将PC机上文件复制到手机：adb push hosts /system/etc/hosts
如果要查看是否修改成功，可以在PC上执行adb shell，运行cat /system/etc/hosts；或者在手机上运行cat /system/etc/hosts。
方法二：简单粗暴：在手机上下载RE文件管理器，直接修改
