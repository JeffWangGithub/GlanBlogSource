---
date: 2016-01-21 11:41
status: public
title: 'Android DexClassLoader使用'
categories: Android
---

`类加载器` 是一个java的概念：作用就是动态的装载Class文件。java中有ClassLoader类，借助这个类可以装载想要的Class文件，每个ClassLoader对象在初始化时必须制定Class文件的路径。
Android的机制稍微有点不一样，Android会将.class文件编译优化成.dex文件，因此使用java的ClassLoader就不行了，由于.dex文件的产生，想要加载.dex就需要一个可以加载.dex文件的ClassLoader，这个类在android里边就是DexClassLoader。

这个类加载器用来从.jar和.apk类型的文件内部加载classes.dex文件。通过这种方式可以用来执行非安装的程序代码，作为程序的一部分进行运行。这个装载类需要一个程序私有的，可写的文件目录去存放优化后的classes文件。通过Contexct.getDir(String, int)来创建这个目录：
     File dexOutputDir = context.getDir("dex", 0);
不要把优化优化后的classes文件存放到外部存储设备上，防代码注入攻击。

###  类装载器DexClassLoader的具体使用

```java
public void exePluginMethd() {
        try {
            //外部插件apk,zip,dex的路径
            String pluginPath = Environment.getExternalStorageDirectory() + File.separator+ "plugin.apk";
            File pluginFile = new File(pluginPath);
            Log.d(Tag, "pluginPath exists = " + pluginFile.exists());

            //dexOutPath
            File optimizedDirectoryFile = getDir("dex", 0) ;
            String dexOutPath = optimizedDirectoryFile.getAbsolutePath();

            //加载外部插件，pluginPath：外部插件apk,zip,dex的路径；dexOutPath:dexOutPath; 第三个参数：native代码的路径；
            DexClassLoader dexClassLoader = new DexClassLoader(pluginPath, dexOutPath, null, getClassLoader());

            //通过反射调用插件中的类
            Class<?> pluginClazz = dexClassLoader.loadClass("com.meilishuo.testmodel.classloader.PluginActivity");
            Object pluginObj = pluginClazz.newInstance();

            Method startPluginMethod = pluginClazz.getMethod("startThis");
            if (startPluginMethod != null) {
                startPluginMethod.invoke(pluginObj);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```
