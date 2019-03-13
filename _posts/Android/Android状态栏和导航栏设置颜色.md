---
date: 2015-10-29 18:57
status: public
title: Android状态栏和导航栏沉浸模式(设置颜色)
categories: Android
---


api 19以上支持状态栏的沉浸模式,即可以自己进行设置状态栏和导航栏的颜色.
1，首先还要再style中设置指定属性，
```xml
  <!--导航栏是否透明，是否填充底部虚拟按键栏 api 19以上有效-->  
<item name="android:windowTranslucentNavigation" tools:targetApi="kitkat">false</item>  
<!--状态栏是否有透明，是否填充状态栏区域 api 19以上有效-->  
<item name="android:windowTranslucentStatus" tools:targetApi="kitkat">true</item>
```
2，再value 19文件加中简历指定的主题
3，然后再再TopBar上指定向下便宜的padding为25dp，因为状态栏的高度为25dp