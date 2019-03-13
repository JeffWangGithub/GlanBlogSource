---
title: Java设计模式总结---单例模式
date: 2017-09-27 18:11:27
tags: 设计模式
categories: Java
---
最近想系统的总结学习一下 java 的设计模式，于是做了计划每周搞一个。从最熟悉的开始吧-----单例 Singleton。
### 使用情景
单例目的是保证只有一个对象，所以一般使用单例有两种情况:一种真的就是需要一个对象，例如缓存，数据库，日志对象，读取配置；另一种是创建一个对象比较耗时，这时我们可以考虑使用单例，只创建一次。
### 单例的实现
单例实现分为两大类:懒汉式和饿汉式
个人习惯使用懒汉式，因为懒汉式是按需加载，这样能够减少类加载时的内存开销
### 懒汉式
####  双重检索版本
```java
public class Single {
    private static Single mInstance;
    private Single() {
    }
    public static Single getInstance() {
        if (mInstance == null) {
            synchronized (Single.class) {
                if (mInstance == null) {
                    mInstance = new Single();
                }
            }
        }
        return mInstance;
    }
}
```
解释:
第一个 if(mInstance== null) ，是为了解决效率问题，synchronized 是对效率有所影响的。
第二个if(mInstance == null)， 视为了放置可能出现多个实例的情况。
但是这样真的保证万无一失了么？？？ 其实不是的，仍然室友一定的概率出现问题。
#### volatile 版本
mInstance = new Single()；的过程其实并非是一个原子操作，再 jvm 中这句话其实做了三件事:1. 给 mInstance分配内存； 2. 给 Single 类的构造函数初执行，生成 Single 实例； 3. 将 Single对象指向分配的内存控件(指向确定后 mInstance 是非 null 的)。
由于 jvm 的即时编译器存在指令重排序的优化，最终不能保证执行顺序是1-2-3。实际执行顺序有可能是1-3-2。加入执行顺序是1-3-2 : 3执行完成时，mInstance 已经是非空了，但是其还没有进行初始化操作，这时候如果其他线程获取到直接使用就可能会出错。
当然以上情况出现的概率已经非常小了，平时开发可以忽略，但其实毕竟是有这样的问题，那么如何解决呢？ 方案很简单:给 mInstance 添加 volatile 关键字。
```java
public class Single {
    private static volatile Single mInstance;
    private Single() {
    }
    public static Single getInstance() {
        if (mInstance == null) {
            synchronized (Single.class) {
                if (mInstance == null) {
                    mInstance = new Single();
                }
            }
        }
        return mInstance;
    }
}
```
volatile保证了一个写操作完成之前，不会调用读操作。也就是说在上述1-2-3的写操作完成之前，不会调用读操作 if(mInstance==null)。
### 饿汉式
```java
public class Single {
    private static final Single mInstance = new Single();
    private Single() {
    }
    public static Single getInstance() {
        return mInstance;
    }
}
```
### 其他的单例实现方式
#### 静态内部类的方式
```java
public class Single {
    private Single() {}
    private static class SingleHolder {
        private static final Single instance = new Single();
    }

    public static Single getInstance(){
        return SingleHolder.instance;
    }
}
```
由于 SingleHolder 是一个内部类，只有再使用时才会被加载。 getInstance()方法第一次调用时才会到 SingleHolder 内部类的加载。因此，这种方式很好的避免了懒汉式的加载时直接初始化对象的弊端。
#### 枚举极简单例
```
    public enum SingleB {
        INSTANCE;
        public void fun() {
            //dongsomthing
        }
    }
```
由于创建枚举实例的过程是线程安全的，所以这种写法也没有同步的问题。 但是其缺点是不适合需要继承的场景。

参考文章[http://www.jianshu.com/p/d2755af464d2](http://www.jianshu.com/p/d2755af464d2)