---
layout: post
title: 从零开始开发一个动态网站
category: JavaScript
tags: [JavaScript,jQuery]
---

### 引入CSS和JS文件

{% highlight html %}
<!doctype html>
<html>
  <head>
    <link href="style.css" rel="stylesheet">
  </head>
  <body>
    ...

    <script src="jquery.min.js"></script>
    <script src="app.js"></script>
  </body>
</html>
{% endhighlight %}

CSS在`head`中用`link`引用。
JS在`body`中用`script`引用。

### jQuery

{% highlight javascript %}
var main = function() {
  $('.dropdown-toggle').click(function() {
    $('.dropdown-menu').toggle();
  });
}

$(document).ready(main);
{% endhighlight %}

定义了而一个`main`函数，`$(document).ready(main)`表示**当页面加载完毕后**就执行`main`函数。

jQuery的选择器是以CSS选择器为基础的。

jQuery的`toggle()`方法用于切换元素的可见状态：如果被选元素可见，则隐藏这些元素，如果被选元素隐藏，则显示这些元素。

`$()`表示用CSS选择器构造一个jQuery对象。

![jquery](http://s3.amazonaws.com/codecademy-content/courses/ltp2/img/yfp/summary.svg "jquery")

{% highlight javascript %}
var main = function() {
  $('.icon-menu').click(function() {
    $('.menu').animate({
      left: "0px"
    }, 200);

    $('body').animate({
      left: "285px"
    }, 200);
  });


  $('.icon-close').click(function() {
    $('.menu').animate({
      left: "-285px"
    }, 200);

    $('body').animate({
      left: "0px"
    }, 200);
  });
};
$(document).ready(main);
{% endhighlight %}

jQuery的`animate()`用来实现动画效果，这里是在左侧显示出`.menu`左侧菜单栏，然后把`body`推到右边，然后点击关闭之后再复原。

### JavaScript

单引号和双引号都可以用来表示字符串。

`===`是等于判断符。

我觉得JS的`function`写法很有趣，相当于是把`function`也当做一个`var`，然后用`function`来赋值。

