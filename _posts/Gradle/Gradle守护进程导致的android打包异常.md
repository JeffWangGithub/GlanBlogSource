---
title: Gradle守护进程导致的android打包异常
date: 2017-08-18 11:34:35
tags: gradle 守护进程，org.gradle.daemon
categories: Gradle
---
公司内部打包平台可以根据不同分支不同渠道信息进行自动化打包。昨天发版打包时竟然发现了一个特别诡异的问题导致生成的 apk 安装包直接崩溃。
### 问题描述
 先选择含有听云sdk 代码的分支进行自动化打包，打包完毕验证 apk 没有问题； 然后选择不含有听云 sdk 代码的分支进行自动化打包，打包完成，验证 apk，直接启动崩溃。查看 log
java.lang.NoClassDefFoundError: Failed resolution of: Lcom/networkbench/agent/impl/instrumentation/NBSSQLiteInstrumentation;
看到上述 log，第一反应有点懵逼，怎么会包听云 sdk 中的类呢。明明代码中没有引入听云 sdk。 难道 代码 checkout 出错了？难道 build脚本我忘记 clean Task 了？ 难道 clean 代码没有 clean 干净？...... 

### 问题排查
拿到 log 之后如果发现了异常问题， 竟然输出了很多听云 plugin 打印的 处理 class 字节码的 log( tag 为[NBSAgent.info]的 log)， 这不科学啊，因为我当前代码并未嵌入听云 sdk。 
根据 log 明确了排查方向: 1. 打包脚本没有 clean 掉上次缓存； 2. 代码 checkout 错误
** I 排查打包脚本 **
打开打包脚本排查是否进行了 clean 的操作， 现实是残忍的，打包经本没有发现问题，确实已经执行了 clean 的操作。
** II 查看代码是否正常 **
进入缓存的代码区，查看 git branch 信息以及 git log 信息， 不幸的是仍然没有发现异常。
难道是自动话打包平台除了 bug？？ 
** III 排查打包平台部署主机 **
直接在打包平台部署的主机上新 clone 一份代码，直接执行 gradle clean assembleRelease Task 直接打包。 更不幸的是，打包 log 中依然有听云嵌码的 log。验证生成的 apk 果然又发生了同样的启动崩溃。
好吧到此位置，上述原因都已经排除了。那会是什么原因呢？ 
** IV 更换主机打包 **
更换主机导没有任何异常，生成的 apk 是 ok 的。
经过上述过程的排查，原因只剩下两个: 1. 听云 的 gradle plugin 导致的问题； 2. 打包平台部署主机的环境问题。
** V 排查听云 plugin  **
想要排查这个问题，就需要知道一些 java 字节码处理的技术，以及听云 plugin 处理 class 字节码的原理。
** 听云字节码处理原理: **  
听云 plugin 字节码处理是通过Java Instrumentation + ASM 技术来实现的。这也就意味着，听云可能是将他们自己的代码注入到了 Java 进程中。 从这个原理分析，如果第一次打含听云 sdk 的代码，之后继续打不含听云 sdk 的代码，如果这个过程中 Java 进程没有死掉，也就也围着第二次很可能会被听云 plugin 进行嵌码。
验证上述猜想: 通过 ps 命令过滤出来打包主机上的 java 进程， 然后将 java 进程杀死，重新打包。 验证生成的 apk 是 ok 的。看来确实是这个原因导致的。
那么，问题回到了 Java 进程为什么在两次打包中没有死掉呢？ 
查看了主机的 gradle 配置项, 发现了以下两个配置项，gradle 守护进程和并行构建都开启了。其实，当初打开这两个配置的原因就是希望构建速度能够更快，没有考虑到会发生这种问题。
```
org.gradle.parallel=ture //并行构建启动
org.gradle.daemon=true //gradle守护进程
```
** 建议 **: debug 状态下提高构建速度，可以将这两个选项打开。 但是真正在线上机器做 release 打包时还是关闭了吧。不要仅仅为快一点，导致存在其他的风险，特别是像我们这种自动打包平台，随时都可能从任何分支进行构建 apk。

