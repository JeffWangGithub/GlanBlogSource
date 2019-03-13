---
date: 2016-01-12 16:28
status: public
title: 'Android的Window和Window Manger'
categories: Android
---

WindowManger是外界访问Window的入口，Window的具体实现再WindowmanagerService中，Window Manger和WindowManagerService的交互是一个IPC过程。
### Window Manager的功能
WindowManger提供的功能其实很简单：开发者用到的基本就三个功能，即添加View，更新View和删除View。
Window Manger实现了ViewManager的接口，addView(), updateViewLayout()和removeView()这三个方法都是定义的ViewManager中的。
### Window的内部机制（Window和View的关系）
每一个Window都对应这一个View和一个ViewRootImpl，Window和View是通过ViewRootImpl来建立联系的，因此Window并不是实际存在的，它是以View的形式存在的。View是Android中试图的成仙方式。
### Activity 与View显示
    1. Activity的setContentView(View view) 方法将需要显示的View设置给Window。
    2. Window内部的实现，其实是PhoneWindow 的实现：a.先检测是否有DecoreView,如果没有创建DecoreView;b.将View    添加到DecoreView的mContentParent中；c. ActivityThread的handleResumeActivity()方法现调用activity的onResume()方法，然后调用WindowManager.addView()方法将DecoreView添加到WindowManager中。
    通过上边三个过程就解释了为什么声明周期中Resume调用的时候代表View界面显示。
具体android的源码可以自行查看。     
### Dialog和Toast的显示过程基本也是这一个过程，可以查看具体源码。