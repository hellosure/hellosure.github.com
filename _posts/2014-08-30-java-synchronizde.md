---
layout: post
title: Java多线程总结之由synchronized说开去
category: Java
tags: [Java,多线程]
---

迁移自[Java多线程总结之由synchronized说开去](http://hellosure.iteye.com/blog/1121157)

## synchronized与wait()/notify()：
##### synchronized：
> 最重要一条： 
**synchronized是针对对象的隐式锁使用的，注意是对象！ **

举个小例子，该例子没有任何业务含义，只是为了说明synchronized的基本用法：

{% highlight java %}
Class MyClass(){  
  	synchronized void myFunction(){  
    		//do something  
  	}  
}  
  
public static void main(){  
  	MyClass myClass = new MyClass();  
  	myClass.myFunction();  
}  
{% endhighlight %}
好了，就这么简单。 
myFunction()方法是个同步方法，隐式锁是谁的？答：是该方法所在类的对象。 
看看怎么使用的：myClass.myFunction();很清楚了吧，隐式锁是myClass的。 
说的再明白一点，**线程想要执行myClass.myFunction();就要先获得myClass的锁。**

下面总结一下： 
1、synchronized关键字的作用域有二种： 
1）是某个对象实例内，synchronized aMethod(){}可以防止多个线程同时访问这个对象的synchronized方法（如果一个对象有多个synchronized方法，只要一个线程访问了其中的一个synchronized方法，其它线程不能同时访问这个对象中任何一个synchronized方法）。这时，不同的对象实例的synchronized方法是不相干扰的。也就是说，其它线程照样可以同时访问相同类的另一个对象实例中的synchronized方法； 

2）是某个类的范围，synchronized static aStaticMethod{}防止多个线程同时访问这个类中的synchronized static 方法。它可以对类的所有对象实例起作用。（注：这个可以认为是对Class对象起作用） 

2、除了方法前用synchronized关键字，synchronized关键字还可以用于方法中的某个区块中，表示只对这个区块的资源实行互斥访问。用法是: synchronized(this){/*区块*/}，它的作用域是this，即是当前对象。当然这个括号里可以是任何对象，synchronized对方法和块的含义和用法并不本质不同； 

3、synchronized关键字是不能继承的，也就是说，基类的方法synchronized f(){} 在继承类中并不自动是synchronized f(){}，而是变成了f(){}。继承类需要你显式的指定它的某个方法为synchronized方法； 

synchronized可能造成死锁，比如：
{% highlight java %}
class DeadLockSample{    
    public final Object lock1 = new Object();    
    public final Object lock2 = new Object();    
    
    public void methodOne(){    
       synchronized(lock1){    
          ...    
          synchronized(lock2){...}    
       }    
    }    
    
    public void methodTwo(){    
       synchronized(lock2){    
      ...    
          synchronized(lock1){...}    
       }    
    }    
}  
{% endhighlight %}

##### RubyGems（Ruby程序包管理器）：
> 下载[rubygems-2.1.7.zip](http://production.cf.rubygems.org/rubygems/rubygems-2.1.7.zip)压缩包，解压到文件夹cd打开，    安装命令：ruby setup.rb 查看更多的信息和参数：ruby setup.rb --help
##### Jekyll：

        {% highlight bash %}
        ~ $ gem install jekyll
        ~ $ jekyll new myblog
        ~ $ cd myblog
        ~/myblog $ jekyll server
        # => 浏览器打开 http://localhost:4000{% endhighlight %}

## Jekyll-Bootstrap搭建博客：
##### 1. [GitHub](https://github.com)上面创建新的仓库，例如codingtiger.github.com。
##### 2. 安装Jekyll-Bootstrap

        {% highlight bash %}
        git clone https://github.com/plusjade/jekyll-bootstrap.git codingtiger.github.com
        #克隆jekyll-bootstrap代码到本地
        cd codingtiger.github.com
        git remote set-url origin git@github.com:codingtiger/codingtiger.github.com.git
        git push origin master{% endhighlight %}
> 安装的过程中可以通过本机安装的Windows Git客户端操作，也可以通过GitHub for Windows操作。

##### 2. 创建文章
> rake post会自动在_posts文件夹下面创建Markdown文件，编辑文件即可。Markdown文件可以使用Sublime Text 2的Markdown插件MarkdownEditing来编辑。

        {% highlight ruby %}
        rake post title="Hello World"
        Creating new post: ./_posts/2013-10-23-hello-world.md{% endhighlight %}

> 创建完成通过安装在本地的Jekyll就可以进行调试。**本地调试过程如果出现以下问题：error: invalid byte sequence in GBK.可以找到ruby安装目录下convertible.rb，修改**

        {% highlight ruby %}
        self.content = File.read(File.join(base, name)){% endhighlight %}

> 为

        {% highlight ruby %}
        self.content = File.read(File.join(base, name), :encoding => "utf-8"){% endhighlight %}

> 如果还出现这个问题，可以在当前命令行窗口执行以下命令，需要每次打开命令行窗口都执行：

        {% highlight bash %}
        chcp 65001
        #DOS命令chcp显示或设置活动代码页（字符集编码）编号，65001对应UTF-8{% endhighlight %}

##### 3. 发布
> 可以通过[GitHub for Windows](http://github-windows.s3.amazonaws.com/GitHubSetup.exe)进行发布操作。也可以使用以下Git命令发布。
        {% highlight bash %}
        git add .
        git commit -m "注释内容"
        git push origin master{% endhighlight %}
##### 4. 域名绑定
> 上传成功之后，等10分钟左右，就可以使用[codingtiger.github.io](http://codingtiger.github.io/)进行访问。如果不想使用这个域名，可以在工程根目录下面创建CNAME文件，写入绑定域名，例如www.wozhimeng.com。

-EOF-
