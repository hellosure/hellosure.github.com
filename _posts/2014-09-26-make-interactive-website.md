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

我觉得JS的`function`写法很有趣，相当于是把`function`也当做一个`var`，然后用`function`来赋值。

### Event

用户与页面的互动可以产生事件，比如点击。

响应事件的函数叫做`event handler`：

{% highlight javascript %}
var main = function() {
  $(".like-button").click(function() {
    $(this).toggleClass("active");
  });
};
{% endhighlight %}

`this`表示当前所点击的HTML元素，那么也就是说`$(this)`就表示当前所点击的jQuery对象，在这里也就是`$(".like-button")`。

jQuery的`toggleClass()`用来改变样式效果，这个`.active`是一个CSS类。

{% highlight javascript %}
var main = function() {
  $(document).keypress(function() {
		$(".btn").toggleClass("btn-like");
  });
};
$(document).ready(main);
{% endhighlight %}

`document`表示整个页面，那么`$(document)`表示整个页面所表示的jQuery对象。


{% highlight javascript %}
var main = function() {
  $(document).keypress(function(event) {
   if(event.which == 103){
     $(".btn").toggleClass("btn-like");
   }
  });
};
{% endhighlight %}

点击不同的按键，有不同的`event code`，这里讲`event`作为方法参数。

----

html标签的中的class属性中间存在空格,表示是此标签引用两个class：

    <div class="description row"></div>

这里`.description`和`.row`都可以引用到这个元素。

{% highlight javascript %}
var main = function() {
  $('.article').click(function() {
    $('.article').removeClass('current');
    $('.description').hide();

    $(this).addClass('current');
    $(this).children('.description').show();
  });
};
{% endhighlight %}

`$(this).children('.description')`指的是当前对象子对象中`.description`的类。

`hide()`和`show()`是jQuery中常用方法，用来控制隐藏和显示。

这里用`current`的class来标示哪个是当前元素。

#### `==`与`===`

1. 对于`string`,`number`等基础类型，==和===是有区别的
1）不同类型间比较，==之比较“转化成同一类型后的值”看“值”是否相等，===如果类型不同，其结果就是不等
2）同类型比较，直接进行“值”比较，两者结果一样

2. 对于`Array`,`Object`等高级类型，==和===是没有区别的
进行“指针地址”比较。

### DOM操作

#### `$()`

两种用途：

1. 选取已存在的HTML元素，比如`$('p')`选取页面中所有的`<p>`元素。

2. 为页面创建新的HTML元素，比如`$('<h1>')`就表示创建一个新的`<h1>`元素，其中`<>`表明是创建而不是选取。
 
#### `.text()`

为HTML元素添加文本。

![text](http://s3.amazonaws.com/codecademy-content/courses/ltp2/img/dom/text.svg "text")

#### `.appendTo()`

{% highlight javascript %}
$('.btn').click(function() {
  $('<li>').text('New item').appendTo('.items');
});
{% endhighlight %}

`$('<li>')`创建一行，且文本显示为New item，并且加到`.items`尾部。

另外，相对应的加在首部，用`.prependTo()`。

从首部删除用`remove()`。

#### `.hide()`

hides the selected HTML element

#### `.show()`

displays an element

#### `.toggle()`

alternates hiding and showing an element

#### `.addClass()`

adds a CSS class to an element

#### `.removeClass()`

removes a class from an element

#### `.toggleClass()`

alternates adding and removing a class from an element

----

### DOM

Document Object Model (DOM)

![dom](http://s3.amazonaws.com/codecademy-content/courses/ltp2/img/dom/dom.svg "dom")

#### 节点关系

![parent](http://s3.amazonaws.com/codecademy-content/courses/ltp2/img/dom/traversal.svg "parent")

`.next()`：获取下一个兄弟节点。

`.prev()`：获取上一个兄弟节点。

`.children()`：获取所有子节点。

----

### 动画效果

`.slideDown()`：以指定的速度从上到下显示出来。

{% highlight javascript %}
$('body').click(function() {
  $('.slide').slideDown(600).addClass('active-slide');
});
{% endhighlight %}

`.slideUp()`：以制定的速度 从下到上显示出来。

`.fadeIn()`：由模糊到清晰的显示出来。

`.fadeOut()`：由清晰到模糊的消失出去。

`.animate()`：第一个参数是一组CSS属性，第二个参数是动画用时。

{% highlight javascript %}
$('.icon-menu').click(function() {
  $('.menu').animate({
      width: "193px"
    },
    300);
});
{% endhighlight %}


-EOF-
