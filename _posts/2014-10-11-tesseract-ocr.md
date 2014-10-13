---
layout: post
title: Tesseract OCR初探
category: OCR
tags: [OCR,Tesseract]
---

### OPENCV & OCR

+ **OpenCV**（Open Source Computer Vision Library，跨平台计算机视觉库），专注机器视觉，是个更大范围的概念

+ **OCR** （Optical Character Recognition，光学字符识别），专注于字符识别

### OCR工具

+ 收费

> **ABBYY Cloud OCR SDK**确实很强大，但是试用版的有很多限制。它是现有对中文识别最靠谱的，但是收费。

+ 开源

> 开源的OCR工具还比较多，最流行也是Google支持的是**Tesseract**

### Tesseract简介

tesseact其实全称是tesseract-ocr，是个自动识别字符的程序，项目网址是：<http://code.google.com/p/tesseract-ocr/>。虽然其主流平台是三大系统（Win/Linux/Mac OS），但在android和iphone上也是可以跑的 - 这点对我来讲非常重要。
你可以直接想在其命令行工具使用，或者下载其SDK开发自己的程序。

tesseract支持多种语言 - 你只需下载对应的训练过的语言文件即可，并且可以通过config文件来调整行为：比如只识别数字，比如只识别指定的words或者指定的pattern。另外提一下，tesseract只支持字符识别，不支持条形码(barcode)识别。

### 改善tesseract识别正确率的方法

    (1)please check DPI of your image and size of text
    
    (2)try to set different segmentation mode (-psm option for command line) if you try to OCR small part of text (line, text)
    
    (3)try to add border (see issue 398)
    
    (4)try to pre-process image (increase DPI, resize, blur/sharpen image) before OCR (see issue 191)
    
    (5)try to remove noise dewarp (so there are straight text lines) image and binarize image

从上面第三点可以看出，tesseract不会对图片进行归一化，大尺寸的图片更容易识别。

#### 设置识别白名单

还有一个很重要的方法：设置识别白名单，如只识别数字，或大写字母，可以大大提高识别率。

将`tessedit_char_whitelist 0123456789` 放在`config/digits`中，数字可被替换。
测试200多个单个字符（200张图片），识别率达到90%，字符为黑体印刷体。
目前测试中增加字体宽度，对识别率，无明显影响。

### windows中命令行使用tesseract

1. 下载安装Tesseract-OCR引擎(3.0版本+才支持中文识别)

`tesseract-ocr-setup-3.01-1.exe`

下载完后进行安装,默认情况下安装程序会给你配置系统环境变量,以指向安装目录（之后可以通过DOS界面在任意目录运行tesseract）。

其安装目录中的 tessdata 目录存放的是语言字库文件，和在命令行界面中可能用到的参数所对应的文件.  这个安装程序默认包含了英文字库。

如果想能识别中文，可以到<http://code.google.com/p/tesseract-ocr/downloads/list>下载对应的语言的字库文件. 

简体中文字库文件下载地址为:<http://tesseract-ocr.googlecode.com/files/chi_sim.traineddata.gz> 下载完成后解压，然后将该文件剪切到tessdata目录下去就可以了。

2. 使用Tessract-OCR引擎识别验证码

打开DOS界面，输入tesseract。

    `tesseract imagename outputbase [-l lang] [-psm pagesegmode] [configfile...]`
    tesseract    图片名  输出文件名 -l 字库文件 -psm pagesegmode 配置文件

例如：

> `tesseract code.jpg result  -l chi_sim -psm 7 nobatch`
> 
> `-l chi_sim` 表示用简体中文字库（需要下载中文字库文件，解压后，存放到tessdata目录下去,字库文件扩展名为  .raineddata 简体中文字库文件名为:  chi_sim.traineddata）
> 
> `-psm 7` 表示告诉tesseract code.jpg图片是一行文本，这个参数可以减少识别错误率.  默认为 3
> 
> `configfile` 参数值为`tessdata\configs` 和  `tessdata\tessconfigs` 目录下的文件名。

我准备了一张验证码code.jpg放在桌面，然后cmd到desktop，然后输入`tesseract code.jpg result`，这样可以在`result.txt`中看到结果。

#### 实际测试

试了一下现在想做的实际的例子，记录一下：

（1）如果直接把整个图片进行识别，很乱，不OK。

（2）把字符码截图出来，把“IP-F2MPCC75”识别为了“HPFZMPCC75”。

（3）把号码截图出来，把“64500366”正确识别出来。

所以现在的问题有两个：

（1）不能用一张图片来搞定，要拍两张图，而且要对着拍照，这样要求太苛刻了。
不过由于号码是固定的，可以预存，所以只需要对着字符码拍照即可。所以这个问题应该不存在了。

（2）字符码的识别有些问题，比如“Z”和“2”。
这个可能就需要训练了。

（3）这个例子中还不存在这个问题，因为字符都是规则的，但是有些图片里字符是歪的或者不是标准字体，很可能是识别不正确的。
这种情况也需要进行训练。

**也就是说：要想提高识别率，除了设置白名单、提升图片精确度这两种做法之外，还有训练这种做法。**

我自己的理解，提升识别正确度：

+ 设置白名单

+ 提升图片质量

+ 训练

### tesseract训练

tesseract是自带训练工具的。

关于如何训练样本，Tesseract-OCR官网有详细的介绍<http://code.google.com/p/tesseract-ocr/wiki/TrainingTesseract3>。

网上也有很多训练的例子，比如<http://my.oschina.net/lixinspace/blog/60124>、<http://blog.wudilabs.org/entry/f25efc5f/>这里就不赘述了。

### tesseract对IOS的支持

tesseract-ocr是开源的，但是用在IOS上可能有点曲折，在网上有解决方法。
另外github中有demo：

<https://github.com/nolanbrown/Tesseract-iPhone-Demo>

<https://github.com/ldiqual/tesseract-ios>

<https://github.com/gali8/Tesseract-OCR-iOS>

### tesseract对Android的支持

github中有demo，<https://github.com/rmtheis/android-ocr>，它还需要<https://github.com/rmtheis/tess-two>的支持。

tess-two是Tesseract Tools for Android (tesseract-android-tools) 的一份拷贝，并添加了一些功能。Tesseract Tools for Android是Tesseract OCR和Leptonica图像处理库的Android API与构建文件的集合。

共三个项目，tess-two, tess-two-test 以及eyes-two。其中tess-two和eyes-two为android lib项目，供其它项目引用。tess-two封装Tesseract的Android API，eyes-two封装leptonica的Android API。tess-two-test为OCR的测试。

#### 编译

关于tess-two的编译过程，可以参考github上的描述，但是我本地发现build不成功，报错为：

    Android NDK: ERROR:C:/android-ndk-r10b/sources/cxx-stl/gnu-libstdc++/Android.mk:gnustl_static: LOCAL_SRC_FILES points to a missing file    
    Android NDK: Check that C:/android-ndk-r10b/sources/cxx-stl/gnu-libstdc++/4.8/libs/armeabi/thumb/libgnustl_static.a exists  or that its path is correct   
    C:/android-ndk-r10b/build/core/prebuilt-library.mk:45: *** Android NDK: Aborting    .  Stop.

但是查看路径之后发现，我安装的ndk10的版本不是4.8，而是4.9。
但是ndk9的版本是4.8，所以我想还是安装ndk9好了。

1. 把ndk9的下载压缩包解压，放在c盘根目录下。在环境变量Path中添加`C:\android-ndk-r9d`。在.bash_profile中也写进去。

2. 在cmd中进入到`C:\android-ndk-r9d\samples\hello-jni`，执行`ndk-build`，然后等待片刻出现`libs`文件夹，其中有`.so`文件，这就说明build成功了。

然后就可以开始对tess-two进行build：

1. 用cmd到tess-two目录中执行`ndk-build`，这是因为已经把NDK路径添加到path路径中了，所以可以直接找到这个命令。这里需要两个小时。这步的结果是在tess-two路径中添加了`libs`和`obj`目录，里面是.so、.o、.o.d文件。

2. 把`C:\Users\sure\Desktop\software\adt-bundle-windows-x86_64-20140624\sdk\tools`加入到环境变量`Path`中，这样就可以使用`android`命令。在cmd中执行`android update project --path .`。这步的结果是更新了`local.properties`文件，添加了'proguard-project.txt'文件，看了下文件内容其实就是指明了本地sdk地址。

3. 执行`ant release`。这步的结果是在tess-two中添加了`bin`和`gen`目录，做的事情是将java文件编译打包了。

#### 导入

1. 将tess-two导入到eclipse。 File -> Import -> Existing Projects into workspace -> tess-two directory.

2. 右击该工程 Android Tools -> Fix Project Properties.

3. 右击该工程 Properties -> Android ，在project build target中，选择一个较新的android版本，并在Is Library前点上勾，点击OK。

这个地方我遇到了很多问题，理论上直接Fix Project Properties可以消除的一些错误，我却没能完成。后来我在右击该工程->build path->configure build path 的soure中添加了res和gen文件夹，在library中add library加入了android class path container和JRE6。然后又右击该工程 Properties -> Android -> Project Build Target选择15。这样tess-two工程才没有错误了。

#### 下载文字库

在手机中SD卡添加`/mnt/sdcard/tesseract/tessdata`路径，并且传入`C:\Program Files (x86)\Tesseract-OCR\tessdata`路径下的`eng.traineddata`文件。

#### 测试一

用github上的android-ocr<https://github.com/rmtheis/android-ocr>。

导入之后，工程名称自动为OCRTest。选择Project Build Target为15（选这个是因为我的测试手机是这个版本）。然后右击该工程 -> Properties -> Android -> Library -> Add, 选择tess-two。

在手机上开始Run了之后，首先是"Downloading data for English..."的提示框。然后是"Uncompressing data for English..."。然后是"Downloading data for orientation and script detection..."。

可以用了，是个拍照框，点击拍照之后，就可以识别出文字。
对英文的识别还比较不错，不过这个例子中还想翻译，这个功能我的应用是用不到的。

> 在应用的场景上比较类似，拍照识别，另外识别正确度还可以，可以参考。

#### 测试二

用的<http://blog.csdn.net/wp8868191/article/details/9219399>中的例子。

自己尝试做了拍照识别和从相册选择图片识别。

但是手机运算能力太差，图片太大、分辨率太高的话，识别时间会很长，所以在选取图片的时候调用了系统裁剪功能，并且另开线程来处理识别。

推荐测试的时候不要用太大的图片。

用java写了图片的预处理，所以拿过来试试能否提高识别成功率：

无奈安卓无法使用java.awt里面的包，所以还费了一些时间替换成android.graphics中的一些类实现相同功能。

测试发现灰度化后是能提高一些识别率，在电脑上灰度化后再用三个算法二值化后还能进一步提高识别率。

但在手机上用大津法、最大熵法进行二值化花费时间太久（几乎没算成功），所以后来这两个方法就没有调用，而只用迭代法二值化效果不理想。

最后的效果是，能识别一些比较规整的文字，照片的话最好只裁剪文字部分去识别（而且要照的比较清晰）。

也能识别一些简单的英文、数字验证码。

> 这个应用的界面不是很推荐，操作太复杂，不过图片预处理的部分还是可以看看的。

#### 测试三

用的是<http://www.cnblogs.com/muyun/archive/2012/06/12/2546693.html>的例子。

> 这个例子很简单，不带拍照功能。另外试了一下识别率很低。所以不做考虑了。

#### 测试四

来自于<http://gaut.am/making-an-ocr-android-app-using-tesseract/>

首先用<http://labs.makemachine.net/2010/03/simple-android-photo-capture/>是个简单的项目，用来拍照得到bitmap位图文件。

然后对位图文件做个处理：

{% highlight java %}

// _path = path to the image to be OCRed
ExifInterface exif = new ExifInterface(_path);
int exifOrientation = exif.getAttributeInt(
        ExifInterface.TAG_ORIENTATION,
        ExifInterface.ORIENTATION_NORMAL);

int rotate = 0;

switch (exifOrientation) {
case ExifInterface.ORIENTATION_ROTATE_90:
    rotate = 90;
    break;
case ExifInterface.ORIENTATION_ROTATE_180:
    rotate = 180;
    break;
case ExifInterface.ORIENTATION_ROTATE_270:
    rotate = 270;
    break;
}

if (rotate != 0) {
    int w = bitmap.getWidth();
    int h = bitmap.getHeight();

    // Setting pre rotate
    Matrix mtx = new Matrix();
    mtx.preRotate(rotate);

    // Rotating Bitmap & convert to ARGB_8888, required by tess
    bitmap = Bitmap.createBitmap(bitmap, 0, 0, w, h, mtx, false);
}
bitmap = bitmap.copy(Bitmap.Config.ARGB_8888, true);

{% endhighlight %}

然后就可以使用TessBaseAPI了：

{% highlight java %}

TessBaseAPI baseApi = new TessBaseAPI();
// DATA_PATH = Path to the storage
// lang = for which the language data exists, usually "eng"
baseApi.init(DATA_PATH, lang);
// Eg. baseApi.init("/mnt/sdcard/tesseract/tessdata/eng.traineddata", "eng");
baseApi.setImage(bitmap);
String recognizedText = baseApi.getUTF8Text();
baseApi.end();

{% endhighlight %}

这样识别出的文字就在`recognizedText`中了。

> 这个例子的代码在[https://github.com/GautamGupta/Simple-Android-OCR](https://github.com/GautamGupta/Simple-Android-OCR)，可以参考一下。

![orc](http://gaut.am/wp-content/uploads/2011/11/capture_3-1024x768.jpg "ocr")

#### TessBaseAPI

补充一下，使用TessBaseAPI必要的代码

{% highlight java %}

//新建一个TessBaseAPI
TessBaseAPI baseApi=new TessBaseAPI();

//初始化API
//android下面，tessdata肯定得放到sd卡里了，如果直接将tessdata文件夹放在SD卡得根目录下，我们这可以这样写初始化
File path = Environment.getExternalStorageDirectory();//获取SD卡根目录
baseApi.init(path.getAbsolutePath(),"eng");//英文是eng，简体中文是chi_sim，目测应该就是tessdata文件夹中.tessdata文件的文件名

//设置要ocr的图片bitmap，这个我是采用摄像头获得的图片位图，大家也可以从文件获得，只要得到bitmap就行
baseApi.setImage(bitmap);

//根据Init的语言，获得ocr后的字符串
String text= baseApi.getUTF8Text();

//释放bitmap
baseApi.clear();
//如果连续ocr多张图片，这个end可以不调用，但每次ocr之后，必须调用clear来对bitmap进行释放
//释放native内存
baseApi.end();

{% endhighlight %}


-EOF-
