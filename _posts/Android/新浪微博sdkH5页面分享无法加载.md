---
date: 2016-03-14 12:54
status: public
title: adb无线调试
categories: Android
---

在开发Android应用时，通常情况下是通过USB数据线连接设备和计算机，但对于一些需要使用USB设备的应用，这种方法就碰到了麻烦，手机的USB接口已经和外接的USB设备连接，无法再连数据线，此时可以通过网络TCPIP的方法来进行。也就是然ADB 通过网络来连接设备，而无需USB数据线。
具体方法如下：
1. 使用USB数据线连接设备。
2. 在命令行输入adb tcpip 5555 ( 5555为端口号，可以自由指定）。
3. 断开 USB数据，此时可以连接你需要连接的|USB设备。
4. 再计算机命令行输入 adb connect <设备的IP地址>:5555
后面就可以使用ADB ，DDMS 来调试Android应用或显示Logcat 消息。
5. 如果需要恢复到USB数据线，可以在命令行输入adb usb
注： Android设备的IP地址可以在Settings->About Phone->Status 查到