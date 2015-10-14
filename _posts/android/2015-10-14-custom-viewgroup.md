---
layout: post
title: Handler与HandlerThread
category: 安卓
tags: android
---

## 1、简介

### 1.1ViewGroup职责

ViewGroup是放置View的容器，在我们写布局xml的时候，会告诉容器（凡是以layout为开头的属性，都是为用于告诉容器的），我们的宽度（layout_width）、高度（layout_height）、对齐方式（layout_gravity）等；当然还有margin等；于是乎，ViewGroup的职能为：给childView计算出建议的宽和高和测量模式 ；决定childView的位置；为什么只是建议的宽和高，而不是直接确定呢，别忘了childView宽和高可以设置为wrap_content，这样只有childView才能计算出自己的宽和高。

### 1.2 view的职责

View的职责是根据测量模式和ViewGroup给出的建议的宽和高，计算出自己的宽和高；同时还有个更重要的职责是：在ViewGroup为其指定的区域内绘制自己的形态。

### 1.3 ViewGroup和LayoutParams之间的关系

当在LinearLayout中写childView的时候，可以写layout_gravity，layout_weight属性；在RelativeLayout中的childView有layout_centerInParent属性，却没有layout_gravity，layout_weight，这是为什么呢？这是因为每个ViewGroup需要指定一个LayoutParams，用于确定支持childView支持哪些属性，比如LinearLayout指定LinearLayout.LayoutParams等。如果大家去看LinearLayout的源码，会发现其内部定义了LinearLayout.LayoutParams，在此类中，可以发现weight和gravity的身影。


## 2. View的三种测量模式
EXACTLY：表示设置了精确的值，一般当childView设置其宽、高为精确值、match_parent时，ViewGroup会将其设置为EXACTLY；

AT_MOST：表示子布局被限制在一个最大值内，一般当childView设置其宽、高为wrap_content时，ViewGroup会将其设置为AT_MOST；

UNSPECIFIED：表示子布局想要多大就多大，一般出现在AadapterView的item的heightMode中、ScrollView的childView的heightMode中；此种模式比较少见。

## 3. 相关的api以及职责

上面叙述了ViewGroup和View的职责，下面从API角度进行浅析。

View的根据ViewGroup传人的测量值和模式，对自己宽高进行确定（**onMeasure**中完成），然后在**onDraw**中完成对自己的绘制。

ViewGroup需要给View传入view的测量值和模式（**onMeasure**中完成），而且对于此ViewGroup的父布局，自己也需要在onMeasure中完成对自己宽和高的确定。此外，需要在**onLayout**中完成对其childView的位置的指定。

## 4. 例子1
需求：定义一个ViewGroup，内部可以传入0到4个childView，分别依次显示在左上角，右上角，左下角，右下角。
### 4.1、决定该ViewGroup的LayoutParams

这里我们使用的layout只支持margin即可，所以选择MarginLayoutParams

```
    @Override  
    public ViewGroup.LayoutParams generateLayoutParams(AttributeSet attrs)  
    {  
        return new MarginLayoutParams(getContext(), attrs);  
    }  
```

重写父类的该方法，返回MarginLayoutParams的实例，这样就为我们的ViewGroup指定了其LayoutParams为MarginLayoutParams。

### 4.2 onMeasure


```
   /**
	 * 计算所有ChildView的宽度和高度 然后根据ChildView的计算结果，设置自己的宽和高
	 */
	@Override
	protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec)
	{
		/**
		 * 获得此ViewGroup上级容器为其推荐的宽和高，以及计算模式
		 */
		int widthMode = MeasureSpec.getMode(widthMeasureSpec);
		int heightMode = MeasureSpec.getMode(heightMeasureSpec);
		int sizeWidth = MeasureSpec.getSize(widthMeasureSpec);
		int sizeHeight = MeasureSpec.getSize(heightMeasureSpec);


		// 计算出所有的childView的宽和高
		measureChildren(widthMeasureSpec, heightMeasureSpec);
		/**
		 * 记录如果是wrap_content是设置的宽和高
		 */
		int width = 0;
		int height = 0;

		int cCount = getChildCount();

		int cWidth = 0;
		int cHeight = 0;
		MarginLayoutParams cParams = null;

		// 用于计算左边两个childView的高度
		int lHeight = 0;
		// 用于计算右边两个childView的高度，最终高度取二者之间大值
		int rHeight = 0;

		// 用于计算上边两个childView的宽度
		int tWidth = 0;
		// 用于计算下面两个childiew的宽度，最终宽度取二者之间大值
		int bWidth = 0;

		/**
		 * 根据childView计算的出的宽和高，以及设置的margin计算容器的宽和高，主要用于容器是warp_content时
		 */
		for (int i = 0; i < cCount; i++)
		{
			View childView = getChildAt(i);
			cWidth = childView.getMeasuredWidth();
			cHeight = childView.getMeasuredHeight();
			cParams = (MarginLayoutParams) childView.getLayoutParams();

			// 上面两个childView
			if (i == 0 || i == 1)
			{
				tWidth += cWidth + cParams.leftMargin + cParams.rightMargin;
			}

			if (i == 2 || i == 3)
			{
				bWidth += cWidth + cParams.leftMargin + cParams.rightMargin;
			}

			if (i == 0 || i == 2)
			{
				lHeight += cHeight + cParams.topMargin + cParams.bottomMargin;
			}

			if (i == 1 || i == 3)
			{
				rHeight += cHeight + cParams.topMargin + cParams.bottomMargin;
			}

		}
		
		width = Math.max(tWidth, bWidth);
		height = Math.max(lHeight, rHeight);

		/**
		 * 如果是wrap_content设置为我们计算的值
		 * 否则：直接设置为父容器计算的值
		 */
		setMeasuredDimension((widthMode == MeasureSpec.EXACTLY) ? sizeWidth
				: width, (heightMode == MeasureSpec.EXACTLY) ? sizeHeight
				: height);
	}
```

### 4.3 onLayout

```
// abstract method in viewgroup
	@Override
	protected void onLayout(boolean changed, int l, int t, int r, int b)
	{
		int cCount = getChildCount();
		int cWidth = 0;
		int cHeight = 0;
		MarginLayoutParams cParams = null;
		/**
		 * 遍历所有childView根据其宽和高，以及margin进行布局
		 */
		for (int i = 0; i < cCount; i++)
		{
			View childView = getChildAt(i);
			cWidth = childView.getMeasuredWidth();
			cHeight = childView.getMeasuredHeight();
			cParams = (MarginLayoutParams) childView.getLayoutParams();

			int cl = 0, ct = 0, cr = 0, cb = 0;

			switch (i)
			{
			case 0:
				cl = cParams.leftMargin;
				ct = cParams.topMargin;
				break;
			case 1:
				cl = getWidth() - cWidth - cParams.leftMargin
						- cParams.rightMargin;
				ct = cParams.topMargin;

				break;
			case 2:
				cl = cParams.leftMargin;
				ct = getHeight() - cHeight - cParams.bottomMargin;
				break;
			case 3:
				cl = getWidth() - cWidth - cParams.leftMargin
						- cParams.rightMargin;
				ct = getHeight() - cHeight - cParams.bottomMargin;
				break;

			}
			cr = cl + cWidth;
			cb = cHeight + ct;
			childView.layout(cl, ct, cr, cb);
		}

	}
```

接下来我们测试一下下：

```
<com.millions.CustomImgContainer xmlns:android="http://schemas.android.com/apk/res/android"  
    xmlns:tools="http://schemas.android.com/tools"  
    android:layout_width="200dp"  
    android:layout_height="200dp"  
    android:background="#AA333333" >  
  
    <TextView  
        android:layout_width="50dp"  
        android:layout_height="50dp"  
        android:background="#FF4444"  
        android:gravity="center"  
        android:text="0"  
        android:textColor="#FFFFFF"  
        android:textSize="22sp"  
        android:textStyle="bold" />  
  
    <TextView  
        android:layout_width="50dp"  
        android:layout_height="50dp"  
        android:background="#00ff00"  
        android:gravity="center"  
        android:text="1"  
        android:textColor="#FFFFFF"  
        android:textSize="22sp"  
        android:textStyle="bold" />  
  
    <TextView  
        android:layout_width="50dp"  
        android:layout_height="50dp"  
        android:background="#ff0000"  
        android:gravity="center"  
        android:text="2"  
        android:textColor="#FFFFFF"  
        android:textSize="22sp"  
        android:textStyle="bold" />  
  
    <TextView  
        android:layout_width="50dp"  
        android:layout_height="50dp"  
        android:background="#0000ff"  
        android:gravity="center"  
        android:text="3"  
        android:textColor="#FFFFFF"  
        android:textSize="22sp"  
        android:textStyle="bold" />  
  
</com.millions.CustomImgContainer> 
```


这样就会出现4个角落各有一个方块的图像啦

## 5、 例子2

4中的例子比较简单，我们来一个复杂点的并且比较有用的（但是可以看出基本的步骤还是差不多滴），流式布局:

![flowLayout](http://7lrz7b.com1.z0.glb.clouddn.com/Screenshot_2015-10-14-22-01-17.jpeg?imageView2/2/w/500/h/500)

其实思路完全一样，多余的话不说了 上代码

### 5.1 定义LayoutParams

仍然选择最简单的MarginLayoutParams

```
 @Override
    public LayoutParams generateLayoutParams(AttributeSet attrs) {
        return new ViewGroup.MarginLayoutParams(
                DemoApplication.getInstance(),
                attrs);
    }
```

### 5.2 onMeasure

```
 @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        //获取父view算出的宽高 以及模式
        int sizeWidth = MeasureSpec.getSize(widthMeasureSpec);
        int sizeHeight = MeasureSpec.getSize(heightMeasureSpec);
        int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        int heightMode = MeasureSpec.getMode(heightMeasureSpec);

        Log.e(TAG, sizeWidth + "," + sizeHeight);


        //height与width由每一行view的height和width获取
        int width = 0;
        int height = 0;

        //lineWidth 和 lineHeight 表示当前行的高度和宽度
        int lineWidth = 0;
        int lineHeight = 0;

        int childNum = getChildCount();

        //计算每个子视图的宽高 从而计算包裹子视图的宽高
        for (int idx = 0; idx < childNum; idx++) {
            //获取每个child的结果measure结果
            View child = getChildAt(idx);
            measureChild(child,
                    widthMeasureSpec,
                    heightMeasureSpec);

            //计算width height lineWidth lineHeight 等参数
            MarginLayoutParams layoutParams = (MarginLayoutParams)child.getLayoutParams();
            int childWidth = layoutParams.leftMargin+
                    layoutParams.rightMargin+
                    child.getMeasuredWidth();
            int childHeight = layoutParams.topMargin+
                    layoutParams.bottomMargin+
                    child.getMeasuredHeight();

            if (childWidth+lineWidth > sizeWidth) {
                width = Math.max(width, lineWidth);
                lineHeight = childHeight;
                lineWidth = childWidth;
                height += lineHeight;
            } else {
                lineWidth += childWidth;
                lineHeight = Math.max(childHeight, lineHeight);
            }

            if (idx == childNum-1) {
                width = Math.max(width, lineWidth);
                height = height+lineHeight;
            }
        }

        setMeasuredDimension(widthMode == MeasureSpec.EXACTLY ? sizeWidth : width,
                heightMode == MeasureSpec.EXACTLY ? sizeHeight: height);
    }
```

### 5.2 onLayout

```
 //计算childView的left,top,right,bottom 这些都是相对于父ViewGroup的
    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        mLineHeights.clear();
        mAllViews.clear();

        int width = getWidth();
        int childNum = getChildCount();


        int lineWidth = 0;
        int lineHeight = 0;


        List<View> lineView = new ArrayList<>();
        //先遍历，将child view 分为若干行 记录每一行的高度
        for(int idx = 0; idx < childNum; idx++) {
            View childView = getChildAt(idx);
            MarginLayoutParams lp = (MarginLayoutParams)childView.getLayoutParams();
            if (lp.leftMargin+lp.rightMargin+childView.getMeasuredWidth()+lineWidth > width) {
                //换行 将当前一行的view加入mAllViews 重置行宽
                mAllViews.add(lineView);
                mLineHeights.add(lineHeight);
                lineWidth = 0;
                //lineHeight = 0;
                //建立新的行 避免重复使用相同的引用
                lineView = new ArrayList<>();
            }
            lineHeight = Math.max(lineHeight, childView.getMeasuredHeight()+lp.topMargin+lp.bottomMargin);
            lineWidth += childView.getMeasuredWidth()+lp.leftMargin+lp.rightMargin;
            lineView.add(childView);
        }
        mLineHeights.add(lineHeight);
        mAllViews.add(lineView);

        int lineNums = mAllViews.size();
        //当前的左边界
        int left = 0;
        //当前的上边界
        int top = 0;
        //每一行从左到右 计算边界 并调用child.layout设置边界
        for (int line = 0; line < lineNums; line++) {
            lineView =  mAllViews.get(line);
            int colNum = lineView.size();

            Log.e(TAG, "第" + line + "行 ：" + lineView.size() + " , " + lineView);
            Log.e(TAG, "第" + line + "行， ：" + lineHeight);

            for (int col = 0; col < colNum; col++) {
                View childView = lineView.get(col);
                if (childView.getVisibility() == View.GONE)
                {
                    continue;
                }
                MarginLayoutParams lp = (MarginLayoutParams) childView.getLayoutParams();
                int lc = left+lp.leftMargin;
                int rc = lc+childView.getMeasuredWidth();
                int tc = top+lp.topMargin;
                int bc = tc+childView.getMeasuredHeight();

                Log.e(TAG, childView + " , l = " + lc + " , t = " + tc + " , r ="
                        + rc + " , b = " + bc);

                childView.layout(lc, tc, rc, bc);
                left = rc+lp.rightMargin;
            }
            left = 0;
            top += mLineHeights.get(line);
        }
    }
``` 

最后我们来简单的测试一下啊

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:background="#E1E6F6"
    android:orientation="vertical">

    <com.millions.mydemos.ui.MyFlowLayout
        android:layout_width="fill_parent"
        android:layout_height="wrap_content">

        <TextView
            style="@style/text_flag_01"
            android:text="来一发么亲" />

        <TextView
            style="@style/text_flag_01"
            android:text="人生简直寂寞如雪" />

        <TextView
            style="@style/text_flag_01"
            android:text="一个避孕套引发的血案就问你怕不怕啊" />

        <TextView
            style="@style/text_flag_01"
            android:text="啪啪啪啪啪啪" />

        <TextView
            style="@style/text_flag_01"
            android:text="美好时光 冈本003" />

        <TextView
            style="@style/text_flag_01"
            android:text="努力ing" />

        <TextView
            style="@style/text_flag_01"
            android:text="I thick i can" />
    </com.millions.mydemos.ui.MyFlowLayout>

</LinearLayout>
```

搞定，其实思路也很简单 就是要考虑好怎么计算整个layout的宽高，以及每个子view应该放在相对于父viewGroup的什么位置上





