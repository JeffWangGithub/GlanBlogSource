---
title: 应用分包主dex过大那些事儿
date: 2017-03-20 10:05:41
tags: Too many classes in --main-dex-list， main dex capacity exceeded
categories: Android
---
Too many classes in --main-dex-list, main dex capacity exceeded
```
com.android.dex.DexException: Too many classes in --main-dex-list, main dex capacity exceeded
  at com.android.dx.command.dexer.Main.processAllFiles(Main.java:494)
  at com.android.dx.command.dexer.Main.runMultiDex(Main.java:332)
  at com.android.dx.command.dexer.Main.run(Main.java:243)
  at com.android.dx.command.dexer.Main.main(Main.java:214)
  at com.android.dx.command.Main.main(Main.java:106)
```
随着项目的不断更新迭代，第三方sdk的不断引入，我们虽然很早之前就进行了分包的操作，然而最近却又发生了，令人感到可恶的事情，那就是不知不觉中main-dex竟然又超了。这可如何是好，真是没有救了。好吧，不废话了，直接说一下我们的处理。
首先关注你的项目的buildToolsVersion版本号 :
一、 情况之一就是buildToolsVersion < 24
我们最初就是这种情况，于是我们商定了几个解决的方案:
a. 通过对proguard文件的处理，尽量减小keep住的代码，希望能以此保证dex中class类数量不超限。
由于之前没有对proguard文件进行限制，所以导致很多keep的范围特别广，比如keep com.google和keep com.tencent这种的，这种其实不可避免的会导致dex中可能含有一些不必要的类。
经过我们的kepp处理，确实达到了一个能够是主dex不超限的临界值，但是，这个不能解决问题，因为版本迭代更新肯定会再次变大，于是我们想到了使用b方案
b. 从主dex中移除第三方sdk中的activity以及activity引用的类
如果你去查资料你会发现，好多国外的项目也同样遇到了，这种尴尬的问题，他们的方法其实就是移除了所有的activity和activity引用类。详情查看blog: [http://blog.osom.info/2014/12/too-many-methods-in-main-dex.html](http://blog.osom.info/2014/12/too-many-methods-in-main-dex.html) .
此处我说一下Android gradle插件版本在2.0之上的解决方式， 更低版本的插件，其实解决方式更为简单，请google一下即可
```
afterEvaluate {
        tasks.matching {
            it.name.startsWith('collect') && it.name.endsWith('MultiDexComponents')
        }.each{
            println "main-dex-filter: found task $it.name"
            it.filter { name, attrs ->
                def componentName = attrs.get('android:name')
                if ('activity'.equals(name)) {
                    //注意此处的过滤条件，写你自己需要过滤的类的相关过滤条件
                    if(componentName.contains("com.netease.newsreader.activity.debug") || componentName.contains("com.netease.epay.sdk") ){
                        //包含debug和epay相关的activity
                        println "debug code and epay code : main-dex-filter: skipping, detected activity [$componentName]"
                        return false
                    }
                    else if(componentName.contains("com.netease.newsreader.activity") || componentName.contains("com.netease.nr")){
//                      当前包代码
                        println "main-dex-filter: keeping, detected $name [$componentName]"
                        return true
                    }
                    return true
                } else {
                    println "main-dex-filter: keeping, detected $name [$componentName]"
                    return true
                }
            }
        }
}
```
简单的说明一下这种方式的原理: 就是在生成主dex组件之前，我们强制过滤调含有第三方activity的类，这样，根据main-dex-list文件生成的maindexComponent就不包含第三方主dex，类分析工具根据这个生成的maindexComponent的jar包分析依赖的时候也不会包含移除的这部分activity的依赖。
注意:此处需要大致了解一下dex分包的过程， 我就大概说一下，不太理解的地方可以google一下: 1. 首先buildTools会分析proguard文件和AndroidManifest.xml文件，将四大组件、Application、instrumentation和keep住的类，生成一个componentClasses.jar(在build/intermediates/multi-dex/release/componentClasses.jar路径下)；2. 然后会根据这个jar中的类，去分析类依赖生成一个maindexList.txt文件，此文件中包含了分析出来的需要放到主dex中的类； 3. 根据这个文件进行分包的操作。

二、 buildToolsVersion > 24 的情况
省省心吧，你不会发生这个问题，buildToolsVersion >24之后，发现会吧四大组件等不必要放到主dex中的东西，默认放到其他地方，因此你的buildToolsVersion大于24，就不会发生此问题。不要问我如何发现的？其实我是搞了一周减小主dex包之后偶然发现的，当时真是蓝瘦香菇啊。
好了，到此你应该明白了，大招就是将buildToolsVersion的版本号提高到24以上。祝好吧!!
