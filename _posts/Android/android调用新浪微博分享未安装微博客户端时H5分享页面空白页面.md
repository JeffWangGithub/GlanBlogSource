---
title: android调用新浪微博分享未安装微博客户端时H5分享页面空白页面
date: 2016-08-31 13:17:49
categories: Android
---
问题描述: 最近修改客户端微博sdk，由于原来使用的是open api的分享和认证，现在准备切换为微博的sdk来实现。使用过程中出现了未安装新浪微博客户端时，web分享，oss web认证，登录页面打开空白的问题，[https://github.com/sinaweibosdk/weibo_android_sdk/issues/180](https://github.com/sinaweibosdk/weibo_android_sdk/issues/180)。
尝试第一步: 无奈之下，只能看官方demo然后对比自己的代码了。然并卵，代码改成一样的都不行。
尝试第二部: 既然这样不行，那只能自己再去写一个demo了，自己写的代码代码和有问题的工程几乎是一样的。Run Demo，见了鬼了，没有任何问题。
没有办法了，只能看官方github上的issue了，竟然周到了碰到相同问题的哥们。于是了解到了他们产生此问题的原因:`起用了webview的pauseTimers()方法`
WebView的pauseTimers()方法:
官方API解释:
`Pauses all layout, parsing, and JavaScript timers for all WebViews. This is a global requests, not restricted to just this WebView. This can be useful if the application has been paused.`
大致含义是是一个全局的设置，暂停掉所有的布局，解析以及js执行，到这儿恍然大悟。我们代码中的正文页有使用webview，并在页面onPause的使用掉用了webview的pauseTimers()方法。恰巧，调用微博sdk的h5进行登录认证的时候，正好我们的页面处于onPause的状态，导致了微博sdk中的webview停止了渲染，出现白页。
如上:解决方式，不执行webview的pauseTimers()方法(即不启用webview的暂停渲染)