---
title: RecyclerView绑定View时更新数据问题解决
date: 2018-02-01 09:58:18
tags:
---
线上版本报了一个很奇怪的异常`Java.lang.IllegalStateException: Cannot call this method while RecyclerView is computing a layout or scrolling at android.support.v7.widget.RecyclerView.assertNotInLayoutOrScroll`
看到这个确实是源码里抱出来的，查了很久后来发现: 在 bindView 的时候不能直接调用 notify 更新 view，如果在 bindView 的时候直接调用notify 的操作，就会出现上边异常。
解决方案:
将 notify 的操作放置到Runnable 队列里去执行，以保证本次 notify 执行完成后再执行下一次 notify 的操作。

```
@Override
public void onBindFooter(BaseRecyclerViewHolder holder, Integer footerData) {
        PageAdapter<T, HD>  adapter = getAdapter();
        if (adapter != null && adapter.getFooterData() != null && adapter.getFooterData() == BaseFooterHolder.TYPE_LOADING) {
            //加载本地数据
            Core.task().call(new Callable<D>() {
                @Override
                public D call() throws Exception {
                    return loadMoreLocalData(mNextPageIndex, getLocalPageSize());
                }
            }).enqueue(new Callback<D>() {
                @Override
                public void onSuccess(final D result) {
                    RecyclerView recyclerView = getRecyclerView();
                    if (recyclerView != null) {
                        //Java.lang.IllegalStateException: Cannot call this method while RecyclerView is computing a layout or scrolling
                        //at android.support.v7.widget.RecyclerView.assertNotInLayoutOrScroll
                        recyclerView.post(new Runnable() {
                            @Override
                            public void run() {
                                onResponse(false, false, result);
                            }
                        });
                    }
                }
            });
        }
    }
```