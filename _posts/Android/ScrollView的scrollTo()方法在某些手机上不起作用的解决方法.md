---
date: 2015-05-08 15:52
status: public
title: ScrollView的scrollTo()方法在某些手机上不起作用的解决方法
categories: Android
---

scrollView的scrollTo()方法在某些手机上不起作用(例如小米上有时候就不起作用，在三星note4就ok)，个人猜测是由于scrollTo()调用的时候有其他的动画在执行，导致了将其抵消了(纯属猜测，没来得及验证)。

曾苦于一时没有解决方案，未曾找到取消滑动动画的方法，后偶然发现，smoothScrollTo()方法可以打断动画，将  scrollTo()换成smoothScrollTo()方法可正常定位位置，但定位过程有动画要耗费一些时间，不是本想要的快速定位。 
经尝试发现以下写法即可满足需求： 
```
  view plaincopy  //滚动到原点    
  scrollView.scrollTo(0, 0);   
  scrollView.smoothScrollTo(0, 0);     
  //注意两个方法调用先后顺序不可颠倒
```


