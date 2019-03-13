---
date: 2016-01-11 12:27
status: public
title: 'Android AsyncTask的骗术'
categories: Android
---

### Why
首先，说一下标题为什么说是`骗术`? Google提供的AsyncTask，我一直认为是一个使用线程池来处理任务的并发的高性能类，淡然这句话猛一看没有什么问题。但是问题出现的我的使用上，如果你使用它时默认也是调用的execute(Params... params)这个方法，那我感觉我应该善意的提醒你“哥们你被骗了”，其实这个方法内部的执行不是并发处理的，而是串行处理的。下面我们来看一下源码。
### 源码分析
```java
    @MainThread
    public final AsyncTask<Params, Progress, Result> execute(Params... params) {
        return executeOnExecutor(sDefaultExecutor, params);
    }
```
该方法直接支行了executeOnExecutor方法，猛一看是没有什么问题。然而仔细看我们就需要研究一下`sDefaultExecutor`这个东西。
```java
    /**
     * An {@link Executor} that executes tasks one at a time in serial
     * order.  This serialization is global to a particular process.
     */
    public static final Executor SERIAL_EXECUTOR = new SerialExecutor();
    private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;
```
细心的朋友们已经看到了，从`SERIAL_EXECUTOR`的注视上看到了`executes tasks one at a time in serial order（按照序列执行任务）`。好了，如果还不死心，就只能去看`SerialExecutor`源码了。
```java
    private static class SerialExecutor implements Executor {
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
        Runnable mActive;
        public synchronized void execute(final Runnable r) {
            mTasks.offer(new Runnable() {
                public void run() {
                    try {
                        r.run();
                    } finally {
                        scheduleNext();
                    }
                }
            });
            if (mActive == null) {
                scheduleNext();
            }
        }
        protected synchronized void scheduleNext() {
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive);
            }
        }
    }
```
其实这个SerialExecutor是AsyncTask的内部类，从上边源码会发现sDefaultExecutor实际上是一个串行的线程池。先把任务插入到mTasks的队列中，没有活动的再去调用下一个。从这一点可以看出，再默认情况下AsyncTask是串行执行的。
### #验证：
```java
 @Override
    public void onClick(View v) {
        switch (v.getId()){
            case R.id.execute:
                Log.d("AsyncTaskActivity","**********execute串行执行方法**********");
                new MyAsyncTask("execute1").execute("");
                new MyAsyncTask("execute2").execute("");
                new MyAsyncTask("execute3").execute("");
                new MyAsyncTask("execute4").execute("");
                new MyAsyncTask("execute5").execute("");
                break;
            case R.id.executeOnExecutor:
                Log.d("AsyncTaskActivity","**********executeOnExcutor并发执行方法**********");
                new MyAsyncTask("executeOnExcutor1").executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR,"");
                new MyAsyncTask("executeOnExcutor2").executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR,"");
                new MyAsyncTask("executeOnExcutor3").executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR,"");
                new MyAsyncTask("executeOnExcutor4").executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR,"");
                new MyAsyncTask("executeOnExcutor5").executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR,"");
                break;
        }
    }
    class MyAsyncTask extends AsyncTask<String,Integer,String>{
        private String mName;
        public MyAsyncTask(String name ){
            this.mName = name;
        }
        @Override
        protected String doInBackground(String... params) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return mName;
        }
        @Override
        protected void onPostExecute(String s) {
            super.onPostExecute(s);
            SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            Log.d(TAG,"finish at "+ simpleDateFormat.format(new Date()));
        }
    }
```
看一下代码打印的log
```
01-07 17:06:54.628  D/AsyncTaskActivity: **********execute串行执行方法**********
01-07 17:06:55.630  D/AsyncTaskActivity: finish at 2016-01-07 17:06:55
01-07 17:06:56.635  D/AsyncTaskActivity: finish at 2016-01-07 17:06:56
01-07 17:06:57.636  D/AsyncTaskActivity: finish at 2016-01-07 17:06:57
01-07 17:06:58.638  D/AsyncTaskActivity: finish at 2016-01-07 17:06:58
01-07 17:06:59.638  D/AsyncTaskActivity: finish at 2016-01-07 17:06:59
01-07 17:07:01.980  D/AsyncTaskActivity: **********executeOnExcutor并发执行方法**********
01-07 17:07:02.982  D/AsyncTaskActivity: finish at 2016-01-07 17:07:02
01-07 17:07:02.982  D/AsyncTaskActivity: finish at 2016-01-07 17:07:02
01-07 17:07:02.982  D/AsyncTaskActivity: finish at 2016-01-07 17:07:02
01-07 17:07:02.982  D/AsyncTaskActivity: finish at 2016-01-07 17:07:02
01-07 17:07:02.982  D/AsyncTaskActivity: finish at 2016-01-07 17:07:02
```
从Log日志上可以看到execute方法是串行执行的，executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR,"")是并行执行的。
### How should we do ?
既然AsyncTask默认是串行执行的，那么我如果想要使用并行执行任务，应该怎么办呢？其实也很简单，AsyncTask已经给我们提供了，只需要简单调用就OK了。`executeOnExecutor(Executor exec, Params.. params)`该方法也是public的，因此我么直接调用它，给他传递一个Executor就OK了，这里AsyncTask默认也给我们设定了一个并行的Executor叫做`THREAD_POOL_EXECUTOR`。因此，我们只需要 ```java new MyAsyncTask().executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR,params);```就OK了.
`注意：executeOnExecutor这个方法是API 11以上才有的`
```java
    /** An {@link Executor} that can be used to execute tasks in parallel. */
    public static final Executor THREAD_POOL_EXECUTOR
            = new ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE,
                    TimeUnit.SECONDS, sPoolWorkQueue, sThreadFactory);
                        @MainThread
    public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params) {
        if (mStatus != Status.PENDING) {
            switch (mStatus) {
                case RUNNING:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task is already running.");
                case FINISHED:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task has already been executed "
                            + "(a task can be executed only once)");
            }
        }
        mStatus = Status.RUNNING;
        onPreExecute();
        mWorker.mParams = params;
        exec.execute(mFuture);
        return this;
    }
```


