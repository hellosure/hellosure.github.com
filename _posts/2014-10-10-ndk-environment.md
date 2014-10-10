---
layout: post
title: Android NDK环境搭建
category: Android
tags: [Android,NDK]
---

### NDK下载地址

http://dl.google.com/android/ndk/android-ndk32-r10-windows-x86.zip

http://dl.google.com/android/ndk/android-ndk32-r10-windows-x86_64.zip

http://dl.google.com/android/ndk/android-ndk32-r10-darwin-x86.tar.bz2

http://dl.google.com/android/ndk/android-ndk32-r10-darwin-x86_64.tar.bz2

http://dl.google.com/android/ndk/android-ndk32-r10-linux-x86.tar.bz2

http://dl.google.com/android/ndk/android-ndk32-r10-linux-x86_64.tar.bz2

http://dl.google.com/android/ndk/android-ndk64-r10-windows-x86.zip

http://dl.google.com/android/ndk/android-ndk64-r10-windows-x86_64.zip

http://dl.google.com/android/ndk/android-ndk64-r10-darwin-x86.tar.bz2

http://dl.google.com/android/ndk/android-ndk64-r10-darwin-x86_64.tar.bz2

http://dl.google.com/android/ndk/android-ndk64-r10-linux-x86.tar.bz2

http://dl.google.com/android/ndk/android-ndk64-r10-linux-x86_64.tar.bz2

http://dl.google.com/android/ndk/android-ndk-r10-cxx-stl-libs-with-debug-info.zip

http://dl.google.com/android/ndk/android-ndk-r9d-windows-x86.zip

http://dl.google.com/android/ndk/android-ndk-r9d-windows-x86_64.zip

http://dl.google.com/android/ndk/android-ndk-r9d-darwin-x86.tar.bz2

http://dl.google.com/android/ndk/android-ndk-r9d-linux-x86.tar.bz2

http://dl.google.com/android/ndk/android-ndk-r9d-linux-x86_64.tar.bz2

http://dl.google.com/android/ndk/android-ndk-r9d-cxx-stl-libs-with-debug-info.zip

### Windows

#### 下载安装cygwin

由于NDK编译代码时必须要用到make和gcc，所以你必须先搭建一个linux环境，cygwin是一个在windows平台上运行的unix模拟环境,它对于学习unix/linux操作环境，或者从unix到windows的应用程序移植，非常有用。通过它，你就可以在不安装linux的情况下使用NDK来编译C、C++代码了。下面我们一步一步的安装cygwin吧。 

首先，你得先跑到<http://www.cygwin.com/>下载setup.exe

1、 然后双击运行吧，运行后你将看到安装向导界面：

2、 点击下一步，此时让你选择安装方式：

 1）Install from Internet：直接从Internet上下载并立即安装（安装完成后，下载好的安装文件并不会被删除，而是仍然被保留，以便下次再安装）。

 2）Download Without Installing：只是将安装文件下载到本地，但暂时不安装。

 3）Install from Local Directory：不下载安装文件，直接从本地某个含有安装文件的目录进行安装。

3、选择第一项，然后点击下一步：

4、选择要安装的目录，注意，最好不要放到有中文和空格的目录里，似乎会造成安装出问题，其它选项不用变，之后点下一步：

5、上一步是选择安装cygwin的目录，这个是选择你下载的安装包所在的目录，默认是你运行setup.exe的目录，直接点下一步就可以：

6、此时你共有三种连接方式选择：

 1) Direct Connection：直接连接。

 2) Use IE5 Settings：使用IE的连接参数设置进行连接。

 3) Use HTTP/FTP Proxy：使用HTTP或FTP代理服务器进行连接（需要输入服务器地址、端口号）。

根据自己的网络连接的实情情况进行选择，一般正常情况下，均选择第一种，也就是直接连接方式。然后再点击“下一步”，

7、 这是选择要下载的站点，我用的是<http://mirrors.kernel.org/>，速度感觉还挺快，选择后点下一步.(有的网友说选择台湾的站点也比较快)

8、 此时会下载加载安装包列表

9、Search是可以输入你要下载的包的名称，能够快速筛选出你要下载的包。那四个单选按钮是选择下边树的样式，默认就行，不用动。View默认是Category，建议改成full显示全部包再查，省的一些包被隐藏掉。左下角那个复选框是是否隐藏过期包，默认打钩，不用管它就行，下边开始下载我们要安装的包吧，为了避免全部下载，这里列出了后面开发NDK用得着的包：autoconf2.1、automake1.10、binutils、gcc-core、gcc-g++、gcc4-core、gcc4-g++、gdb、pcre、pcre-devel、gawk、make共12个包

10、然后开始选择安装这些包吧，点skip，把它变成数字版本格式，要确保Bin项变成叉号，而Src项是源码，这个就没必要选了。

11、下面测试一下cygwin是不是已经安装好了。

运行cygwin，在弹出的命令行窗口输入：`cygcheck -c cygwin`命令，会打印出当前cygwin的版本和运行状态，如果status是ok的话，则cygwin运行正常。


然后依次输入`gcc –-version`，`g++ --version`，`make –-version`，`gdb -–version`进行测试，如果都打印出版本信息和一些描述信息，非常高兴的告诉你，你的cygwin安装完成了！
         
#### 配置NDK环境变量


1、首先找到cygwin的安装目录，找到一个`home\<你的用户名>\.bash_profile`文件，我的是：`C:\cygwin64\home\sure\.bash_profile`，

2、打开`.bash_profile`文件，添加`NDK=/cygdrive/<你的盘符>/<android ndk 目录>` 例如：

    NDK=/cygdrive/c/android-ndk-r10b
    export NDK


(NDK这个名字是随便取的，为了方面以后使用方便，选个简短的名字，然后保存)

3、打开cygwin，输入cd $NDK，如果输出上面配置的`/cygdrive/c/android-ndk-r10b`信息，则表明环境变量设置成功了。

#### 用NDK来编译程序


1、现在我们用安装好的NDK来编译一个简单的程序吧，我们选择ndk自带的例子hello-jni，我的位于`C:\android-ndk-r10b\samples\hello-jni`.


2、运行cygwin，输入命令`cd /cygdrive/e/android-ndk-r10b/samples/hello-jni`，进入到`C:\android-ndk-r10b\samples\hello-jni`目录。


3、 输入`$NDK/ndk-build`，执行成功后，它会自动生成一个libs目录，把编译生成的.so文件放在里面。($NDK是调用我们之前配置好的环境变量，ndk-build是调用ndk的编译程序)


(早期NDK版本是make APP=hello-jni ,还要对应app和source2个目录的项目目录，现在改成了$NDK/ndk-build）


4、此时去hello-jni的libs目录下看有没有生成的`.so`文件，如果有，你的ndk就运行正常啦！

