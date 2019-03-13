---
date: 2015-05-27 15:21
status: public
title: 'IntellJ IDEA清理Android冗余文件(Android Lint使用)'
categories: Android
---

## Lint官网简介
>The Android lint tool is a static code analysis tool that checks your Android project source files for potential bugs and optimization improvements for correctness, security, performance, usability, accessibility, and internationalization.

>In Android Studio, the configured lint and other IDE inspections run automatically whenever you compile your program. You can also manually run inspections in Android Studio by selecting Analyze > Inspect Code from the application or right-click menu. The Specify Inspections Scope dialog appears so you can specify the desired inspection profile and scope.

以上大概意思是：Lint是一种代码分析工具，可以帮助开发找到项目工程中存在的问题以及一些冗余的不规范的文件代码。
## IntellJ IDEA和Android Studio中使用Lint
Lint工具支持很多IDE开发工具，目前比较流行的开发工具莫过于IntellJ IDEA和Android Studio了，当然现在还有很多人在使用Eclipse进行开发。
至于Eclipse中使用Lint工具，大家可以自行百度一下，资料很多。
```
**IntellJ IDEA和Android Studio中使用：Analyze > Inspect Code from the application or right-click menu.**
```
代码分析完毕后：Lint工具会列出所有项目中存在的问题，保持未使用的资源文件，不规范的代码等等。
我们发布项目的时候都想尽量是的apk包比较小，这个时候就有必要使用Lint工具进行对代码和文件的清理。