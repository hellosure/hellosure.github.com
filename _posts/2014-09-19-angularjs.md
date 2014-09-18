---
layout: post
title: AngularJS学习指引
category: JavaScript
tags: [Angular,JavaScript]
---

### 搭建学习环境

+ 安装[node.js](http://nodejs.org/)

+ 安装[karma](http://karma-runner.github.io/0.12/index.html) 

+ 获取[angular-phonecat](https://github.com/angular/angular-phonecat)源代码

```
git clone git://github.com/angular/angular-phonecat.git
```
+ 开启运行 

```
cd angular-phonecat
npm start
```

+ 访问[http://localhost:8000/app/index.html](http://localhost:8000/app/index.html)可以看到已经运行成功。

### 引导AngularJS应用

在angular-phonecat/app路径中就是项目代码，先看`index.html`

`ng-app`指令：

    <html lang="en" ng-app>
    
`ng-app`指令标记了AngularJS脚本的作用域，在<html>中添加`ng-app`属性即说明整个`<html>`都是AngularJS脚本作用域。开发者也可以在局部使用`ng-app`指令，如`<div ng-app>`，则AngularJS脚本仅在该`<div>`中运行。

另外可以看到好几行代码载入angular的相关js脚本，当浏览器将整个HTML页面载入完毕后将会执行js脚本，js脚本运行后将会寻找含有`ng-app`指令的HTML标签，该标签即定义了AngularJS应用的作用域。

> AngularJS将会链接根作用域中的DOM，从用ngApp标记的HTML标签开始，逐步处理DOM中指令和绑定。
一旦AngularJS应用引导完毕，它将继续侦听浏览器的HTML触发事件，如鼠标点击事件、按键事件、HTTP传入响应等改变DOM模型的事件。这类事件一旦发生，AngularJS将会自动检测变化，并作出相应的处理及更新。

可以看到完成引导AngularJS应用非常简便：
1. `<html>`标记`ng-app`
2. 引入所需angular的相关js脚本

### 双大括号绑定的AngularJS表达式

在`index.html`中添加表达式：

    <p>Nothing here {{'yet' + '!'}}</p>
    <p>1 + 2 = {{ 1 + 2 }}</p>

保存之后，不需要重启server，刷新页面即可看到页面中显示：

    Nothing here yet!
    1 + 2 = 3

这里演示了AngularJS模板的核心功能：绑定，这个绑定由双大括号{{}}和表达式组成。

> AngularJS表达式是一种类似于JavaScript的代码片段，AngularJS表达式仅在AngularJS的作用域中运行，而不是在整个DOM中运行。
这个绑定告诉AngularJS需要运算其中的表达式并将结果插入DOM中，接下来我们将看到，DOM可以随着表达式运算结果的改变而实时更新。

### 静态模板

在`index.html`中添加静态代码：

```
<ul>
    <li>
        <span>Nexus S</span>
        <p>
        Fast just got faster with Nexus S.
        </p>
    </li>
    <li>
        <span>Motorola XOOM™ with Wi-Fi</span>
        <p>
        The Next, Next Generation tablet.
        </p>
    </li>
</ul>

<p>Total number of phones: 2</p>
```

刷新页面可以看到效果。

### 动态特性

