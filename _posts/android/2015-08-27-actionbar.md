---
layout: post
title: 设置actionBar的overFlow
category: 安卓
tags: android
---

## 场景

安卓3.0之前,很多设备都没有永久menu键,所以action bar上需要一个overflow键.之后有了这个key,就默认不显示action bar的overflow了.为了使overflow常驻,我们可以使用反射的方法设置这个属性的值.
### 设置常驻
与这个值有关的类是`ViewConfiguration`,属性名为:sHasPermanentMenuKey

因为不能直接设置,所以:
```
	ViewConfiguration config = ViewConfiguration.get(this);
	Field menuKey = ViewConfiguration.class
		.getDeclaredField("sHasPermanentMenuKey");
	menuKey.setAccessible(true);
	menuKey.setBoolean(config, false);
```

因为这个变量是private的 所以需要设置accessible为true


### 设置显示图标

我们发现menu的实现类就是MenuBuilder 所以需要看看这个类中有和icon相关的代码没而后发现了方法 setOptionalIconsVisible 这个是一个具有package scope的所以又需要通过反射来调用之,设置icons为可见

我们重写了onMenuOpend方法 当menu的class为MenuBuilder时,就用反射获取了这个method,设置其为可调用,用Method的invoke方法调用了它,传入true作为参数

```
	/**
	 * 设置menu显示icon
	 */
	@Override
	public boolean onMenuOpened(int featureId, Menu menu)
	{

		if (featureId == Window.FEATURE_ACTION_BAR && menu != null)
		{
			if (menu.getClass().getSimpleName().equals("MenuBuilder"))
			{
				try
				{
					Method m = menu.getClass().getDeclaredMethod(
							"setOptionalIconsVisible", Boolean.TYPE);
					m.setAccessible(true);
					m.invoke(menu, true);
				} catch (Exception e)
				{
					e.printStackTrace();
				}
			}
		}

		return super.onMenuOpened(featureId, menu);
	}
```

### 设置不显示home图标
```
		getActionBar().setDisplayShowHomeEnabled(false);
```

### 自定义overflow button 图标
我们为主题添加了一项style: `android:actionOverflowButtonStyle`,这个style指定了图片的src为一个drawble
```
    <style name="AppBaseTheme" parent="android:Theme.Holo.Light.DarkActionBar">

        <!-- API 14 theme customizations can go here. -->
        <item name="android:actionOverflowButtonStyle">@style/weixinActionOverflowButtonStyle</item>
    </style>

    <style name="weixinActionOverflowButtonStyle">
        <item name="android:src">@drawable/actionbar_add_icon</item>
    </style>
```