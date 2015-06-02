---
layout: post
title: Android消息传递
category: Android
tags: [Android,onTouch,onTouchEvent]
---

### 一般情况

父子节点传递Event

<http://www.cnblogs.com/sunzn/archive/2013/05/10/3064129.html>

一图胜千言
<http://blog.csdn.net/chenzhiqin20/article/details/8816364>

<http://www.infoq.com/cn/articles/android-event-delivery-mechanism>

### onTouch 和 onTouchEvent 的区别

1.onTouch方法：

onTouch方法是View的 OnTouchListener接口中定义的方法。

当一个View绑定了OnTouchLister后，当有touch事件触发时，就会调用onTouch方法。（当把手放到View上后，onTouch方法被一遍一遍地被调用），这个方法有返回值 false 和true

2.onTouchEvent方法：

onTouchEvent方法是override 的Activity的方法。

覆写了Activity的onTouchEvent方法后，当屏幕有touch事件时，此方法就会别调用。

（当把手放到Activity上时，onTouchEvent方法就会一遍一遍地被调用）

3.touch事件的传递：

在一个Activity里面放一个TextView的实例tv，绑定onTouchClickListener监听器，并且这个tv的属性设定为 fill_parent，占满整个屏幕。

在这种情况下，当手放到屏幕上的时候，首先会是tv响应touch事件，执行onTouch方法。

(1) 如果onTouch返回值为true，
表示这个touch事件被onTouch方法处理完毕，不会把touch事件再传递给Activity，也就是说onTouchEvent方法不会被调用。

（当把手放到屏幕上后，onTouch方法被一遍一遍地被调用）

(2) 如果onTouch的返回值是false，
表示这个touch事件没有被tv完全处理，onTouch返回以后，touch事件被传递给Activity，onTouchEvent方法被调用。

（当把手放到屏幕上后，onTouch方法调用一次后，onTouchEvent方法就会一遍一遍地被调用）

-EOF-
