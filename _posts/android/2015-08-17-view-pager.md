---
layout: post
title: 简单的viewPager自动滑动
category: 安卓
tags: android
---

## viewPager子类的实现
下面是一个viewPager子类的实现,主要是重写了onMeasure.

```
public class MyViewPager extends ViewPager {

	public MyViewPager(Context context) {
        super(context);
    }

    public MyViewPager(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {

        int height = 0;
        for(int i = 0; i < getChildCount(); i++) {
            View child = getChildAt(i);
            child.measure(widthMeasureSpec, MeasureSpec.makeMeasureSpec(0, MeasureSpec.UNSPECIFIED));
            int h = child.getMeasuredHeight();
            if(h > height) {
            	height = h;
            }
        }

        heightMeasureSpec = MeasureSpec.makeMeasureSpec(height, MeasureSpec.EXACTLY);

        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    }
}
```

## MyPageAdapter的实现
主要重写了instantiateItem, getCount, isViewFromObject, destroyItem几个方法

```
public class MyPagerAdapter extends PagerAdapter {
		private List<ImageView> list;

		public MyPagerAdapter(List<ImageView> list) {
			this.list = list;
		}

		@Override
		public Object instantiateItem(ViewGroup container, int position) {
			container.addView(list.get(position));
			return list.get(position);
		}

		@Override
		public int getCount() {
			return list.size();
		}

		@Override
		public boolean isViewFromObject(View arg0, Object arg1) {
			return arg0 == arg1;
		}

		@Override
		public void destroyItem(ViewGroup container, int position, Object object){
			container.removeView(list.get(position));
     	}

}
```

## MyViewPagerScroller的实现

```
public class MyViewPagerScroller extends Scroller {

	private int mScrollDuration = 500; 

	public void setScrollDuration(int duration) {
		this.mScrollDuration = duration;
	}

	public MyViewPagerScroller(Context context) {
		super(context);
	}

	public MyViewPagerScroller(Context context, Interpolator interpolator) {
		super(context, interpolator);
	}

	public MyViewPagerScroller(Context context, Interpolator interpolator, boolean flywheel) {
		super(context, interpolator, flywheel);
	}

	@Override
	public void startScroll(int startX, int startY, int dx, int dy, int duration) {
		super.startScroll(startX, startY, dx, dy, mScrollDuration);
	}

	@Override
	public void startScroll(int startX, int startY, int dx, int dy) {
		super.startScroll(startX, startY, dx, dy, mScrollDuration);
	}

	public void initViewPagerScroll(ViewPager viewPager) {
		try {
			Field mScroller = ViewPager.class.getDeclaredField("mScroller");
			mScroller.setAccessible(true);
			mScroller.set(viewPager, this);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```

## 滑动的实现
这里定义了一个延迟触发的事件
```
m_handler.postDelayed(m_runnable, SLIDE_INTERVAL);
```

m_runnable的定义:
```
	m_runnable = new Runnable() {
			public void run() {
				if (m_vpPosition >= m_ivDots.length -1) {
					m_vpPosition = 0;
				} else {
					m_vpPosition++;
				}
				m_advViewPager.setCurrentItem(m_vpPosition, true);
				m_handler.postDelayed(m_runnable, SLIDE_INTERVAL);
			}
		};
```

## 对点击图片的处理
当按住图片时,停止了图片的运动.当手指离开时,重新开始移动图片
```
		m_advViewPager.setOnTouchListener(new OnTouchListener() {
			@Override
			public boolean onTouch(View v, MotionEvent e) {
				if (e.getAction() == MotionEvent.ACTION_DOWN) {

					m_handler.removeCallbacks(m_runnable);

				} else if (e.getAction() == MotionEvent.ACTION_UP || e.getAction() == MotionEvent.ACTION_OUTSIDE) {
					m_handler.postDelayed(m_runnable, SLIDE_INTERVAL);
	    }
                
				return false;
			}
            
		});
```
