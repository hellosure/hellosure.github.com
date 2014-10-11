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

### tesseract对IOS的支持

tesseract-ocr是开源的，但是用在IOS上可能有点曲折，在网上有解决方法。
另外github中有demo，<https://github.com/nolanbrown/Tesseract-iPhone-Demo>

### tesseract对Android的支持

github中有demo，<https://github.com/rmtheis/android-ocr>，它还需要<https://github.com/rmtheis/tess-two>的支持。

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

### tesseract训练

tesseract是自带训练工具的。

### tesseract SDK



-EOF-
