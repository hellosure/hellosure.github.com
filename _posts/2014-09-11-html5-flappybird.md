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

{% highlight javascript %}
var fps=30;				//游戏的帧数，推荐在30~60之间
function init(){
	ctx=document.getElementById('canvas').getContext('2d');	
	ctx.lineWidth=2;
	canvas=document.getElementById("canvas");
	setInterval(run, 1000/fps);
}
{% endhighlight %}
游戏主逻辑`run()`将会以每秒fps帧的速度运行，这和后面绘制移动物体有关。

还有一些全局变量的设置
{% highlight javascript %}
var boxx=0;
var boxy=0;
var boxwidth=384;
var boxheight=512;
var backgroundwidth=384;
var backgroundheight=448;
var groundwidth=18.5;
var groundheight=64;
var	birdwidth=46;
var	birdheight=32;
var	birdx=192-birdwidth;
var	birdy=224-birdheight;
var birdvy=0;        //鸟初始的y轴速度
var gravity=1;		 //重力加速度
var jumpvelocity=11;	 //跳跃时获得的向上速度
var pipewidth=69;	 //管道的宽度
var blankwidth=126;  //上下管道之间的间隔
var pipeinterval=pipewidth+120;	//两个管道之间的间隔
var birdstate;
var upbackground;
var bottombackground;
var birdimage;
var pipeupimage;
var pipedownimage;
var pipenumber=0;		//当前已经读取管道高度的个数
var fps=30;				//游戏的帧数，推荐在30~60之间
var gamestate=1;		//游戏状态：0--未开始，1--已开始，2--已结束
var times;
var canvas;
var ctx;
var i;
var bottomstate;
var pipeheight=[];
var pipeoncanvas=[ 	 //要显示在Canvas上的管道的location和height
	[0,0],
	[0,0],
	[0,0]];
{% endhighlight %}

#### canvas

大多数 Canvas 绘图 API 都没有定义在 `<canvas>` 元素本身上，而是定义在通过画布的 `getContext()` 方法获得的一个“绘图环境”对象上。

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
