---
date: 2015-04-14 13:00
status: public
title: 'onNewIntent(Intent intent)和Activity的启动模式'
categories: Android
---

### onNewIntent()方法使用场景
当一个应用的某个Activity提供多种调用启动的情况（当前应用调用，其他应用通过隐式Intent调用，或者WAP页通过scheme调用），多个调用希望只有一个Activity的实例存在，这时候就需要Activity的onNewIntent(Intent intent)方法了。
### onNewIntent()方法的使用
>
1 在清单文件（AndroidMainfest.xml）中，对应的Activity上加上启动模式lanuchMode="singleTask".
2,在Activity中复写onNewIntent(Intent intent)方法，在此方法中调用setIntent(intent)方法，将新的Intent赋值给Activity的Intent。

### onNewIntent()调用时机解释
>
onNewIntent（）非常好用，Activity第一启动的时候执行onCreate()---->onStart()---->onResume()等后续生命周期函数，也就时说第一次启动Activity并不会执行到onNewIntent(). 
而后面如果再有想启动Activity的时候，那就是执行onNewIntent()---->onResart()------>onStart()----->onResume().  如果android系统由于内存不足把已存在Activity释放掉了，那么再次调用的时候会重新启动Activity即执行onCreate()---->onStart()---->onResume()等。

### Activity的启动模式简介
>
 Activity启动模式设置：
<activity android:name=".MainActivity"android:launchMode="standard"/>
        Activity的四种启动模式：
        1.standard
        默认启动模式，每次激活Activity时都会创建Activity，并放入任务栈中。
        2.singleTop
        如果在任务的栈顶正好存在该Activity的实例， 就重用该实例，否者就会创建新的实例并放入栈顶(即使栈中已经存在该Activity实例，只要不在栈顶，都会创建实例)。
        3.singleTask
        如果在栈中已经有该Activity的实例，就重用该实例(会调用实例的onNewIntent())。重用时，会让该实例回到栈顶，因此在它上面的实例将会被移除栈。如果栈中不存在该实例，将会创建新的实例放入栈中。
        4.singleInstance
        在一个新栈中创建该Activity实例，并让多个应用共享改栈中的该Activity实例。一旦改模式的Activity的实例存在于某个栈中，任何应用再激活改Activity时都会重用该栈中的实例，其效果相当于多个应用程序共享一个应用，不管谁激活该Activity都会进入同一个应用中     





