---
date: 2015-07-02 16:37
status: public
title: Zxing二维码生成的Bitmap保存到本地变全部黑色的bug解决
categories: Android
---

zxing生成二维码，背景色为黑色，设置给ImageView时能正常显示，但是当保存到本地之后就是一张纯黑色的图片。
原因：二维码色块的颜色为黑色，图片背景色也为黑色(二维码之外的背景色)。
解决方式：更改二维码之外色块的颜色。
```java
public final class EncodingHandler {
	private static final int BLACK = 0xff000000; //色块可以改变二维码的颜色
	private static final int BACKGROUNDCOLRO = 0xffffffff;//生成的Bitmap的背景色

	public static Bitmap createQRCode(String str,int widthAndHeight) throws WriterException {
		Hashtable<EncodeHintType, String> hints = new Hashtable<EncodeHintType, String>();  
        hints.put(EncodeHintType.CHARACTER_SET, "utf-8");

		BitMatrix matrix = new MultiFormatWriter().encode(str,
				BarcodeFormat.QR_CODE, widthAndHeight, widthAndHeight);
		int width = matrix.getWidth();
		int height = matrix.getHeight();
		int[] pixels = new int[width * height];
		
		for (int y = 0; y < height; y++) {
			for (int x = 0; x < width; x++) {
				if (matrix.get(x, y)) {
					pixels[y * width + x] = BLACK;
				}else{
					pixels[y * width + x] = BACKGROUNDCOLRO; //解决此Bitmap保存到本地图片变黑色的bug
				}
			}
		}
		Bitmap bitmap = Bitmap.createBitmap(width, height,
				Bitmap.Config.ARGB_8888);
		bitmap.setPixels(pixels, 0, width, 0, 0, width, height);
		return bitmap;
	}
}
```