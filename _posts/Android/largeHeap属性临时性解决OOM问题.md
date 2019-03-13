---
date: 2015-04-13 20:28
status: public
title: largeHeap属性临时性解决OOM问题
categories: Android
---

### AndroidMainfest.xml中的application标签下的android:largeHeap属性可以用于解决临时性OOM问题(注：不能从根本上解决)。

dalvik与GC相关的属性有：
dalvik.vm.heapstartsize：初始化dalvik分配的内存大小。
dalvik.vm.heapgrowthlimit：`没有在mainfest中设置android:largeheap="true"时，应用的最大内存，超过这个值限制值会有OOM产生。`
dalvik.vm.heapsize：`在mainfest中设置android:largeheap="true"时，应用的最大内存，超过这个值限制值会有OOM产生。`

`注：从根本上讲要解决大图片的OOM问题，还是要对图片进行压缩。`

