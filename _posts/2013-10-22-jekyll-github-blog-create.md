---
layout: page
title: GitHub上使用jekyll bootstrap搭建博客
category: ruby
tags: [jekyll,bootstrap,博客,GitHub]
---

GitHub是一个用于使用Git版本控制系统项目的共享虚拟主机服务。它由GitHub公司（曾称Logical Awesome）的开发者Chris Wanstrath、PJ Hyett和Tom Preston-Werner使用Ruby on Rails编写而成。GitHub设计了Pages功能，允许用户自定义项目首页。

Jekyll是一个静态站点生成器，它会根据Markdown (or Textile), Liquid, HTML & CSS生成静态网站。支持从其他博客系统WordPress、Drupal等进行迁移。

Jekyll-Bootstrap是基于Jekyll和前端CSS/HTML框架Bootstrap的博客系统。

### Jekyll

Jekyll安装：

*   Ruby：

    > 在 Windows 平台下安装 Ruby 可以使用 RubyInstaller。本次安装使用[rubyinstaller-1.9.3-p448.exe](http://dl.bintray.com/oneclick/rubyinstaller/rubyinstaller-1.9.3-p448.exe?direct)软件包；安装完成之后还要安装DEVELOPMENT KIT（基于MSYS/MinGW的工具使你能够构建许多原生的C/C++扩展为Ruby），对应的软件包为[DevKit-tdm-32-4.5.2-20111229-1559-sfx.exe](https://github.com/downloads/oneclick/rubyinstaller/DevKit-tdm-32-4.5.2-20111229-1559-sfx.exe)。安装步骤如下：

        cd D:\ruby\DevKit-tdm
        #解压DevKit-tdm-32-4.5.2-20111229-1559-sfx.exe的路径为D:\ruby\DevKit-tdm
        ruby dk.rb init
        #生成config.yml，这里会检查将要添加DevKit支持的Ruby列表，只支持通过RubyInstaller安装的Ruby
        #如果这里列出的Ruby与你的要求不符，可以手动修改
        ruby dk.rb review  
        #检查要添加DevKit支持的Ruby列表是否有误
        ruby dk.rb install

*   RubyGems（Ruby程序包管理器）：
    
    > 下载[rubygems-2.1.7.zip](http://production.cf.rubygems.org/rubygems/rubygems-2.1.7.zip)压缩包，解压到文件夹cd打开，    安装命令：ruby setup.rb 查看更多的信息和参数：ruby setup.rb --help

*   Jekyll：

        ~ $ gem install jekyll
        ~ $ jekyll new myblog
        ~ $ cd myblog
        ~/myblog $ jekyll server
        # => 浏览器打开 http://localhost:4000

### Jekyll-Bootstrap

Jekyll-Bootstrap搭建博客：

*   [GitHub](https://github.com)上面创建新的仓库，例如codingtiger.github.com

*   安装Jekyll-Bootstrap
        
        git clone https://github.com/plusjade/jekyll-bootstrap.git codingtiger.github.com
        #克隆jekyll-bootstrap代码到本地
        cd codingtiger.github.com
        git remote set-url origin git@github.com:codingtiger/codingtiger.github.com.git
        git push origin master

    安装的过程中可以通过本机安装的Windows Git客户端操作，也可以通过GitHub for Windows操作。

*   发布
    
    > 可以通过[GitHub for Windows](http://github-windows.s3.amazonaws.com/GitHubSetup.exe)进行发布操作。也可以使用以下Git命令发布。

        git add .
        git commit -m "注释内容"
        git push origin master

*   域名绑定

    > 上传成功之后，等10分钟左右，就可以使用http://codingtiger.github.io进行访问。如果不想使用这个域名，可以在工程根目录下面创建CNAME文件，写入绑定域名，例如www.wozhimeng.com。
    
本文内容叙述尚不够详尽，会持续更新。

-EOF-

