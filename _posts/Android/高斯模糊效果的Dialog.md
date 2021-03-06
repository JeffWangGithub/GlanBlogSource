---
date: 2015-11-09 15:39
status: public
title: 高斯模糊效果的Dialog
categories: Android
---

Android默认是没有高斯模糊效果的，Android默认是含有半透明效果的Dialog。但是iOS上的弹出曾是可以只是高斯模糊效果的。自从苹果引入了高斯模糊，这个模糊的玻璃效果就变得是分流行。但是很不幸的是Android系统本身并没有提供这种API。
其实原则上讲，Android是不推荐使用高斯模糊的，因为实现高斯模糊其实是比较耗资源的。但是非要实现也是有办法的。
###  下边实现含有高斯模糊效果的Dialog
![](调起微信朋友圈，分享多张图片和描述/5D4CFC7D-F98B-486E-B306-10ABA4B5C052.png?r=63)

整体思想：
    1，将Dialog设置为全屏,（把布局的根RootView可以设置为居中不全屏，达到效果上Dialog的效果）
    2，获取到Activity 的缓存背景图(类似于截屏的原理)。然后将这个图片进行高斯模糊的处理。
    3，将处理后的图片设置Dialog的背景。
### 将Activity 的缓存普进行高斯模糊处理，并去除图片状态栏的高度
```java
	private void setBlurBg() {
		//设置高斯模糊
		decorView = mActivity.getWindow().getDecorView();
		new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					decorView.setDrawingCacheQuality(View.DRAWING_CACHE_QUALITY_LOW);
					decorView.setDrawingCacheEnabled(true);
					decorView.buildDrawingCache();
					Bitmap drawingCache = decorView.getDrawingCache();
				   //将Activity	的缓存图片，去掉状态栏，状态栏的高度就是25dp					
					Bitmap bitmap1 = Bitmap.createBitmap(drawingCache, 0, DisplayUtil.dipToPixels(mActivity, 25), drawingCache.getWidth(), drawingCache.getHeight() - DisplayUtil.dipToPixels(mActivity, 25));
					final Bitmap bitmap = BlurFilterUtil.boxBlurFilterBmp(bitmap1);
					Canvas canvas = new Canvas(bitmap);
					canvas.drawARGB(220,255,255,255);//让图片绘制的更加白
					mActivity.runOnUiThread(new Runnable() {
						@Override
						public void run() {
							mRootView.setBackgroundDrawable(new BitmapDrawable(bitmap));
						}
					});
					if(!drawingCache.isRecycled() && drawingCache != null){
						drawingCache.isRecycled();
						drawingCache = null;
					}
					if(!bitmap1.isRecycled() && bitmap1 != null){
						bitmap1.isRecycled();
						bitmap1 = null;
					}
					if(!bitmap.isRecycled() && bitmap != null){
						bitmap.isRecycled();
						bitmap1 = null;
					}
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		}).start();
	}

```
### 高斯模糊的处理工具类。高斯模糊处理可以去看一下开源的库
``` java
public class BlurFilterUtil {
	// 以下代码均是高斯模糊代码
	public static Bitmap boxBlurFilterBmp(Bitmap bmp) {
		if(bmp == null || bmp.isRecycled()){
			return null;
		}
		float hRadius = 7;
		float vRadius = 7;
		int iterations = 3;
		int width = bmp.getWidth();
		int height = bmp.getHeight();
		int[] inPixels = new int[width * height];
		int[] outPixels = new int[width * height];
		bmp.getPixels(inPixels, 0, width, 0, 0, width, height);
		for (int i = 0; i < iterations; i++) {
			blur(inPixels, outPixels, width, height, hRadius);
			blur(outPixels, inPixels, height, width, vRadius);
		}
		blurFractional(inPixels, outPixels, width, height, hRadius);
		blurFractional(outPixels, inPixels, height, width, vRadius);
		Bitmap bitmap = Bitmap.createBitmap(width, height,
				Bitmap.Config.ARGB_8888);
		bitmap.setPixels(inPixels, 0, width, 0, 0, width, height);
		Bitmap bitmap1 = Bitmap.createBitmap(bitmap, 5, 5,
				bitmap.getWidth() - 15, bitmap.getHeight() - 5);
		if (!bitmap.isRecycled()) {
			bitmap.recycle();
		}
		if (!bmp.isRecycled()) {
			bmp.recycle();
		}
		return bitmap1;
	}

	private static void blur(int[] in, int[] out, int width, int height,
			float radius) {
		int widthMinus1 = width - 1;
		int r = (int) radius;
		int tableSize = 2 * r + 1;
		int divide[] = new int[256 * tableSize];

		for (int i = 0; i < 256 * tableSize; i++)
			divide[i] = i / tableSize;

		int inIndex = 0;

		for (int y = 0; y < height; y++) {
			int outIndex = y;
			int ta = 0, tr = 0, tg = 0, tb = 0;

			for (int i = -r; i <= r; i++) {
				int rgb = in[inIndex + clamp(i, 0, width - 1)];
				ta += (rgb >> 24) & 0xff;
				tr += (rgb >> 16) & 0xff;
				tg += (rgb >> 8) & 0xff;
				tb += rgb & 0xff;
			}

			for (int x = 0; x < width; x++) {
				out[outIndex] = (divide[ta] << 24) | (divide[tr] << 16)
						| (divide[tg] << 8) | divide[tb];
				int i1 = x + r + 1;
				if (i1 > widthMinus1)
					i1 = widthMinus1;
				int i2 = x - r;
				if (i2 < 0)
					i2 = 0;
				int rgb1 = in[inIndex + i1];
				int rgb2 = in[inIndex + i2];

				ta += ((rgb1 >> 24) & 0xff) - ((rgb2 >> 24) & 0xff);
				tr += ((rgb1 & 0xff0000) - (rgb2 & 0xff0000)) >> 16;
				tg += ((rgb1 & 0xff00) - (rgb2 & 0xff00)) >> 8;
				tb += (rgb1 & 0xff) - (rgb2 & 0xff);
				outIndex += height;
			}
			inIndex += width;
		}
	}

	private static void blurFractional(int[] in, int[] out, int width,
			int height, float radius) {
		radius -= (int) radius;
		float f = 1.0f / (1 + 2 * radius);
		int inIndex = 0;
		for (int y = 0; y < height; y++) {
			int outIndex = y;
			out[outIndex] = in[0];
			outIndex += height;
			for (int x = 1; x < width - 1; x++) {
				int i = inIndex + x;
				int rgb1 = in[i - 1];
				int rgb2 = in[i];
				int rgb3 = in[i + 1];

				int a1 = (rgb1 >> 24) & 0xff;
				int r1 = (rgb1 >> 16) & 0xff;
				int g1 = (rgb1 >> 8) & 0xff;
				int b1 = rgb1 & 0xff;
				int a2 = (rgb2 >> 24) & 0xff;
				int r2 = (rgb2 >> 16) & 0xff;
				int g2 = (rgb2 >> 8) & 0xff;
				int b2 = rgb2 & 0xff;
				int a3 = (rgb3 >> 24) & 0xff;
				int r3 = (rgb3 >> 16) & 0xff;
				int g3 = (rgb3 >> 8) & 0xff;
				int b3 = rgb3 & 0xff;
				a1 = a2 + (int) ((a1 + a3) * radius);
				r1 = r2 + (int) ((r1 + r3) * radius);
				g1 = g2 + (int) ((g1 + g3) * radius);
				b1 = b2 + (int) ((b1 + b3) * radius);
				a1 *= f;
				r1 *= f;
				g1 *= f;
				b1 *= f;
				out[outIndex] = (a1 << 24) | (r1 << 16) | (g1 << 8) | b1;
				outIndex += height;
			}
			out[outIndex] = in[width - 1];
			inIndex += width;
		}
	}
	private static int clamp(int x, int a, int b) {
		return (x < a) ? a : (x > b) ? b : x;
	}
}
```