---
layout: post
title: Android Tips Round-Up
category: Android
tags: [Android]
---

直接贴链接了，绝对的诚意之作!

<http://blog.danlew.net/2014/03/30/android-tips-round-up-part-1/>

<http://blog.danlew.net/2014/04/14/android-tips-round-up-part-2/>

<http://blog.danlew.net/2014/04/28/android-tips-round-up-part-3/>

<http://blog.danlew.net/2014/05/12/android-tips-round-up-part-4/>

<http://blog.danlew.net/2014/05/28/android-tips-round-up-part-5/>

--------

<http://www.jianshu.com/p/d6611c8bd45c>

+ 尽量阅读官方文档，这才是原汁原味、不失真的开发指导；

+ 即使你认为设计程序是浪费时间，你只是喜欢写程序，至少你也得用思维导图理清思路，思维导图对于帮助你理解设计文档、理清思路有很大的帮助；

+ 不要用Intent传递大量的数据，这有可能导致ANR或者报异常；

+ 在退出页面后，系统不一定会及时执行onDestory方法，如果你在onDestory方法里做关闭文件、释放内存的操作可能出现退出程序又立即进入时，由于需要重新初始化这些信息导致代码重入的异常；

+ 在改动JNI后，运行程序之前记得卸载掉已经安装在模拟器或者真机上的该程序，如果直接运行，android不会load最新编译的so，也就不能立即看到修改后的效果；

+ 代码至少每天备份一次，或者是完善一个功能就备份一次，不要堆积之后一次性备份，因为在你的代码出问题需要回溯代码时你需要从服务器上重新取代码，同时也可以避免代码不是最新导致最后和其他人合并时不知道改了哪些地方；

+ 将打印信息封装成一个方法，用一个标志位控制这个这个方法的方法体是否需要执行，这样在由debug版释放到release版本时，不需要傻傻地一行一行地去掉代码，你只需要改变标志位的值就可以了；

+ 对于有返回值的JNI函数，即使你不返回任何值，用NDK编译JNI的时候也不会报错，所以在写JNI代码的时候，一定要仔细检查代码；

+ JNI频繁读写文件操作会影响程序的运行性能，可以考虑一次性在内存中申请一块大内存作为缓存空间，用这种空间换时间的方式可以大大提高程序的运行效率；

+ 不要指望类的finalize方法去处理需要回收和销毁的工作，因为finalize是系统回调的方法，调用时机不可预见，切记；

+ 使用文件流、Cursor时，使用结束后记得一定要关闭，否则可能导致内存泄漏，严重的情况可能引发程序崩溃；

+ 优先使用Google搜索引擎(少用百度)，如果不能正常使用Google搜索引擎建议通过代理、VPN、修改hosts文件等方式搭建梯子。这里提供一个免费的谷歌搜索引擎

+ 对于不需要使用硬件加速的activity（没有动画效果、视频播放以及各种多媒体文件的操作都可以关掉硬件加速），在AndroidManifest.xml文件中通过“android:hardwareAccelerated="false"”关掉硬件加速可节省应用内存；

+ 对于需要横竖屏转换的应用，又不想在横竖屏切换的时候重新跑onCreate方法，可以在AndroidManifest.xml文件中对应的Activity标签下调用“android:configChanges="screenSize|orientation"”；

+ 为了减轻应用程序主进程的内存压力，对于耗内存比较多的界面（比如视频播放界面、flash播放界面等），可以在AndroidManifest.xml文件中对应的Activity标签下调用“android:process=".processname"”单开一个进程，但在退出这个界面的时候一定要在该界面的onDestory方法中调用System的kill方法来杀掉该进程；

+ 在res/values/arrays.xml文件中定义的单个数组的元素个数不宜过大，过大会导致加载数据时非常慢，有时候你需要使用数组资源时数据有可能还没加载完成；

+ 一个Activity中最耗费内存的是activity的背景（多数情况如此，特别是对于分辨率很大的机器，一个界面的背景算下来都需要好几兆内存），所以在程序界面较多时，可以考虑将图片转换成静态的drawable，然后多个activity共用这一张背景图；

+ 可以通过为application、activity自定义主题的方式来关掉多点触摸功能，只需要在自定义的主题下添加这两个标签：
  <item name="android:windowEnableSplitTouch">false</item>
  <item name="android:splitMotionEvents">false</item>

+ 很多游戏进入时，播放的片头动画多数是一个视频文件；

+ Android单个dex文件的方法数不能超过65536个

+ 使用模拟器genymotion代替android自带模拟器（它需要虚拟机vituralbox的支持，不过官网已经提供了一个集成虚拟机的安装包了，直接下载下来安装即可）

+ 给Application或者activity设置自定义主题时，最好不要设置为全透明，否则在activity按Home键回退到桌面的时候效果很渣；

+ 放在apk的assets或者raw目录下的数据文件最好做加密处理，在需要使用的时候才解密，这样可以避免在apk被他人破解时数据也被破解的问题；

+ 最好不要再activity的onCreate方法里面调用popupwindow的show方法，有可能由于activity没有完全初始化导致程序异常（android.view.WindowManager$BadTokenException: Unable to add window -- token null is not valid），如果非要在一进activity就显示popupwindow，建议用handler.post、View.postDelay来处理；

+ 对于自定义View，在构造方法里面是获取不到视图的宽高的（此时获取长宽都为0），需要在onMeasure方法中或者跑了onMeasure方法后才能够获取到视图的宽高，不过你可以通过在构造方法里面强制测量视图的宽高来实现在构造方法里获取视图的宽高信息，具体见MeasureSpec介绍及使用详解

+ 如果你觉得在安装Eclipse后还需要配置android开发环境很麻烦，你可以直接使用ADT Bundle，它是一个懒人套餐，下载下来就可以用了，可以在这里下载。

+ 有时间看看阿里技术嘉年华、InfoQ演讲与访谈、Google IO视频，可以学习到一些解决问题、做大项目的经验。

+ 当应用中动画比较多，并且动画都是通过图片来切换的时候，可以考虑借用Cocos的精灵表单思想，这样就可以避免图片命名的烦恼。

-EOF-
