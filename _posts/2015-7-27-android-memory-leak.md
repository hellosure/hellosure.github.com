---
layout: post
title: 如何查找Android内存泄露
category: Android
tags: [Android,内存泄露]
---

### Memory Monitor

eclipse中可以使用DDMS的heap视图来看data object的total size项。

android studio中直接观察memory monitor。（Memory Monitor是android studio自带的，不需要额外安装，这个工具显然更直观一些）

    个人觉得这种方式可以作为查找内存泄露的突破口，如果发现一些异常，比如抖动大说明GC频繁，或比如GC对内存释放不明显，或比如GC之后内存并没有稳定在一定的范围之内，则可能是有内存泄露问题，可以接下去分析。


### Heap and Allocation Tracker

用android studio的Heap and Allocation Tracker工具（是android studio自带，不需要额外安装）来查看一段操作时间内各种调用对内存的影响。

    个人觉得这个工具很方便用来查看各操作allocation的情况，如果某调用产生的allocation大小或数量很大，可能就是需要引起关注的地方。


### MAT分析Leak Suspects

通过DDMS导出hprof，然后用MAT来生成报告，报告里会提供Leak Suspects和Dominator Tree。

    个人觉得这个方法，是作为前面两个方法的补充，也是功能最强大的。如果说前两个方法是用来判断可能有内存泄露，那这个方法就是用来确定到底什么地方引发的内存泄露。

PS1：MAT下载地址<http://www.eclipse.org/mat/downloads.php>

PS2：导出的hprof需要用hprof-conv工具转化一下才能导入MAT分析。

-EOF-
