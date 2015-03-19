---
layout: post
title: Android屏幕适配
category: Android
tags: [Android,设计]
---

### 基本概念

这篇资料很好：
<http://www.cnblogs.com/yaozhongxiao/archive/2014/07/14/3842908.html>

### 实际开发

Android有个自动匹配机制去选择对应的布局和图片资源

#### 界面布局方面

需要根据物理尺寸的大小准备多套布局

layout: 放一些通用布局xml文件，比如界面中顶部和底部的布局，不会随着屏幕大小变化，类似windos窗口的title bar
layout-small: 屏幕尺寸小于3英寸左右的布局
layout-normal: 屏幕尺寸小于4.5英寸左右
layout-large: 4英寸-7英寸之间
layout-xlarge: 7-10英寸之间

#### 图片资源方面

需要根据dpi(屏幕密度destiny)值准备多套图片资源:

drawable
drawalbe-ldpi
drawable-mdpi
drawable-hdpi
drawable-xhdpi

### dpi计算方法

    dpi = √（长度像素数² + 宽度像素数²） / 屏幕对角线英寸数

计算WVGA（800*480）分辨率，3.7英寸的密度DPI

1. 开根号(800平方+480平方)=933，我理解这个是对角线的像素，而3.7英寸其实也是对角线的长度。
2. 933/3.7=252
3. dpi的范围属于hdpi

### 设计图中的dp换算到不同dpi屏幕上的px

    px = dp * (dpi/160)

也就是说在dpi=160（标准dip）的屏幕上，1dp=1px。
在dpi=320的屏幕上，1dp=2px，因此看起来是一样大的，就是和设计图是长得一样的，只不过因为屏幕本身destiny大，因此看起来要清晰一些。

### dpi和dp

设计图给的是dp，而dpi是和实际各种设备屏幕相关的。因此在开发的时候不需要考虑dpi，只考虑dp。

用dp做基准而不是px的原因在于：
在不同dpi屏幕上，显示效果都是一样的。

### 在开发时获取dpi的代码

    DisplayMetrics metrics = new DisplayMetrics();
    getWindowManager().getDefaultDisplay().getMetrics(metrics);
    iDensity = (int)( metrics.density * 160 );

### sp和dp

除了文字之外的都用dp，文字用sp

### layout_weight

通过LinearLayout的weightSum控制一共分为多少格子，然后在子View中通过

    android:layout_width="0"
    android:layout_weight="1"

来设置它占据其中的一格。

－EOF-
