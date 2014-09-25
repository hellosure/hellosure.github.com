---
layout: post
title: 初学CSS3
category: CSS3
tags: [CSS3,CSS]
---

### CSS框模型

![css](http://images.cnblogs.com/cnblogs_com/ericgisser/0G501FF-3.png "css")

### 边框

通过 CSS3，您能够创建圆角边框，向矩形添加阴影，使用图片来绘制边框 - 并且不需使用设计软件，比如 PhotoShop。

    border-radius
    box-shadow
    border-image

### 背景

    background-size
    background-origin

### 文本效果

    text-shadow
    word-wrap

### 字体

在 CSS3 之前，web 设计师必须使用已在用户计算机上安装好的字体。
通过 CSS3，web 设计师可以使用他们喜欢的任意字体。
当您您找到或购买到希望使用的字体时，可将该字体文件存放到 web 服务器上，它会在需要时被自动下载到用户的计算机上。
您“自己的”的字体是在 CSS3 `@font-face` 规则中定义的。

### 转换

通过 CSS3 转换，我们能够对元素进行移动、缩放、转动、拉长或拉伸。

### 过渡

通过 CSS3，我们可以在不使用 Flash 动画或 JavaScript 的情况下，当元素从一种样式变换为另一种样式时为元素添加效果。

### 动画

通过 CSS3，我们能够创建动画，这可以在许多网页中取代动画图片、Flash 动画以及 JavaScript。
`@keyframes` 规则用于创建动画。在 `@keyframes` 中规定某项 CSS 样式，就能创建由当前样式逐渐改为新样式的动画效果。

### 多列

    column-count
    column-gap
    column-rule

### 用户界面

重设元素尺寸、盒尺寸以及轮廓等

    resize
    box-sizing
    outline-offset

### 优先级

当同一个 HTML 元素被不止一个样式定义时，会使用哪个样式呢？

一般而言，所有的样式会根据下面的规则层叠于一个新的虚拟样式表中，其中数字 4 拥有最高的优先权。

1浏览器缺省设置

2外部样式表（推荐这种方式）

3内部样式表（位于 <head> 标签内部）（当单个文档需要特殊的样式时，就应该使用内部样式表）

4内联样式（在 HTML 元素内部）（当样式仅需要在一个元素上应用一次时，不推荐这样使用）

例如，外部样式表拥有针对 h3 选择器的三个属性：

    h3 {
      color: red;
      text-align: left;
      font-size: 8pt;
      }

而内部样式表拥有针对 h3 选择器的两个属性：

    h3 {
      text-align: right; 
      font-size: 20pt;
      }

假如拥有内部样式表的这个页面同时与外部样式表链接，那么 h3 得到的样式是：

    color: red; 
    text-align: right; 
    font-size: 20pt;

### 选择器

在这个例子中，h1 是选择器，`color` 和 `font-size` 是属性，red 和 14px 是值。

`h1 {color:red; font-size:14px;}`作用是将 h1 元素内的文字颜色定义为红色，同时将字体大小设置为 14 像素。

### 选择器分组

在下面的例子中，我们对所有的标题元素进行了分组。所有的标题元素都是绿色的。

    h1,h2,h3,h4,h5,h6 {
      color: green;
    }

### 继承

如果你不希望 "Verdana, sans-serif" 字体被所有的子元素继承，又该怎么做呢？比方说，你希望段落的字体是 Times。没问题。创建一个针对 p 的特殊规则，这样它就会摆脱父元素的规则：

    body  {
     font-family: Verdana, sans-serif;
    }
    p  {
     font-family: Times, "Times New Roman", serif;
    }

### 派生选择器

    strong {
     color: black;
    }
    h2 {
     color: red;
    }
    h2 strong {
     color: blue;
    }
    
其中h2 strong是个派生选择器，指的是h2中的strong。

### id 选择器

以 "#" 来定义。

    #red {color:red;}
    <p id="red">这个段落是红色。</p>

### 类选择器

以一个点号显示。

    .center {text-align: center}
    
在上面的例子中，所有拥有 `center` 类的 HTML 元素均为居中。

    .fancy td {
    	color: #f60;
    	background: #666;
    	}
    	
指的是 `.fancy`中的`td`，名为 fancy 的更大的元素可能是一个表格或者一个 div。

    td.fancy {
    	color: #f60;
    	background: #666;
    	}
    	
在上面的例子中，类名为 fancy 的表格单元将是带有灰色背景的橙色。`<td class="fancy">`

### 属性选择器

下面的例子为带有 title 属性的所有元素设置样式：

    [title]
    {
    color:red;
    }

下面的例子为 title="W3School" 的所有元素设置样式：

    [title=W3School]
    {
    border:5px solid blue;
    }

### 外边距合并

外边距合并（叠加）是一个相当简单的概念。但是，在实践中对网页进行布局时，它会造成许多混淆。
简单地说，外边距合并指的是，当两个垂直外边距相遇时，它们将形成一个外边距。合并后的外边距的高度等于两个发生合并的外边距的高度中的较大者。

### 伪类

用于向某些选择器添加特殊的效果：

    a:link {color: #FF0000}		/* 未访问的链接 */
    a:visited {color: #00FF00}	/* 已访问的链接 */
    a:hover {color: #FF00FF}	/* 鼠标移动到链接上 */
    a:active {color: #0000FF}	/* 选定的链接 */
    
-EOF-
