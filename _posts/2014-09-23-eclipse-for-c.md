---
layout: post
title: 使用Eclipse搭建C++开发环境
category: Eclipse
tags: [Eclipse,C++]
---

Eclipse for C++和MinGW安装配置C/C++开发环境

### [Eclipse IDE for C/C++ Developers](http://www.eclipse.org/downloads/)

### [MinGW](http://sourceforge.net/projects/mingw/files/)

#### 安装

在上面的mingw页面下载mingw-get-inst-xxxxxx.exe

#### 更新

在MinGW Installation Manager中勾选要更新的package，然后Update  Catalogue

#### 配置

把mingw的bin目录加进环境变量的Path。

在eclipse for C++中，依次点击打开“Window>preferences>C/C++>New CDTProject Wizard，在右侧，选择Preferred Toolchains，按下图中步骤设置Executable（可执行程序）的默认编译器为MinGWGCC，这样在新建工程的时候就不需要重复选择编译器了，其他工程类型的设置过程类同。 

![mingw](http://img.blog.csdn.net/20140527091448171?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbHlzMDc5NjIwMDA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center "mingw")

点击右侧NewC/C++ Project Wizard >(下方NewC/C++Project Wizard)Makefile Project,在左侧，选择“MakefileProject”，在右侧，选择Binary Parsers标签（默认已选中），勾选“PEWindows Parser”，点击OK保存。

![wizard](http://img.blog.csdn.net/20140527091548781?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbHlzMDc5NjIwMDA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center "wizard")

#### 创建C++工程

依次点击 File>NewProject>C++ Project，输入工程名hellocpp，Project type选择Executable\HelloWorld C++ Project，Toolchains选择MinGW GCC（默认选中），点击"Finish"完成C++工程的创建

![cplusproj](http://img.blog.csdn.net/20140527091631187?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbHlzMDc5NjIwMDA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center "cplusproj")

在ProjectExplorer中右键工程hellocpp，依次选择 MakeTargets>Create...，输入Target(目标文件名），例如：hello，点击"OK"，完成Target的创建；

在ProjectExplorer中右键工程hellocpp，依次选择 MakeTargets>Build...，选择刚才创建的Target，点击"Build"，完成Target的构建，这时我们从ProjectExplorer中发现生成了hellocpp.exe，运行即可。

#### 创建C工程

点击NewProject>C Project，其他的差不多。

-EOF-
