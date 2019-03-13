---
date: 2015-05-26 14:05
status: public
title: WebView中实现js调用alert操作点按返回键后阻塞js的解决方法
categories: Android
---

Android Webview默认的alert弹出框是带有ip地址的，但是实际开发中我们一般都需要讲起进行自定义去除ip地址的显示。
但是如果你的alert点击按钮之后支行了js代码，就容易出现一下的坑。
状况描述：alert出现之后，不点击alert上的按钮，而是点击Android的返回键史得alert弹出框消失。此时如果你的wap页面在alert按钮点击之后掉用了js，那么你的js永远不会被执行。
解决方案：弹出的Dialog设置setCancable（false）属性，使其不能进行点击反悔的操作。
```
/**
 * 处理alert弹出框
 */
public boolean onJsAlert(WebView view, String url, String message,
                         final JsResult result) {
    MessageBoxDialog.Builder builder = new MessageBoxDialog.Builder(
            WebActivity.this, 1);
    builder.setMessageAndStyle(message, R.style.MessageBoxMessageStyle)
            .setCancelButton("知道了", R.style.CancelButtonStyle, 0, new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) {
                    result.confirm();
                }
            }).setCancelable(false)
            .create().show();
    //以上代码注意，必须设置setCancelable(false)，禁止点击返回键弹框消失，防止js阻塞
    return true;
}

/**
 * 处理confirm弹出框
 */

public boolean onJsConfirm(WebView view, String url, String message,
                           final JsResult result) {

    MessageBoxDialog.Builder builder = new MessageBoxDialog.Builder(
            WebActivity.this, 2);
    builder.setMessageAndStyle(message,
            R.style.MessageBoxMessageStyle)
            .setOtherButton(R.string.sure, R.style.DoneButtonStyle,
                    R.drawable.btn_warn_bg,
                    new DialogInterface.OnClickListener() {

                        @Override
                        public void onClick(DialogInterface dialog,
                                            int which) {

                            if (!WebActivity.this.isFinishing()) {
                                result.confirm();
                                dialog.dismiss();
                            }
                        }
                    })
            .setCancelButton(R.string.cancel,
                    R.style.CancelButtonStyle, R.drawable.btn_done_bg,
                    new DialogInterface.OnClickListener() {

                        @Override
                        public void onClick(DialogInterface dialog,
                                            int which) {
                            result.cancel();
                            dialog.dismiss();
                        }
                    }).setCancelable(false).create().show();
    //以上代码注意，必须设置setCancelable(false)，禁止点击返回键弹框消失，防止js阻塞
    return true;
}
```