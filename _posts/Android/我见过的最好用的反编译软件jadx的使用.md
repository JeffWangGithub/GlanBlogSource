---
date: 2015-06-17 18:35
status: public
title: 我见过的最好用的反编译软件jadx的使用
categories: Android
---

Android的反编译软件有很多，但是大部分原理都是相同的。都是使用了apktools这一套东西讲起进行反编译的。
但是有些人开发了一些GUI的可视化工具，并增加了很多查找，搜索的功能。
jadx就是目前为止我见过的非常牛逼的，功能非常实用的反编译软件。
此开源项目的地址：[https://github.com/skylot/jadx](https://github.com/skylot/jadx)
### 使用方式：
####安装：
`直接使用git将项目下载下来，并使用gradle命令进行编译`
```c
git clone https://github.com/skylot/jadx.git
cd jadx
./gradlew dist
```
`注意：在Windows系统下, 使用gradlew.bat代替 ./gradlew`
编译后自动生成了运行的脚本。
### 运行
1. 使用一下命令行
```
cd build/jadx/
bin/jadx -d out lib/jadx-core-*.jar //out伟输出的目录
#or
bin/jadx-gui lib/jadx-core-*.jar
```
2. 直接进入到build/jadx/bin目录，直接使用终端打开jadx-gui脚本。
### 使用演示：
反编译微信为例：
具体演示了，其自带的查找和全局查找功能。

![](我见过的最好用的反编译软件jadx的使用/jadx.gif)