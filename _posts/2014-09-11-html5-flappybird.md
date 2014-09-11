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
var birdwidth=46;
var birdheight=32;
var birdx=192-birdwidth;
var birdy=224-birdheight;
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

#### canvas知识点补充

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


### 游戏基本场景的绘制

游戏中的基本场景包括上方静止的背景，下方移动地面的绘制，以及管道的绘制。

#### 静止的背景图片

首先是静止的图片，只要用`Image`对象保存图片地址后使用`drawImage`指定位置和大小就行了。

{% highlight javascript %}
var backgroundwidth=384;
var backgroundheight=448;	
var upbackground;	
function init(){	
  upbackground = new Image();
  upbackground.src = "images/background.png";
  ctx.drawImage(upbackground, 0, 0, backgroundwidth, backgroundheight);
}
{% endhighlight %}

#### 动态的地面

下方动态的地面较为复杂，先贴出代码:

{% highlight javascript %}
//绘制下方的动态背景
function drawmovingscene(){
  if(bottomstate==1){
    for(i=0;i<times;i++)
	ctx.drawImage(bottombackground,groundwidth*i,backgroundheight,groundwidth,groundheight);
    bottomstate=2;
    }
  else if(bottomstate==2){
    for(i=0;i<times;i++)
	ctx.drawImage(bottombackground,groundwidth*(i-0.25),backgroundheight,groundwidth,groundheight);
    bottomstate=3;
    }
  else if(bottomstate==3){
    for(i=0;i<times;i++)
	ctx.drawImage(bottombackground,groundwidth*(i-0.5),backgroundheight,groundwidth,groundheight);
    bottomstate=4;
    }
  else if(bottomstate==4){
    for(i=0;i<times;i++)
	ctx.drawImage(bottombackground,groundwidth*(i-0.75),backgroundheight,groundwidth,groundheight);
    bottomstate=1;
    }
}
{% endhighlight %}

这里找到的地面图片是一窄条，因此想要绘制下面的完整地需要先计算出多少条能将下部填满，使用了

{% highlight javascript %}
for(i=0;i<times;i++)
  ctx.drawImage(bottombackground, groundwidth*i, backgroundheight, groundwidth, groundheight);
{% endhighlight %}

就绘制出了下方地面的一帧图像，想要让地面动起来，我选择每一帧都让绘制的图片向左移动1/4宽度，这样就可以在游戏运行时显示地面在移动，这里使用了一个`bottomstate`状态量，以此来记录当前地面的绘制状态，每次加1，到4后下一帧变为1。

> 可以想象这里的`drawmovingscene()`肯定是从主函数`run()`中能调用的，每帧都绘制一次，就得到了动态的地面。另外循环中的`times`是地板图片的条数Math.ceil(boxwidth/groundwidth)+1;  

#### drawImage知识点补充

在画布上定位图像：

    context.drawImage(img,x,y);
    
在画布上定位图像，并规定图像的宽度和高度：

    context.drawImage(img,x,y,width,height);
   
剪切图像，并在画布上定位被剪切的部分：

    context.drawImage(img,sx,sy,swidth,sheight,x,y,width,height);

#### 移动的管道

然后是移动的管道。对于管道的绘制首先需要随机生成若干个管道的高度，并将其并存放在一个`pipeheight`数组中待用

{% highlight javascript %}
//随机生成管道高度数据
function initPipe(){
  for(i=0;i<200;i++)
    pipeheight[i]=Math.ceil(Math.random()*216)+56;//高度范围从56~272
  for(i=0;i<3;i++){	
    pipeoncanvas[i][0]=boxwidth+i*pipeinterval;
    pipeoncanvas[i][1]=pipeheight[pipenumber];
    pipenumber++;
  }
}
{% endhighlight %}

鉴于管道在画面中不会同时出现4个，因此我首先取三个管道高度数据放入pipecanvas数组，并根据画面的宽度和管道的间隔生成管道位置，为绘制管道作准备，这是`pipecanvas`的结构：

{% highlight javascript %}
var pipeoncanvas=[ 	 //要显示在Canvas上的管道的location和height
	[0,0],
	[0,0],
	[0,0]];
{% endhighlight %}

> 因为画面上最多同时出现三个管道，因此这里`pipeoncanvas`创建为只有三个元素

下面就要对管道进行绘制了，先实现一根管道上下两部分的绘制

{% highlight javascript %}
//使用给定的高度和位置绘制上下两根管道
function drawPipe(location,height){
  //绘制下方的管道
  ctx.drawImage(pipeupimage,0,0,pipewidth*2,height*2,location,boxheight-(height+groundheight),pipewidth,height);
  //绘制上方的管道
  ctx.drawImage(pipedownimage,0,793-(backgroundheight-height-blankwidth)*2,pipewidth*2,
    (backgroundheight-height-blankwidth)*2,location,0,pipewidth,backgroundheight-height-blankwidth);
}
{% endhighlight %}

函数比较简单不再赘述，在run函数中加入drawAllPipe函数，来绘制要显示的三根管道:

{% highlight javascript %}
//绘制需要显示的管道
function drawAllPipe(){
  for(i=0;i<3;i++){
    pipeoncanvas[i][0]=pipeoncanvas[i][0]-4.625;
  }
  if(pipeoncanvas[0][0]<=-pipewidth){
    pipeoncanvas[0][0]=pipeoncanvas[1][0];
    pipeoncanvas[0][1]=pipeoncanvas[1][1];
    pipeoncanvas[1][0]=pipeoncanvas[2][0];
    pipeoncanvas[1][1]=pipeoncanvas[2][1];
    pipeoncanvas[2][0]=pipeoncanvas[2][0]+pipeinterval;
    pipeoncanvas[2][1]=pipeheight[pipenumber];
    pipenumber++;
  }
  for(i=0;i<3;i++){
    drawPipe(pipeoncanvas[i][0],pipeoncanvas[i][1]);
  }
}
{% endhighlight %}

这里会先判断第一根管道是否已经移出画布，如果移出了画布则后面的管道数据向前顺延，并将新的管道高度读入第三根管道，处理完后按顺序意思绘制三根管道。

基本场景绘制结束。


-EOF-
