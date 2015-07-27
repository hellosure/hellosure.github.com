---
layout: post
title: 若干Android代码风格Tips
category: Android
tags: [Android,代码风格]
---

### 防止Null Pointer Error

1. 不要对sdk（包括android/第三方库）的调用返回值做任何理想化的假设，比如cursor要判空。

2. 不要对自己写的方法入参做任何理想假设，比如很多用getActivity()传入context，都容易出错。自己写的方法，可以一开始就把入参都拦截check一下，判断是否合法，再开始做实际操作。


### 布局文件与资源

1. layout xml中的高低版本兼容问题：

（1）对于高低版本有不同attribute来实现相同功能的情况，比如right/end：都写一遍。

（2）对于某种attribute只在高版本中能实现：如果不影响低版本布局，可以不分xml，在代码中实现功能。如果影响低版本布局，可以分版本写xml。

2. png和9.png文件不能重名

3. layout xml文件中定义各view id规范：比如emoji_view.xml中一个用于delete的Button，可以将id命名为btn_emoji_delete

4. 为了减少apk打包大小，对于resource中的icon，都放置最大分辨率图片到drawable-xxxhdpi中。但是由于assets中图片资源比较大，为了降低不同分辨率手机运行时内存，所以分dpi放置在assets中。

5. 避免inflate方法中将root view赋值为null。

6. 对于sharedPreferences的put操作：commit()是同步方式，apply()是异步方式，要按不同使用场景选取合适的。

7. 避免layout xml中嵌套太多，比如RelativeLayout的子view都确定height来撑起其height，而不要用一个多余的LinearLayout来撑起其height。

-EOF-
