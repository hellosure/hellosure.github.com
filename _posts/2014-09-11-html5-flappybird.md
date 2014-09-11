---
layout: post
title: HTML5和JavaScript实现flappybird
category: HTML5
tags: [HTML5,JavaScript]
---

### 在线试玩

[flappybird](http://hellosure.github.io/flappybird.html)

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

{% highlight javascript %}
//游戏的主要逻辑及绘制  
function run(){  
  //游戏未开始  
  if(gamestate==0){  
    drawBeginScene();   //绘制开始场景  
    drawBird();         //绘制鸟  
    drawTip();          //绘制提示  
  }  
  //游戏进行中  
  if(gamestate==1){  
      birdvy=birdvy+gravity;  
      drawScene();        //绘制场景  
      drawBird();         //绘制鸟  
      drawScore();        //绘制分数  
      checkBird();        //检测鸟是否与物体发生碰撞  
  }  
  //游戏结束  
  if(gamestate==2){  
    if(birdy+birdheight<backgroundheight)    //如果鸟没有落地  
      birdvy=birdvy+gravity;  
    else {  
      birdvy=0;  
      birdy=backgroundheight-birdheight;  
    }  
    drawEndScene();     //绘制结束场景  
    drawBird();         //绘制鸟  
    drawScoreBoard();   //绘制分数板  
  }  
} 
{% endhighlight %}

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

### 鸟的绘制

这里的鸟有一个扇翅膀的动作，我拿到的图片是三个动作连在一起的，因此需要对图片进行裁剪，每次使用1/3，用状态量需要记录下鸟当前的翅膀状态，并根据状态决定下一帧的绘制，代码如下：

{% highlight javascript %}
function drawBird(){
  birdy=birdy+birdvy;
  if(birdstate==1||birdstate==2||birdstate==3){
    ctx.drawImage(birdimage,0,0,92,64,birdx,birdy,birdwidth,birdheight);
    birdstate++;
  }
  else if(birdstate==4||birdstate==5||birdstate==6){
    ctx.drawImage(birdimage,92,0,92,64,birdx,birdy,birdwidth,birdheight);
    birdstate++;
  }
  else if(birdstate==7||birdstate==8||birdstate==9){
    ctx.drawImage(birdimage,184,0,92,64,birdx,birdy,birdwidth,birdheight);
    birdstate++;
    if(birdstate==9) birdstate=1;
  }
}
{% endhighlight %}

在反复尝试后，这里我选择3帧改变一次翅膀的位置，每帧状态量加1。
这里有必要说一下`drawImage`这个函数，在使用9个参数的时候，第2-5个参数可以指定位置和宽高对图片进行裁剪，有兴趣的同学可以去查一下相关的资料。

游戏开始时需要设定鸟的初始位置。要让鸟移动起来，还要给鸟添加纵向的速度值，在游戏开始时这个值会是0。

{% highlight javascript %}
  birdy=birdy+birdvy;
  birdvy=birdvy+gravity;
{% endhighlight %}

每一帧鸟的位置都是由上一帧的位置加上速度决定的，在运行过程中每一帧速度都会减去重力值（由我设定的），在检测到用户输入会赋给鸟一个固定的速度（后面会提到），形成了跳跃的动作。
至此，我们在一帧中已经绘制了基本场景和鸟，下面是碰撞检测。

### 碰撞检测

这里我们需要依次检测鸟是否与管道以及地面发生碰撞。

{% highlight javascript %}
function checkBird(){
  //先判断第一组管道
  //如果鸟在x轴上与第一组管道重合
  if(birdx+birdwidth>pipeoncanvas[0][0]&&birdx+birdwidth<pipeoncanvas[0][0]+pipewidth+birdwidth){
    //如果鸟在y轴上与第一组管道上部或下部重合
    if(birdy<backgroundheight-pipeoncanvas[0][1]-blankwidth||birdy+birdheight>backgroundheight-pipeoncanvas[0][1])
      gamestate=2;	//游戏结束
    }
  //判断第二组管道
  //如果鸟在x轴上与第二组管道重合
  else if(birdx+birdwidth>pipeoncanvas[1][0]&&birdx+birdwidth<pipeoncanvas[1][0]+pipewidth+birdwidth){
    //如果鸟在y轴上与第二组管道上部或下部重合
    if(birdy<backgroundheight-pipeoncanvas[1][1]-blankwidth||birdy+birdheight>backgroundheight-pipeoncanvas[1][1])
      gamestate=2;	//游戏结束
    }
  //判断是否碰撞地面
  if(birdy+birdheight>backgroundheight)
    gamestate=2;		//游戏结束
}
{% endhighlight %}

这里的注释比较详细，我简单解释一下，判断会先看鸟在x轴上是否与某一管道有重合，如果有则再检测y轴上是否有重合，两项都符合则游戏结束。地面则较为简单。

### 添加键盘和鼠标控制

想要在HTML中读取用户输入，需要在init中增加监听事件:

{% highlight javascript %}
  canvas.addEventListener("mousedown",mouseDown,false);
  window.addEventListener("keydown",keyDown,false);
{% endhighlight %}

mousedow字段监听鼠标按下事件并调用`mouseDown`函数，keydown字段监听按键事件并调用`keyDown`函数。
这两个函数定义如下: 

{% highlight javascript %}
//处理键盘事件
function keyDown(){
  if(gamestate==0){
    playSound(swooshingsound,"sounds/swooshing.mp3");
    birdvy=-jumpvelocity;
    gamestate=1;
  }
  else if(gamestate==1){
    playSound(flysound,"sounds/wing.mp3");
    birdvy=-jumpvelocity;
  }
}
{% endhighlight %}

键盘不区分按下的键，会给将鸟的速度变为一个设定的值（jumpvelocity）

{% highlight javascript %}
function mouseDown(ev){
  var mx;			//存储鼠标横坐标
  var my;			//存储鼠标纵坐标
  if ( ev.layerX ||  ev.layerX == 0) { // Firefox
    mx= ev.layerX;
    my = ev.layerY;
  } else if (ev.offsetX || ev.offsetX == 0) { // Opera
    mx = ev.offsetX;
    my = ev.offsetY;
  }
  if(gamestate==0){
    playSound(swooshingsound,"sounds/swooshing.mp3");
    birdvy=-jumpvelocity;
    gamestate=1;
  }
  else if(gamestate==1){
    playSound(flysound,"sounds/wing.mp3");
    birdvy=-jumpvelocity;
  }
  //游戏结束后判断是否点击了重新开始
  else if(gamestate==2){
    //鼠标是否在重新开始按钮上
    if(mx>boardx+14&&mx<boardx+89&&my>boardy+boardheight-40&&my<boardy+boardheight){
      playSound(swooshingsound,"sounds/swooshing.mp3");
      restart();
    }
  }
}
{% endhighlight %}

### 增加计分，添加开始提示和结束积分板

计分的实现比较简单，使用一全局变量即可，在每次通过管道时分数加1，并根据全局变量的值将分数绘制在画布上。

{% highlight javascript %}
var highscore=0;		//得到过的最高分
var score=0				//目前得到的分数
//通过了一根管道加一分
if(birdx+birdwidth>pipeoncanvas[0][0]-movespeed/2&&birdx+birdwidth<pipeoncanvas[0][0]+movespeed/2
  ||birdx+birdwidth>pipeoncanvas[1][0]-movespeed/2&&birdx+birdwidth<pipeoncanvas[1][0]+movespeed/2){
    playSound(scoresound,"sounds/point.mp3");
    score++;
}

function drawScore(){
  ctx.fillText(score,boxwidth/2-2,120);
}
{% endhighlight %}

在绘制文本之前需要先指定字体和颜色:
    
    	ctx.font="bold 40px HarlemNights";			//设置绘制分数的字体 
	ctx.fillStyle="#FFFFFF";

开始时的提示和结束的计分板都是普通的图片，计分板上用两个文本绘制了当前分数和得到的最高分数

{% highlight javascript %}
function drawTip(){
  ctx.drawImage(tipimage,birdx-57,birdy+birdheight+10,tipwidth,tipheight);	
}

//绘制分数板
function drawScoreBoard(){
  //绘制分数板
  ctx.drawImage(boardimage,boardx,boardy,boardwidth,boardheight);	
  //绘制当前的得分
  ctx.fillText(score,boardx+140,boardheight/2+boardy-8);//132
  //绘制最高分
  ctx.fillText(highscore,boardx+140,boardheight/2+boardy+44);//184
}
{% endhighlight %}

这里的最高分highscroe会在每次游戏结束时更新

{% highlight javascript %}
//刷新最好成绩
function updateScore(){
  if(score>highscore)
    highscore=score;
}
{% endhighlight %}

### 给鸟添加俯仰动作，添加音效

我察觉到鸟的动作还不是非常丰富，有必要给鸟添加上仰、俯冲的动作使其更富动感。代码如下：

{% highlight javascript %}
function drawBird(){
  birdy=birdy+birdvy;
  if(gamestate==0){
    drawMovingBird();
  }
  //根据鸟的y轴速度来判断鸟的朝向,只在游戏进行阶段生效
  else if(gamestate==1){
    ctx.save();
    if(birdvy<=8){
      ctx.translate(birdx+birdwidth/2,birdy+birdheight/2);
      ctx.rotate(-Math.PI/6);
      ctx.translate(-birdx-birdwidth/2,-birdy-birdheight/2);	
    }
    if(birdvy>8&&birdvy<=12){
      ctx.translate(birdx+birdwidth/2,birdy+birdheight/2);
      ctx.rotate(Math.PI/6);
      ctx.translate(-birdx-birdwidth/2,-birdy-birdheight/2);	
    }
    if(birdvy>12&&birdvy<=16){
      ctx.translate(birdx+birdwidth/2,birdy+birdheight/2);
      ctx.rotate(Math.PI/3);
      ctx.translate(-birdx-birdwidth/2,-birdy-birdheight/2);	
    }
    if(birdvy>16){
      ctx.translate(birdx+birdwidth/2,birdy+birdheight/2);
      ctx.rotate(Math.PI/2);
      ctx.translate(-birdx-birdwidth/2,-birdy-birdheight/2);	
    }
    drawMovingBird();
    ctx.restore();
  }
  //游戏结束后鸟头向下并停止活动
  else if(gamestate==2){
    ctx.save();
    ctx.translate(birdx+birdwidth/2,birdy+birdheight/2);
    ctx.rotate(Math.PI/2);
    ctx.translate(-birdx-birdwidth/2,-birdy-birdheight/2);	
    ctx.drawImage(birdimage,0,0,92,64,birdx,birdy,birdwidth,birdheight);
    ctx.restore();
  }
}
{% endhighlight %}

这里使用了图片的旋转操作。过程是保存绘画状态，将绘画原点移到鸟的中心，旋转一定的角度，将原点移回原位（防止影响其他物体的绘制），恢复绘画状态：

旋转的角度我根据鸟当前的速度来判断，birdvy<=8时向上旋转30度，8<birdvy<=12时向下旋转30度，12<birdvy<=16时向下旋转60度，birdvy>16时向下旋转90度，在确定了旋转角度后再使用之前的方法进行鸟的绘制，这样就同时实现了鸟的俯仰和扇动翅膀。

#### 音效

关于在HTML中使用音效，我查阅了许多资料，经过反复试验后，排除了许多效果不佳的方法，最终选择使用audio这个HTML标签来实现音效的播放。
要使用audio标签，首先要在HTML的body部分定义：

{% highlight html %}
<audio id="flysound" playcount="1" autoplay="true" src="">
Your browser doesn't support the HTML5 element audio.
</audio>
<audio id="scoresound" playcount="1" autoplay="true" src="">
Your browser doesn't support the HTML5 element audio.
</audio>
<audio id="hitsound" playcount="1" autoplay="true" src="">
Your browser doesn't support the HTML5 element audio.
</audio>
<audio id="deadsound" playcount="1" autoplay="true" src="">
Your browser doesn't support the HTML5 element audio.
</audio>
<audio id="swooshingsound" playcount="1" autoplay="true" src="">
Your browser doesn't support the HTML5 element audio.
</audio>
{% endhighlight %}

为了时播放音效时不发生冲突，我为每个音效定义了一个audio标签，这样在使用中就不会出现问题。
然后将定义的变量与标签绑定：

{% highlight javascript %}
//各种音效
var flysound;		//飞翔的声音
var scoresound;		//得分的声音
var hitsound;		//撞到管道的声音
var deadsound;		//死亡的声音
var swooshingsound;		//切换界面时的声音
function init(){
  flysound = document.getElementById('flysound');
  scoresound = document.getElementById('scoresound');
  hitsound = document.getElementById('hitsound');
  deadsound = document.getElementById('deadsound');
  swooshingsound = document.getElementById('swooshingsound');
}
{% endhighlight %}

再定义用来播放音效的函数

{% highlight javascript %}
function playSound(sound,src){
  if(src!='' && typeof src!=undefined){
    sound.src = src;
  }
}
{% endhighlight %}

函数的两个参数分别指定了要使用的标签和声音文件的路径，接下来只要在需要播放音效的地方调用这个函数并指定声音文件就行了，比如

    playSound(flysound,"sounds/wing.mp3"); 
    
这里在点击键盘按键且游戏正在运行的时候使鸟跳跃，并播放扇动翅膀的音效。

### 小结

这个例子是借用的别人的，给我的感觉做这个游戏需要有这样一些基本的想法：

1. 动画的效果就是每帧重新绘制，所以游戏的整体基调就是不停在重绘。
2. 动画的效果也可能需要state来配合，比如一组重复动作的每个阶段。
3. 图片需要进行剪裁和放置。
4. 有些检测比如碰撞检测都是在重绘之后判断图像来实现。
5. 监听事件比较简单只需要addEventListener通知到function即可。
6. html5的支持有canvas、audio、以及画布上的一些API比如drawImage。


另外就是大量的绘制图像，X/Y坐标的选取，这就不纯是技术问题了。


-EOF-
