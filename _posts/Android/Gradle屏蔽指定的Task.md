---
title: Gradle屏蔽指定的Task
date: 2017-06-13 18:32:37
tags: gradle，屏蔽，task
categories: Android
---
公司项目使用了Fabric 进行crash的统计，但是最近不知是何故，fabric的几个域名无法访问了，更悲催的是fabric的gradle插件中有个任务需要访问这个域名上传一些编译生成的信息，这就直接导致客户端无法打正式的release包，一直卡在`crashlyticsUploadDeobsRelease `这个task。
测试了各种办法:a. 公司网络有没有对fabric指定的域名做黑名单； b. 通过代理vpn访问指定fabric的域名；均没有结果。
好吧，活人不能被尿憋死，只能放大招了，屏蔽调这个task。原理很简单获取到这个task，然后将enable属性设置成false。
gradle屏蔽掉指定的task
```
allprojects {
    repositories {
        jcenter()
    }

    //skip Test tasks
    gradle.taskGraph.whenReady {
        tasks.each { task ->
            if (task.name.contains("crashlyticsUploadDeobsRelease"))
            {
                task.enabled = false
            }
        }
    }

}
```