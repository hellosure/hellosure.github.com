---
layout: post
title: AngularJS学习指引
category: JavaScript
tags: [Angular,JavaScript]
---

### 搭建学习环境

安装[node.js](http://nodejs.org/)

安装[karma](http://karma-runner.github.io/0.12/index.html) 

获取[angular-phonecat](https://github.com/angular/angular-phonecat)源代码

    git clone git://github.com/angular/angular-phonecat.git

开启运行 

    cd angular-phonecat
    npm start

访问[http://localhost:8000/app/index.html](http://localhost:8000/app/index.html)可以看到已经运行成功。

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


{% highlight html %}
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

{% endhighlight %}


刷新页面可以看到效果。

### 动态特性

下面利用MVC来显示上面的手机列表：

{% highlight html %}
<html ng-app>
<head>
  ...
  <script src="lib/angular/angular.js"></script>
  <script src="js/controllers.js"></script>
</head>
<body ng-controller="PhoneListCtrl">

  <ul>
    <li ng-repeat="phone in phones">
      {{phone.name}}
    <p>{{phone.snippet}}</p>
    </li>
  </ul>
</body>
</html>

{% endhighlight %}

在`<li>`标签里面的`ng-repeat="phone in phones"`语句是一个AngularJS迭代器。这个迭代器告诉AngularJS用第一个`<li>`标签作为模板为列表中的每一部手机创建一个`<li>`元素。

`ng-controller`是指明控制器的名称，它在js中定义。

![mvc](http://docs.angularjs.org/img/tutorial/tutorial_02.png "mvc")


控制器在app/js/controller.js:

{% highlight javascript %}
function PhoneListCtrl($scope) {
  $scope.phones = [
    {"name": "Nexus S",
     "snippet": "Fast just got faster with Nexus S."},
    {"name": "Motorola XOOM™ with Wi-Fi",
     "snippet": "The Next, Next Generation tablet."},
    {"name": "MOTOROLA XOOM™",
     "snippet": "The Next, Next Generation tablet."}
  ];
}
{% endhighlight %}

手机的数据此时与注入到我们控制器函数的作用域（$scope）相关联。这个控制器的作用域对所有`<body ng-controller="PhoneListCtrl">`标记内部的数据绑定有效。

### 作用域

AngularJS的作用域理论非常重要：一个作用域可以视作模板、模型和控制器协同工作的粘接器。AngularJS使用作用域，同时还有模板中的信息，数据模型和控制器。这些可以帮助模型和视图分离，但是他们两者确实是同步的！任何对于模型的更改都会即时反映在视图上；任何在视图上的更改都会被立刻体现在模型中。

### 迭代器过滤

{% highlight html %}

<div class="container-fluid">
  <div class="row-fluid">
    <div class="span2">
      <!--Sidebar content-->

      Search: <input ng-model="query">

    </div>
    <div class="span10">
      <!--Body content-->

      <ul class="phones">
        <li ng-repeat="phone in phones | filter:query">
          {{phone.name}}
        <p>{{phone.snippet}}</p>
        </li>
      </ul>

       </div>
  </div>
</div>

{% endhighlight %}

我们现在添加了一个`<input>`标签，并且使用AngularJS的`$filter`函数来处理`ng-repeat`指令的输入。

这样允许用户输入一个搜索条件，立刻就能看到对电话列表的搜索结果。

> 数据绑定：这是AngularJS的一个核心特性。当页面加载的时候，AngularJS会根据输入框的属性值名字，将其与数据模型中相同名字的变量绑定在一起，以确保两者的同步性。
在这段代码中，用户在输入框中输入的数据名字称作`query`，会立刻作为列表迭代器（`phone in phones | filter:query`）其过滤器的输入。当数据模型引起迭代器输入变化的时候，迭代器可以高效得更新DOM将数据模型最新的状态反映出来。

filter过滤器：`filter`函数使用`query`的值来创建一个只包含匹配`query`记录的新数组。

`ng-repeat`会根据`filter`过滤器生成的手机记录数据数组来自动更新视图。整个过程对于开发者来说都是透明的。

### 双向绑定



-EOF-
