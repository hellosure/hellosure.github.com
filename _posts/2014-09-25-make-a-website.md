---
layout: post
title: 从零开始开发一个静态网站
category: CSS
tags: [HTML,CSS]
---

### [make a website](www.codecademy.com/en/skills/make-a-website/resume)

#### HTML

{% highlight html %}
<!DOCTYPE html>
<html>
  <head>
    <link href="http://s3.amazonaws.com/codecademy-content/courses/ltp/css/shift.css" rel="stylesheet">
    <link rel="stylesheet" href="main.css">
  </head>
  
  <body>
    <div class="nav">
      <div class="container">
        <ul>
          <li><a href="#">Airbnb logo</a></li>
          <li><a href="#">Browse</a></li>
        </ul>
        <ul>
          <li><a href="#">Sign Up</a></li>
          <li><a href="#">Log In</a></li>
          <li><a href="#">Help</a></li>
        </ul>
      </div>
    </div>

    <div class="jumbotron">
      <div class="container">
        <h1>Find a place to stay.</h1>
        <p>Rent from people in over 34,000 cities and 192 countries.</p>
      </div>
    </div> 

    <div class="learn-more">
	  <div>
		<div>
	      <div>
			<h3>Travel</h3>
			<p>From apartments and rooms to treehouses and boats: stay in unique spaces in 192 countries.</p>
			<p><a href="#">See how to travel on Airbnb</a></p>
		  </div>
		  <div>
			<h3>Host</h3>
			<p>Renting out your unused space could pay your bills or fund your next vacation.</p>
			<p><a href="#">Learn more about hosting</a></p>
	      </div>
		  <div>
			<h3>Trust and Safety</h3>
			<p>From Verified ID to our worldwide customer support team, we've got your back.</p>
			<p><a href="#">Learn about trust at Airbnb</a></p>
		  </div>
	    </div>
	  </div>
	</div>
  </body>
</html>
{% endhighlight %}

主要注意在div中设置class，目的是为了可以用CSS设置。

#### CSS

控制页面的style和layout

{% highlight css %}
.nav a {
  color: #5a5a5a;
  font-size: 11px;
  font-weight: bold;
  padding: 14px 10px;
  text-transform: uppercase;
}

.nav li {
  display: inline;
}

.jumbotron {
  background-image:url('http://goo.gl/04j7Nn');
  height: 500px;
}

.jumbotron .container {
  position: relative;
  top: 100px;
}

.jumbotron h1 {
  color: #fff;
  font-size: 48px;  
  font-family: 'Shift', sans-serif;
  font-weight: bold;
}

.jumbotron p {
  font-size: 20px;
  color: #fff;
}

.learn-more {
  background-color: #f7f7f7;
}

.learn-more h3 {
  font-family: 'Shift', sans-serif;
  font-size: 18px;
  font-weight: bold;
}

.learn-more a {
  color: #00b0ff;
}
{% endhighlight %}

主要注意选择器的使用。

#### bootstrap

可以理解为是一个CSS框架和工具集。

+ **Grid**

内置有12个等宽的grid列。

![grid](http://s3.amazonaws.com/codecademy-content/courses/ltp/img/grid-0.png "grid")

{% highlight html %}
<div class="row">

  <div class="col-md-6">
    <h4>Content 1</h4>
  </div>

  <div class="col-md-6">
    <h4>Content 2</h4>
  </div>

</div>
{% endhighlight %}

其中`.col-md-6`表示占了12列中的6列，`row`表示这两个分别占6列的列是水平排列的。

+ **Tab**

{% highlight html %}
<ul class="nav nav-tabs">
	<li><a href="#">Primary</a></li>
	<li class="active"><a href="#">Social</a></li>
	<li><a href="#">Promotions</a></li>
	<li><a href="#">Updates</a></li>
</ul>
{% endhighlight %}

只需要为`ul`添加`class="nav nav-tabs"`，一秒变tab。其中`class="active"`的为默认选中的tab项。

+ **Pill**

用于为页面添加一些按钮链接。

{% highlight html %}
<ul class="nav nav-pills">
	<li><a href="#">Primary</a></li>
	<li class="active"><a href="#">Social</a></li>
	<li><a href="#">Promotions</a></li>
	<li><a href="#">Updates</a></li>
</ul>
{% endhighlight %}

只需要为`ul`添加`class="nav nav-pills"`，一秒变pill。其中`class="active"`的为默认选中的pill项。

+ *Jumbotron*

用于展示重要信息。

{% highlight html %}
<div class="jumbotron">
	<h1>Find a place to stay.</h1>
	<p>Rent from people in over 34,000 cities and 192 countries.</p>
</div>
{% endhighlight %}

只需要为`div`添加`class="jumbotron"`，就可以发现这个`div`区域的样式更加醒目了。

+ **More**

更多的bootstrap特性可以看<http://getbootstrap.com/css/>



