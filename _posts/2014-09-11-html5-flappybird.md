---
layout: post
title: HTML5和JavaScript实现flappybird
category: HTML5
tags: [HTML5,JavaScript]
---

[在线试玩flappybird](http://hellosure.github.io/flappybird.html)

### 游戏整体框架的搭建

{% highlight html %}
<body onLoad="init();">
  <canvas id="canvas" width="384" height="512" style="margin-top: 8px;">
    Your browser doesn't support the HTML5 element canvas.
  </canvas>
</body>
{% endhighlight %}

#### canvas

大多数 Canvas 绘图 API 都没有定义在 <canvas> 元素本身上，而是定义在通过画布的 getContext() 方法获得的一个“绘图环境”对象上。

如何通过 canvas 元素来显示一个红色的矩形：

{% highlight html %}
<canvas id="myCanvas"></canvas>

<script type="text/javascript">
var canvas=document.getElementById('myCanvas');
var ctx=canvas.getContext('2d');
ctx.fillStyle='#FF0000';
ctx.fillRect(0,0,80,100);
</script>
{% endhighlight %}

-EOF-
