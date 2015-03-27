---
layout: post
title: 移动设备唯一标示
category: Android
tags: [Android,IOS]
---

### android设备

<http://blog.csdn.net/aminfo/article/details/7604451>

1. DEVICE_ID
假设我们确实需要用到真实设备的标识，可能就需要用到DEVICE_ID。通过 TelephonyManager.getDeviceId()获取，它根据不同的手机设备返回IMEI，MEID或者ESN码.
缺点：在少数的一些设备上，该实现有漏洞，会返回垃圾数据

2. MAC ADDRESS
我们也可以通过Wifi获取MAC ADDRESS作为DEVICE ID
缺点：如果Wifi关闭的时候，硬件设备可能无法返回MAC ADDRESS.。

3. Serial Number
android.os.Build.SERIAL直接读取
缺点：在少数的一些设备上，会返回垃圾数据

4. ANDROID_ID
ANDROID_ID是设备第一次启动时产生和存储的64bit的一个数，
缺点：当设备被wipe后该数改变, 不适用

### iOS设备

不能用mac，只能用idfa。将IDFA作为iOS应用内的优先广告追踪选项
<http://www.cocoachina.com/industry/20130422/6040.html>
<http://www.csdn.net/article/2012-09-24/2810288>

-EOF-
