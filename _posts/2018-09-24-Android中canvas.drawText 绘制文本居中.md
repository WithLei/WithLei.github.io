
---
layout: post  
title:  "Android中canvas.drawText 绘制文本居中"  
date: 2018-09-24  
description: "Android中canvas.drawText 绘制文本居中"
tag: Android开发
---

# Android中canvas.drawText 绘制文本居中


因为最近多开项目，时间主要花在 coding 和 review 上了，抽空写个自定义控件中的小案例，但是虽然知识点很小但是在开发中很常用

---------------------
首先来看这个方法：
```java
drawText(String text, float x, float y, Paint paint)
```
首先第一个参数 **text** 是我们需要绘制的文本，第二、三个参数 **x,y** 是关键所在，其含义为：**x默认是这个字符串的左边在屏幕的位置，y是指定这个字符baseline在屏幕上的位置**。最后第四个参数 **paint** 是我们的画笔，用于定义字体、大小、颜色等属性。
这图用iPad随手画的，可以很清楚地看出x和y参数的意义。
![在这里插入图片描述](https://img-blog.csdn.net/20180924151430124?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODk1Mzc5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
那么怎么做才能获取正确的居中位置呢，这里需要一些运算，直接上代码：
```java
@Override
	protected void onDraw(Canvas canvas) {
		String testString = "测试文字";
		//设置要绘制的字体
		Paint mPaint = new Paint();
		mPaint.setTypeface(Typeface.create(Typeface.DEFAULT_BOLD, Typeface.BOLD));
		mPaint.setTextSize(30);
		mPaint.setColor(ThemeUtil.getThemeColor(context, R.attr.colorAccent));
		//将文字用一个矩形包裹，进而算出文字的长和宽
		Rect bounds = new Rect();
		mPaint.getTextBounds(testString, 0, testString.length(), bounds);
		FontMetricsInt fontMetrics = mPaint.getFontMetricsInt();
		//计算长宽
		int x = getMeasuredWidth() / 2 - bounds.width() / 2;
		int y = (getMeasuredHeight() - fontMetrics.bottom + fontMetrics.top) / 2 - fontMetrics.top;
		canvas.drawText(testString, x, y, mPaint);
	}
```
在刚开始写的时候，可能很多人会和我一样写成
```java
canvas.drawText(testString, getMeasuredWidth()/2 - bounds.width()/2, getMeasuredHeight()/2 + bounds.height()/2,mPaint);
```
这**不是真正的居中**，虽然 **y** 确实是位于中间，但是会导致文字下沉，其原因还是因为我上面讲的 **y 不是字符中心，而是baseline在屏幕上的位置**。