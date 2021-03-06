---
date: 2015-04-28 10:54
status: public
title: 带网格的GridView实现
categories: Android
---

开发中使用带网格的GridView效果还是比较多的，比如支付宝的客户端。
然后Google原生的GridView并未给我们提供设置网格边框的属性和方法。因此我们不得不自己去实现。
### 实现带网格GridView的方式
1 使用背景的差异（技巧）
```
1 设置GridView背景色，设置水平间方向间隔属性值android:horizontalSpacing和竖直方向间隔属性值android:verticalSpacing
2 设置GridView子项背景色
3 也就是空白的spacing那部分的颜色就是会是网格线的颜色（gridview设置的那个颜色）
```
2 给GridView的每一个子项设置边线(注意：要合理的设置每条线的隐藏和现实)
`注意事项：竖直线可能不显示` 此种方式有一点需要注意，当设置竖直的线的时候，如果你的子项布局的高度是不固定的(wrap_parent或者match_parent)，那么实际上子项是不知道自己的高度的，因此要给子项设置一个精确地值说明其高度，否则竖直线不显示。
3 自定义GridView，复写其onDraw()方法，自己绘制边线（`推荐`）
使用canvas.drawLine()方法自己在适当的位置绘制边线。
```
import android.content.Context;
import android.graphics.Canvas;
import android.graphics.Paint;
import android.util.AttributeSet;
import android.view.View;
import android.widget.GridView;

import com.meilishuo.circle.R;

/**
 * 自定义带有网格线的GridView
 * @title:
 * @description:
 * @company: 美丽说（北京）网络科技有限公司
 * Created by Jeff on 15/4/27.
 */
public class LineGridView extends GridView {
    public LineGridView(Context context) {
        super(context);
    }

    public LineGridView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public LineGridView(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
    }

    protected void dispatchDraw(Canvas canvas) {
        super.dispatchDraw(canvas);
        if (getChildAt(0) != null) {
            View localView1 = getChildAt(0);
            int column = getWidth() / localView1.getWidth();
            int childCount = getChildCount();
            int row = 0;
            if (childCount % column == 0) {
                row = childCount / column;
            } else {
                row = childCount / column + 1;
            }
            int endAllcolumn = (row - 1) * column;
            Paint localPaint, localPaint2;
            localPaint = new Paint();
            localPaint2 = new Paint();
            localPaint.setStyle(Paint.Style.STROKE);
            localPaint2.setStyle(Paint.Style.STROKE);
            localPaint.setStrokeWidth(1);
            localPaint2.setStrokeWidth(1);
//            localPaint.setColor(Color.parseColor("#C12817"));localPaint2.setColor(Color.parseColor("#F75845"));
            localPaint.setColor(getResources().getColor(R.color.home_line_color));
            localPaint2.setColor(getResources().getColor(R.color.home_line_color));
            for (int i = 0; i < childCount; i++) {
                View cellView = getChildAt(i);
                if ((i + 1) % column != 0) {
                    canvas.drawLine(cellView.getRight(), cellView.getTop(), cellView.getRight(), cellView.getBottom(), localPaint);
                    canvas.drawLine(cellView.getRight() + 1, cellView.getTop(), cellView.getRight() + 1, cellView.getBottom(), localPaint2);
                }
                if ((i + 1) <= endAllcolumn) {
                    canvas.drawLine(cellView.getLeft(), cellView.getBottom(), cellView.getRight(), cellView.getBottom(), localPaint);
                    canvas.drawLine(cellView.getLeft(), cellView.getBottom() + 1, cellView.getRight(), cellView.getBottom() + 1, localPaint2);
                }
            }
            if (childCount % column != 0) {
                for (int j = 0; j < (column - childCount % column); j++) {
                    View lastView = getChildAt(childCount - 1);
                    canvas.drawLine(lastView.getRight() + lastView.getWidth() * j, lastView.getTop(), lastView.getRight() + lastView.getWidth() * j, lastView.getBottom(), localPaint);
                    canvas.drawLine(lastView.getRight() + lastView.getWidth() * j + 1, lastView.getTop(), lastView.getRight() + lastView.getWidth() * j + 1, lastView.getBottom(), localPaint2);
                }
            }
        }
    }
}

```
![](带网格的GridView实现/gd_border.png)
4 使用ListView模仿GridView的效果
思想：ListView的每一个item都包含多个块元素，根据总数据的多少来计算现实的行数。