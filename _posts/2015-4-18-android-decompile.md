---
layout: post
title: Android反编译APK
category: Android
tags: [Android,Decompile,APK]
---

1. apktool  

作用：资源文件获取，可以提取出图片文件和布局文件进行使用查看

    cd /Users/hellosure/Documents/Decompile\ Android/apktool1.5.2
    ./apktool d /Users/hellosure/Desktop/fancy/app-release.apk

生成app-release文件夹，里面有assets和res，源代码是smali看不了。
这个工具主要是方便查看资源文件。

2. dex2jar

作用：将apk反编译成java源码（classes.dex转化成jar文件）

    cd /Users/hellosure/Documents/Decompile\ Android/dex2jar-0.0.9.15 
    ./d2j-dex2jar.sh /Users/hellosure/Desktop/fancy/classes.dex

其中classes.dex文件是将apk解压之后得到的。这个工具会生成classes_dex2jar.jar

3. jd-gui

作用：查看APK中classes.dex转化成出的jar文件，即源码文件

将上面一步生成的classes_dex2jar.jar打开，就可以看到java源代码。

-EOF-
