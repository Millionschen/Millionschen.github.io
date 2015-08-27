---
layout: post
title: 自定义view的方法
category: 安卓
tags: Essay
---

## 一. 自定义view的步骤

1. attr.xml 描述属性
2. 在布局文件中使用
3. 在构造方法中获取自定义属性
4. onMeasure
5. onDraw

下面我们以一个例子来介绍自定义view的方法

### 1.attr.xml 描述属性

```
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <attr name="icon" format="reference"/>
    <attr name="color" format="color"/>
    <attr name="text" format="string"/>
    <attr name="text_size" format="dimension"/>

    <declare-styleable name="ChangeColorIconWithText">
        <attr name="icon"/>
        <attr name="color"/>
        <attr name="text"/>
        <attr name="text_size"/>
    </declare-styleable>

</resources>
```

上面的文件首先定义了四个attribute,而后declare-styleable中说明类ChangeColorIconWithText可以使用这几个属性.format分别描述了几个属性的类型.注意text_size的类型为dimension,其他各自xxxSize的类型应当也是dimension

### 2.在布局文件中使用

首先,在最顶级的view中需要加上相关的xml名字空间:gradle中为:    `xmlns:millions="http://schemas.android.com/apk/res-auto"`

之后这些属性就可以像android:xxxx一样使用了.
```
       <com.imooc.weixin6_0.ChangeColorIconWithText
            android:id="@+id/id_indicator_one"
            android:layout_width="0dp"
            android:layout_height="fill_parent"
            android:layout_weight="1"
            android:padding="5dp"
            hyman:icon="@drawable/ic_menu_start_conversation"
            hyman:text="@string/app_name"
            hyman:text_size="12sp"
            hyman:color="#ff45c01a" />
```

### 3. 在构造方法中获取自定义属性
这里首先需要说明以下typedValue类,这个类可以用来将一些系统相关的sp,dp等值转换.
比如:
```
	private int mTextSize = (int) TypedValue.applyDimension(
			TypedValue.COMPLEX_UNIT_SP, 12, getResources().getDisplayMetrics());
```
这里我们用作示例的类是一个有图标和文字两部分组成的类,文字和图标都可以设置颜色以及颜色的alpha值.

我们在这里重写了view的三个构造函数,分别需要1,2,3个参数.我们的代码主要在三个参数的构造函数里

首先,调用父函数的构造函数,并用一个`typedArra` 获取所有自定义的type们

```
		super(context, attrs, defStyleAttr);
		TypedArray a = context.obtainStyledAttributes(attrs,
				R.styleable.ChangeColorIconWithText);
```

然后 获取自定义属性的个数和值

```
		int n = a.getIndexCount();
		for (int i = 0; i < n; i++)
		{
			int attr = a.getIndex(i);
			switch (attr)
			{
			case R.styleable.ChangeColorIconWithText_icon:
				BitmapDrawable drawable = (BitmapDrawable) a.getDrawable(attr);
				mIconBitmap = drawable.getBitmap();
				break;
			case R.styleable.ChangeColorIconWithText_color:
				mColor = a.getColor(attr, 0xFF45C01A);
				break;
			case R.styleable.ChangeColorIconWithText_text:
				mText = a.getString(attr);
				break;
			case R.styleable.ChangeColorIconWithText_text_size:
				mTextSize = (int) a.getDimension(attr, TypedValue
						.applyDimension(TypedValue.COMPLEX_UNIT_SP, 12,
								getResources().getDisplayMetrics()));
				break;
			}

		}
		//a不使用后回收节约内存
		a.recycle();
```

之后为了onDraw初始化一些成员变量的值

```
		mTextBound = new Rect();
		mTextPaint = new Paint();
		mTextPaint.setTextSize(mTextSize);
		mTextPaint.setColor(0Xff555555);
		mTextPaint.getTextBounds(mText, 0, mText.length(), mTextBound);
```
以上几个是为了画文字设置的 Rect是为了限定文字的边框,mTextPaint是在canvas上写出文字用的,设置了大小颜色以及边界等属性.

其中getTextBounds方法为:
```
 public void getTextBounds (char[] text, int index, int count, Rect bounds)

Return in bounds (allocated by the caller) the smallest rectangle that encloses all of the characters, with an implied origin at (0,0).
Parameters
text 	Array of chars to measure and return their unioned bounds
index 	Index of the first char in the array to measure
count 	The number of chars, beginning at index, to measure
bounds 	Returns the unioned bounds of all the text. Must be allocated by the caller.
```

### 4.重写onMeasure
onMeasure主要是为了计算onDraw时需要用的一些尺寸

比如这里的onMeasure可以这么写:

```
	@Override
	protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec)
	{
		super.onMeasure(widthMeasureSpec, heightMeasureSpec);
		int iconWidth = Math.min(getMeasuredWidth() - getPaddingLeft()
				- getPaddingRight(), getMeasuredHeight() - getPaddingTop()
				- getPaddingBottom() - mTextBound.height());

		int left = getMeasuredWidth() / 2 - iconWidth / 2;
		int top = getMeasuredHeight() / 2 - (mTextBound.height() + iconWidth)
				/ 2;
		mIconRect = new Rect(left, top, left + iconWidth, top + iconWidth);
	}
```
这里我对于mIconRect还有一点疑问 left和top等的值难道不是绝对的坐标么?还是只是一个相对的大小呢?需要再研究下.

### 5.onDraw

onDraw的主要在于xfermode

这里是onDraw的代码
```
		canvas.drawBitmap(mIconBitmap, null, mIconRect, null);

		int alpha = (int) Math.ceil(255 * mAlpha);

		// 内存去准备mBitmap , setAlpha , 纯色 ，xfermode ， 图标
		setupTargetBitmap(alpha);
		// 1、绘制原文本 ； 2、绘制变色的文本
		drawSourceText(canvas, alpha);
		drawTargetText(canvas, alpha);
		canvas.drawBitmap(mBitmap, 0, 0, null);
```

![Xfermodes说明](http://hi.csdn.net/attachment/201111/22/0_13219433774KaR.gif)


其中几个函数的代码:

```
    /**
	 * 绘制变色的文本
	 *
	 * @param canvas
	 * @param alpha
	 */
	private void drawTargetText(Canvas canvas, int alpha)
	{
		mTextPaint.setColor(mColor);
		mTextPaint.setAlpha(alpha);
		int x = getMeasuredWidth() / 2 - mTextBound.width() / 2;
		int y = mIconRect.bottom + mTextBound.height();
		canvas.drawText(mText, x, y, mTextPaint);

	}
```


```
	/**
	 * 绘制原文本
	 *	 * @param canvas
	 * @param alpha
	 */
	private void drawSourceText(Canvas canvas, int alpha)
	{
		mTextPaint.setColor(0xff333333);
		mTextPaint.setAlpha(255 - alpha);
		int x = getMeasuredWidth() / 2 - mTextBound.width() / 2;
		int y = mIconRect.bottom + mTextBound.height();
		canvas.drawText(mText, x, y, mTextPaint);

	}
```

```
	/**
	 * 在内存中绘制可变色的Icon
	 */
	private void setupTargetBitmap(int alpha)
	{
		mBitmap = Bitmap.createBitmap(getMeasuredWidth(), getMeasuredHeight(),
				Config.ARGB_8888);
		mCanvas = new Canvas(mBitmap);
		mPaint = new Paint();
		mPaint.setColor(mColor);
		mPaint.setAntiAlias(true);
		mPaint.setDither(true);
		mPaint.setAlpha(alpha);
		mCanvas.drawRect(mIconRect, mPaint);
		mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.DST_IN));
		mPaint.setAlpha(255);
		mCanvas.drawBitmap(mIconBitmap, null, mIconRect, mPaint);
	}
```

然后是重新绘制图片,每次先设置字节的mAlpha值,然后根据是在ui线程还是别的线程调用不同的函数重绘
```
	public void setIconAlpha(float alpha)
	{
		this.mAlpha = alpha;
		invalidateView();
	}

	/**
	 * 重绘
	 */
	private void invalidateView()
	{
		if (Looper.getMainLooper() == Looper.myLooper())
		{
			invalidate();
		} else
		{
			postInvalidate();
		}
	}
```

最后,我们可能需要在app长期处于后台被kill,以及重启时加入保存和恢复机制

```
	@Override
	protected Parcelable onSaveInstanceState()
	{
		Bundle bundle = new Bundle();
		bundle.putParcelable(INSTANCE_STATUS, super.onSaveInstanceState());
		bundle.putFloat(STATUS_ALPHA, mAlpha);
		return bundle;
	}

	@Override
	protected void onRestoreInstanceState(Parcelable state)
	{
		if (state instanceof Bundle)
		{
			Bundle bundle = (Bundle) state;
			mAlpha = bundle.getFloat(STATUS_ALPHA);
			super.onRestoreInstanceState(bundle.getParcelable(INSTANCE_STATUS));
			return;
		}
		super.onRestoreInstanceState(state);
	}
```
注意这里我们先把父类保存的参数先都存起来,再存自己的参数(mAlpha)
然后恢复时判断state是不是bundle,取出自己的数据再调用父类的函数恢复父类的数据

