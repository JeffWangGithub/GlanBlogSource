---
date: 2015-07-08 15:02
status: public
title: 解决scrollView中嵌套编辑框导致不能上下滑动的问题
categories: Android
---

重写EditText的onTouchEvent方法
```java
	@Override
	public boolean onTouchEvent(MotionEvent event) {
		//解决EditText嵌套到ScrollView中时不能滚动的问题
		if(event.getAction()==MotionEvent.ACTION_DOWN){
			this.getParent().requestDisallowInterceptTouchEvent(true);
		} else if(event.getAction() == MotionEvent.ACTION_UP){
			this.getParent().requestDisallowInterceptTouchEvent(false);
		}
		return super.onTouchEvent(event);
	}
```

```
mGoodsDesEditText.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View view, MotionEvent event) {
                if (view.getId() ==R.id.goods_des) {
                    view.getParent().requestDisallowInterceptTouchEvent(true);
                    switch (event.getAction()&MotionEvent.ACTION_MASK){
                        case MotionEvent.ACTION_UP:
                            view.getParent().requestDisallowInterceptTouchEvent(false);
                            break;
                    }
                }
                return false;
            }
        });

```