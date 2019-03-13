---
date: 2015-05-29 19:15
status: public
title: 'IntellJ IDEA非gradle架构下导入v7兼容包(第三方库)'
categories: Android
---

## IntellJ IDEA导入第三方库（或者v7等安卓兼容包）的方式：
IntellJ IDEA导入第三方库都是将第三方库当做Model来进行导入的，这里与Eclipse稍有不同，但是大概的思想都是统一的。
方法过程还是以动态图的形式给出吧

![](IntellJ IDEA导入v7兼容包(第三方库)/添加v7包.gif?r=68)
1. 打开Project Structure ===> Model ===> 点击右边+号，选择Import Model ===> 将你本地的v7包导入。
2. 导入v7包的时候回生成一个名称伟lib 的Library，这个就是v7包需要的两个依赖包v4.jar和v7.jar。（如果没有自动生成，我们可以在Library中添加一个lib的Library，然后讲v4.jar和v7.jar的路径添加到右边的Class中）。
3. 给v7包导入lib依赖(即v4.jar和v7.jar的依赖)。
4. 给你需要的工程导入v7包的依赖和你的工程所需要的其他包的依赖。
5. 编译一下你的项目，是否有报错。

## 编译报错和解决办法
1. 错误一：出现某些资源无法找到的状况。
    >解决方式：修改v7项目和你的项目下的project.properties文件。讲target=android-19改为target=android-21 (注意：21或者以上版本)
![](IntellJ IDEA导入v7兼容包(第三方库)/19-38-23.jpg?r=72)

2. 错误二：编译项目的时候使用的低版本的SDK
    >选择21以上的SDK进行编译（查看与更改方法见下图）    
![](IntellJ IDEA导入v7兼容包(第三方库)/bainyi.gif?r=75)