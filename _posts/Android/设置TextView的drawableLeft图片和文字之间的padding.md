---
date: 2015-05-12 14:34
status: public
title: 设置TextView的drawableLeft图片和文字之间的padding
categories: Android
---

待整理

```
//设置drawableTOP图片和文字之间的padding
tv.setCompoundDrawablePadding(DisplayUtil.dip2px(10,mContext.getResources().getDisplayMetrics().density));
```
xml中使用
```
Related XML Attributes
android:drawablePadding
```

代码中设置drawableLeft/Top/Bottom/Right
```
Drawable drawableLeft = mContext.getResources().getDrawable(R.drawable.goods_prompt);
			drawableLeft.setBounds(0, 0, drawableLeft.getIntrinsicWidth(), drawableLeft.getIntrinsicHeight());
			viewHolder.good_profits.setCompoundDrawables(drawableLeft,null,null,null);
```