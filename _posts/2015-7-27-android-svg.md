---
layout: post
title: 简单聊下Android的SVG
category: Android
tags: [Android,SVG]
---

### WHAT？

#### 什么是SVG？

SVG - Scalable Vector Graphics

<https://en.wikipedia.org/wiki/Scalable_Vector_Graphics>

#### SVG Path Format规范

<http://www.w3.org/TR/SVG11/paths.html#PathData>


### HOW?

#### api21之后如何实现？

可以直接使用SDK中的VectorDrawable、AnimatedVectorDrawable。

实用工具trans svg file to VectorDrawable xml: <http://inloop.github.io/svg2android/>

#### api21之前如何实现？

（1）v7添加了VectorDrawableCompat

<https://android.googlesource.com/platform/frameworks/support/+/master/v7/vectordrawable/src/android/support/v7/graphics/drawable>

说真的，这个类是google临时工写的吧

（2）https://github.com/telly/MrVector

但是它没有实现AnimatedVectorDrawable。这个库曾经比较流行，但是github首页已经写明停止更新，推荐（1）

（3）https://github.com/pents90/svg-android

这个库曾经也比较流行，有些fork它的其他草根lib，但是功能也很弱。

（4）https://github.com/wnafee/vector-compat

这个库也有人推荐。

（5）参考<http://stackoverflow.com/questions/26548354/vectordrawable-is-it-available-somehow-to-pre-lollipop-versions-of-android>



### WHY?

#### 考虑使用svg的原因：

相对点9来说，是更完美的resize解决方案。

#### 最终没使用svg的原因：

android的svg暂时只适用于简单的图案，对shadow、纹理等支持很差。

-EOF-
