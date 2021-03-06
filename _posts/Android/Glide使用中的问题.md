---
title: Glide使用中的问题
date: 2017-02-27 17:49:54
tags: glide, 图片无法加载, 获取缓存图片
categories: Android
---
总结一下最近使用glide的过程中出现的两个比较棘手的问题
1. glide没有提供从缓存中找到对应资源的api。
对于这个问题，其实解决方案想到两个:第一个通过反射不改变源码的情况下能够做到；第二个只能是修改一下相关的源码，自己提供一个api让开发团队小伙伴使用。面对这两个选择我们只能选择第二个了，虽然改了源码对以后版本升级可能会出现一定的影响，但相对于反射来说其优势还是比较明显的。
```
    /**
     * add by guangcong
     * 根据url, 从本地缓存中获取到缓存的图片
     */
    public static File findDiskCache(Context context, String url) {
        OriginalKey originalKey = new OriginalKey(url, EmptySignature.obtain());
        if (DiskLruCacheWrapper.wrapper == null){
            get(context).getEngine().initDiskCacheProvider();
        }
        if(DiskLruCacheWrapper.wrapper != null){
            try {
                File file = DiskLruCacheWrapper.wrapper.get(originalKey);
                if(file != null && file.exists()){
                    return file;
                }
            } catch (Throwable e){
                e.printStackTrace();
            }
        }
        return null;
    }
```
`注意`: 1. 需要设置内存缓存包含DiskCacheStrategy.SOURCE(即至少要缓存original data)；2. 上述api需要更改某些类的修饰类型为public
2. 用户反馈图片无法展示的问题。
对于这个问题，绝逼是一个大坑啊。最初遇到这个问题，排查了所有可能性根本找不出问题所在。于是只能通过记录日志打log的方式，查找问题所在。在log中我们发现图片的请求是成功的，但是再想磁盘写的时候莫名的出错了，而且错误比较奇怪
![](Glide使用中的问题/glide1.jpg)
总是在DecodeJob的上述方法中报错 `Failed to ensureFile java.io.IOException: open failed: ENOENT (No such file or directory)` 而且文件不存在再次创建也不成功。这真是逆天了，根本不知道为什么这样。
黄天不负有心人，我组的大神竟然用一个反馈最多的机型复现了。复现步骤也很简单，就是vivo的一个手机自带了360清理软件，使用清理软件的时候回吧sdcard上我们创建的缓存目录删除掉，之后再创建就无法成功了。
到此我们也发现了问题，最主要的原因是我们使用了Context.getExternalCacheDir()作为缓存目录，此目录是在sdcard上的，清理软件会进行清理。至于为什么清理之后无法再次创建了，真个还是不太明白。于是我们吧缓存目录设置到了/data/data下，问题就完美解决了。glide默认也是缓存到/data/data下的。
