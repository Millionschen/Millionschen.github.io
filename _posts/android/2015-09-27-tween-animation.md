---
layout: post
title: android补间动画(tween animation)
category: 安卓
tags: android
---

## 补间动画

与属性动画比，补间动画的使用可能更为广泛。补间动画对应的xml放置于res/anim文件夹下，常常放置于一个set内部。常见的属性有 **alpha**, **sacle**, **translate**, **rotate**几种。 下面是一个例子：

```
set android:shareInterpolator="false">
    <scale
        android:interpolator="@android:anim/accelerate_decelerate_interpolator"
        android:fromXScale="1.0"
        android:toXScale="1.4"
        android:fromYScale="1.0"
        android:toYScale="0.6"
        android:pivotX="50%"
        android:pivotY="50%"
        android:fillAfter="false"
        android:duration="700" />
    <set android:interpolator="@android:anim/decelerate_interpolator">
        <scale
           android:fromXScale="1.4"
           android:toXScale="0.0"
           android:fromYScale="0.6"
           android:toYScale="0.0"
           android:pivotX="50%"
           android:pivotY="50%"
           android:startOffset="700"
           android:duration="400"
           android:fillBefore="false" />
        <rotate
           android:fromDegrees="0"
           android:toDegrees="-45"
           android:toYScale="0.0"
           android:pivotX="50%"
           android:pivotY="50%"
           android:startOffset="700"
           android:duration="400" />
    </set>
</set>
```

注意对pivotX、pivotY的值，当为50时表明对于其parent的50%，为50%时表面枢轴是对于自己的50%。

另外，默认所有动作都是同事进行的，如果希望分开进行，需要设置startOffset

**常见**使用方法为：

```java
ImageView spaceshipImage = (ImageView) 
findViewById(R.id.spaceshipImage);

Animation hyperspaceJumpAnimation = 
    AnimationUtils.loadAnimation(this, 
    R.anim.hyperspace_jump);

spaceshipImage.startAnimation(hyperspaceJumpAnimation);
```

除了`startAnimation`外，还有`Animation.setStartTime()`,之后再使用`View.setAnimation()`,以及在一些地方用于`setAnimationStyle`


