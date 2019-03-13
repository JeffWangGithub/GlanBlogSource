---
date: 2015-10-16 19:15
status: public
title: 'No resource found that matches the given name (at ''src'' with value '
categories: Android
---

Android编译的到时候出现了No resource found that matches the given name (at 'src' with value '@drawable/default_banner').这个异常，最开始的解决办法如下：
再Model的build.gradle中添加
```
android{
    aaptOptions.cruncherEnabled = false
}

```
但是这句代码只是关闭了aapt对资源的优化，不是根本原因。

## 根本原因查找
查找根本原因，发现是一个.png文件报错了。查找报错日志，发现几个关键点，sRGB和not a PNG file 。于是去查找sRGB的含义，sRGB指的是标准的RGB，因此恍然大悟，原来not a png file说明这个png文件不是真正的png文件，只是一个后缀名为.png的文件。于是打开此文件，果然发现是一个jpg文件。真是日了狗了，这个问题，打了好几个版本了，只是将aapt优化资源临时关闭临时解决的。
![](~/3253A5FF-3EFE-49EF-AC51-E55D2781B73E.png)
