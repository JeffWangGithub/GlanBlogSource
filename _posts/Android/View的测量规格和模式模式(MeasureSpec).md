---
date: 2015-05-06 15:47
status: public
title: View的测量规格和模式模式(MeasureSpec)
categories: Android
---

MeasureSpec.EXACTLY 和MeasureSpec.AT_MOST
EXACTLY ： 指大小为具体值和匹配父窗体。
AT_MOST ： 值包裹内容。
UNSPECIFIED ： 系统默认模式。父布局不对子元素做任何的约束，子元素可以得到任何想要的大小。
例如：
```
wrap_content  传进去的是AT_MOST
110dip或fill_parent 传入的模式是EXACTLY
2.view.layout()决定view在父view中的位置
```

------------------------------------------------------------------------
## MeasureSpec
MeasureSpec的全称是Measure Specification，意为“测量规格”。一个MeasureSpec对象封装了父布局传递给子布局的布局要求，每个MeasureSpec对象代表了一组宽度和高度要求。一个MeasureSpec对象由size和mode组成，MeasureSpec类通过将其封装在一个int值中以减少对象的分配。MeasureSpec的模式有三种：
`UNSPECIFIED`
父布局不对子元素做任何的约束，子元素可以得到任何想要的大小。
`EXACTLY`
子元素会以一个确定的值作为大小。
`AT_MOST`
子元素最多能是指定的大小。

-------------------------------------------------------------------------
## Constants
private static final int MODE_SHIFT
`该值指定了MeasureSpec三种模式切换时的进位大小，其默认值为30，意思是进位的大小为2的30次方。那么为什么会是30呢？我们知道一个int的长度大小为32位，而进位30位意思就是使用int的最高位和倒数第二位也就是32和31位作为标识位。`

private static final int MODE_MASK
该值指定了MeasureSpec在进行位运算时的遮罩常量，其默认值为0x3 << MODE_SHIFT，也就是十六进制下的0x3左移30(MODE_SHIFT)位。十六进制下的0x3换位二进制即为11，左移30位即为：1100 0000 0000 0000 0000 0000 0000 0000。那这个遮罩的作用是什么？为什么要叫遮罩呢？其实其作用非常简单，只是用1标识需要的值而用0标识不需要的值，因为1与任何数做与运算都会得到该数而0与任何数做与运算都会得到0，就这么简单，这也就是为什么称其为遮罩，因为其能通过运算将不需要的值位“隐藏”起来而将需要的值位“显示”出来。

public static final int UNSPECIFIED
该值的意义开头有讲，这里就不重复累赘了。其默认值为0 << MODE_SHIFT，也就是十进制下的0左移30(MODE_SHIFT)位。十进制下的0换位二进制依然为0，左移30位也就是31个0，这里我们为了规范，将在左端补0凑够32位：0000 0000 0000 0000 0000 0000 0000 0000。

public static final int EXACTLY
该默认值为1 << MODE_SHIFT，也就是十进制下的1左移30(MODE_SHIFT)位。十进制下的1换位二进制依然为1，左移30位也就是1后跟30个0，这里我们为了规范，将在左端补0凑够32位：0100 0000 0000 0000 0000 0000 0000 0000。

public static final int AT_MOST
该默认值为2 << MODE_SHIFT，也就是十进制下的2左移30(MODE_SHIFT)位。十进制下的2换位二进制为10，左移30位也就是10后跟30个0：1000 0000 0000 0000 0000 0000 0000 0000。

MeasureSpec的三种模式就介绍到这里，大家有木有发现？这三种模式包括MODE_MASK的值在二进制下都只占两位，移位后刚好在最高位和倒数第二位也就是32和31位上！有木有感触？结合上面对MODE_MASK的介绍自己脑补下，下面我们来看看MeasureSpec类的方法