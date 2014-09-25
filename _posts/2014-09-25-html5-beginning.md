---
layout: post
title: 初学HTML5
category: HTML5
tags: [HTML5,HTML]
---

在[w3school](http://www.w3school.com.cn/html5/index.asp)对html5进行了初步的了解。

### 思想

HTML5减少对外部插件的需求-->用video和audio元素扼杀flash。

HTML5独立于设备-->对各种设备友好

### 新特性

1. 用于绘画的 `canvas` 元素
2. 用于媒介回放的 `video` 和 `audio` 元素
3. 对本地离线存储的更好的支持
4. 新的特殊内容元素，比如 `article`、`footer`、`header`、`nav`、`section`
5. 新的表单控件，比如 `calendar`、`date`、`time`、`email`、`url`、`search`，直接使用即可，非常方便

### 浏览器

最新版本的Safari、Chrome、Firefox 以及 Opera 支持某些 HTML5 特性。Internet Explorer 9 将支持某些 HTML5 特性。

### 媒体

ogg和mp4格式可以覆盖所有支持video的浏览器：

    <video width="320" height="240" controls="controls">
      <source src="movie.ogg" type="video/ogg">
      <source src="movie.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>

相对的，ogg和mp3可以覆盖所有支持audio的浏览器。

### 拖放

在HTML5 中，拖放是标准的一部分，任何元素都能够拖放，抓取对象以后拖到另一个位置。
被拖放的元素中：`draggable="true"`，且设计`ondragstart`事件`="drag(event)"`
其中JS中`dataTransfer.setData()`方法设置被拖数据的数据类型和值：

    function drag(ev){
        ev.dataTransfer.setData("Text",ev.target.id);
    }
    
被放进的元素中：调用`ondragover`事件，放置的动作会发生`ondrop`事件：

    function drop(ev){
        ev.preventDefault(); //调用 preventDefault() 来避免浏览器对数据的默认处理（drop 事件的默认行为是以链接形式打开）
        var data=ev.dataTransfer.getData("Text"); //通过 dataTransfer.getData("Text") 方法获得被拖的数据。该方法将返回在 setData() 方法中设置为相同类型的任何数据。被拖数据是被拖元素的 id ("drag1")
        ev.target.appendChild(document.getElementById(data)); //把被拖元素追加到放置元素（目标元素）中
    }

### Canvas

`canvas`元素本身是没有绘图能力的。所有的绘制工作必须在JavaScript内部完成。

    var c=document.getElementById("myCanvas");
    var cxt=c.getContext("2d");

然后`cxt`就有了很多2D的绘图方法可以调用。

### 矢量图SVG

放大后图像不会失真，和分辨率无关。
SVG使用XML编写的，HTML5有`svg`元素可以直接引用。

对比：

Canvas最适合图像密集型的游戏，其中的许多对象会被频繁重绘。

SVG最适合带有大型渲染区域的应用程序（比如谷歌地图）。

### Geolocation

    getCurrentPosition() 
    watchPosition() 

### 在客户端存储数据

HTML5 提供了两种在客户端存储数据的新方法：

1. `localStorage` - 没有时间限制的数据存储

2. `sessionStorage` - 针对一个 session 的数据存储

之前，这些都是由 `cookie` 完成的。但是 `cookie` 不适合大量数据的存储，因为它们由每个对服务器的请求来传递，这使得 `cookie` 速度很慢而且效率也不高。

在 HTML5 中，数据不是由每个服务器请求传递的，而是只有在请求时使用数据。它使在不影响网站性能的情况下存储大量数据成为可能。

对于不同的网站，数据存储于不同的区域，并且一个网站只能访问其自身的数据。

HTML5 使用 JavaScript 来存储和访问数据。

### 应用程序缓存

HTML5 引入了应用程序缓存，这意味着 web 应用可进行缓存，并可在没有因特网连接时进行访问。
应用程序缓存为应用带来三个优势：

1. 离线浏览 - 用户可在应用离线时使用它们

2. 速度 - 已缓存资源加载得更快

3. 减少服务器负载 - 浏览器将只从服务器下载更新过或更改过的资源。

### 后台运行web worker

web worker 是运行在后台的 JavaScript，独立于其他脚本，不会影响页面的性能。您可以继续做任何愿意做的事情：点击、选取内容等等，而此时 web worker 在后台运行。

创建一个Worker：`var w=new Worker("/example/html5/demo_workers.js");`。

`postMessage()` 方法 - 它用于向 HTML 页面传回一段消息。

当 web worker 传递消息时，会执行事件监听器`onmessage`中的代码。

如需终止 web worker，并释放浏览器/计算机资源，请使用 `terminate()` 方法。

### 服务器发送事件

`Server-Sent` 事件 - 单向消息传递

Server-Sent 事件指的是网页自动获取来自服务器的更新。

以前也可能做到这一点，前提是网页不得不询问是否有可用的更新。通过服务器发送事件，更新能够自动到达。

例子：Facebook/Twitter 更新、估价更新、新的博文、赛事结果等。

    var source=new EventSource("demo_sse.php");
    source.onmessage=function(event){
        document.getElementById("result").innerHTML+=event.data + "<br />";
    };

EventSource 对象用于接收服务器发送事件通知。

创建一个新的 EventSource 对象，然后规定发送更新的页面的 URL（本例中是 "demo_sse.php"）

每接收到一次更新，就会发生 `onmessage` 事件。

当 `onmessage` 事件发生时，把已接收的数据推入 id 为 "result" 的元素中。

为了让上面的例子可以运行，您还需要能够发送数据更新的服务器（比如 PHP 和 ASP）。

服务器端事件流的语法是非常简单的。把 `"Content-Type"` 报头设置为 `"text/event-stream"`。现在，您可以开始发送事件流了。

PHP 代码 (demo_sse.php)：

{% highlight php %}
<?php
header('Content-Type: text/event-stream');
header('Cache-Control: no-cache');

$time = date('r');
echo "data: The server time is: {$time}\n\n";
flush();
?>
{% endhighlight %}


### HTML `<div>` 元素

HTML `<div>` 元素是块级元素，它是可用于组合其他 HTML 元素的容器。

`<div>` 元素没有特定的含义。除此之外，由于它属于块级元素，浏览器会在其前后显示折行。

如果与 CSS 一同使用，`<div>` 元素可用于对大的内容块设置样式属性。

`<div>` 元素的另一个常见的用途是文档布局。它取代了使用表格定义布局的老式方法。


### HTML `<span>` 元素

HTML `<span>` 元素是内联元素，可用作文本的容器。

`<span>` 元素也没有特定的含义。

当与 CSS 一同使用时，`<span>` 元素可用于为部分文本设置样式属性。


### iframe

用于在网页内显示网页。

{% highlight html %}
<html>
  <body>
    <iframe src="/example/html/demo_iframe.html" width="200" height="200" frameborder="0"></iframe>
  </body>
</html>
{% endhighlight %}

其中frameborder是边框大小。

iframe 可用作链接的目标（target）。

链接的 `target` 属性必须引用 iframe 的 `name` 属性：

{% highlight html %}
<iframe src="demo_iframe.htm" name="iframe_a"></iframe>
<p><a href="http://www.w3school.com.cn" target="iframe_a">W3School.com.cn</a></p>
{% endhighlight %}


### HTML header

1. `<title>` 标签定义文档的标题。
定义浏览器工具栏中的标题
提供页面被添加到收藏夹时显示的标题
显示在搜索引擎结果中的页面标题

2. `<link>` 标签最常用于连接样式表
`<script>` 标签用于定义客户端脚本，比如 JavaScript。
而`<style>` 标签用于为 HTML 文档定义样式信息

3. `<meta>` 标签提供关于 HTML 文档的元数据。元数据不会显示在页面上，但是对于机器是可读的。
典型的情况是，meta 元素被用于规定页面的描述、关键词、文档的作者、最后修改时间以及其他元数据。
`<meta>` 标签始终位于 head 元素中。
元数据可用于浏览器（如何显示内容或重新加载页面），搜索引擎（关键词），或其他 web 服务。

### 空格

浏览器总是会截短 HTML 页面中的空格。如果您在文本中写 10 个空格，在显示该页面之前，浏览器会删除它们中的 9 个。如需在页面中增加空格的数量，您需要使用 `&nbsp;` 字符实体。

### 编码

URL 编码会将字符转换为可通过因特网传输的格式。

URL 只能使用 ASCII 字符集来通过因特网进行发送。

由于 URL 常常会包含 ASCII 集合之外的字符，URL 必须转换为有效的 ASCII 格式。

URL 编码使用 "%" 其后跟随两位的十六进制数来替换非 ASCII 字符。

URL 不能包含空格。URL 编码通常使用 + 来替换空格。

### XHTML

我理解就是我认为的HTML，比如<>必须关闭等要求。

-EOF-
