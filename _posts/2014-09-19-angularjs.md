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

{% highlight html %}
Search: <input ng-model="query">
Sort by:
<select ng-model="orderProp">
  <option value="name">Alphabetical</option>
  <option value="age">Newest</option>
</select>


<ul class="phones">
  <li ng-repeat="phone in phones | filter:query | orderBy:orderProp">
    {{phone.name}}
    <p>{{phone.snippet}}</p>
  </li>
</ul>
{% endhighlight %}

增加了一个叫做orderProp的<select>标签，这样我们的用户就可以选择我们提供的两种排序方法。

然后，在filter过滤器后面添加一个orderBy过滤器用其来处理进入迭代器的数据。orderBy过滤器以一个数组作为输入，复制一份副本，然后把副本重排序再输出到迭代器。
AngularJS在select元素和orderProp模型之间创建了一个双向绑定。而后，orderProp会被用作orderBy过滤器的输入。

正如我们在讨论数据绑定和迭代器的时候所说的一样，无论什么时候数据模型发生了改变（比如用户在下拉菜单中选了不同的顺序），AngularJS的数据绑定会让视图自动更新。没有任何笨拙的DOM操作！

还需要修改一下数据模型：

{% highlight javascript %}
function PhoneListCtrl($scope) {
  $scope.phones = [
    {"name": "Nexus S",
     "snippet": "Fast just got faster with Nexus S.",
     "age": 0},
    {"name": "Motorola XOOM™ with Wi-Fi",
     "snippet": "The Next, Next Generation tablet.",
     "age": 1},
    {"name": "MOTOROLA XOOM™",
     "snippet": "The Next, Next Generation tablet.",
     "age": 2}
  ];

  $scope.orderProp = 'age';
}
{% endhighlight %}

我们修改了`phones`模型—— 手机的数组 ——为每一个手机记录其增加了一个`age`属性。我们会根据`age`属性来对手机进行排序。
我们在控制器代码里加了一行让`orderProp`的默认值为`age`。如果我们不设置默认值，这个模型会在我们的用户在下拉菜单选择一个顺序之前一直处于未初始化状态。

> 现在我们该好好谈谈双向数据绑定了。注意到当应用在浏览器中加载时，“Newest”在下拉菜单中被选中。这是因为我们在控制器中把`orderProp`设置成了‘age’。所以绑定在从我们模型到用户界面的方向上起作用——即数据从模型到视图的绑定。
现在当你在下拉菜单中选择“Alphabetically”，数据模型会被同时更新，并且手机列表数组会被重新排序。这个时候数据绑定从另一个方向产生了作用——即数据从视图到模型的绑定。（注意这块的“模型”指的就是`phones`，而不是指的js文件中的代码）

### 依赖注入

数据为json提供：`app/phones/phones.json`文件是一个数据集，它以JSON格式存储了一张更大的手机列表。

下面是这个文件的一个样例：
{% highlight json %}
[
 {
  "age": 13,
  "id": "motorola-defy-with-motoblur",
  "name": "Motorola DEFY\u2122 with MOTOBLUR\u2122",
  "snippet": "Are you ready for everything life throws your way?"
  ...
 },
...
]
{% endhighlight %}

我们在控制器中使用AngularJS服务`$http`向你的Web服务器发起一个HTTP请求，以此从`app/phones/phones.json`文件中获取数据。$http仅仅是AngularJS众多内建服务中之一，这些服务可以处理一些Web应用的通用操作。AngularJS能将这些服务注入到任何你需要它们的地方。

> 服务是通过AngularJS的依赖注入DI子系统来管理的。依赖注入服务可以使你的Web应用良好构建（比如分离表现层、数据和控制三者的部件）并且松耦合（一个部件自己不需要解决部件之间的依赖问题，它们都被DI子系统所处理）。

{% highlight javascript %}
function PhoneListCtrl($scope, $http) {
  $http.get('phones/phones.json').success(function(data) {
    $scope.phones = data;
  });

  $scope.orderProp = 'age';
}

//PhoneListCtrl.$inject = ['$scope', '$http'];
{% endhighlight %}

`$http`向Web服务器发起一个HTTP GET请求，索取`phone/phones.json`（注意，url是相对于我们的index.html文件的）。服务器用json文件中的数据作为响应。

`$http`服务用`success`返回。当异步响应到达时，用这个对象应答函数来处理服务器响应的数据，并且把数据赋值给作用域的`phones`数据模型。

#### JS压缩

由于AngularJS是通过控制器构造函数的参数名字来推断依赖服务名称的。所以如果你要压缩`PhoneListCtrl`控制器的JS代码，它所有的参数也同时会被压缩，这时候依赖注入系统就不能正确的识别出服务了。

为了克服压缩引起的问题，只要在控制器函数里面给`$inject`属性赋值一个依赖服务标识符的数组，就像被注释掉那段最后一行那样：

    PhoneListCtrl.$inject = ['$scope', '$http'];

### 链接与图片模板

数据：现在`app/phones/phones.json`文件包含了唯一标识符和每一部手机的图像链接。这些url现在指向`app/img/phones/`目录。

{% highlight json %}

[
 {
  ...
  "id": "motorola-defy-with-motoblur",
  "imageUrl": "img/phones/motorola-defy-with-motoblur.0.jpg",
  "name": "Motorola DEFY\u2122 with MOTOBLUR\u2122",
  ...
 },
...
]
{% endhighlight %}

模板`app/index.html`

{% highlight html %}
...
        <ul class="phones">
          <li ng-repeat="phone in phones | filter:query | orderBy:orderProp" class="thumbnail">
            <a href="#/phones/{{phone.id}}" class="thumb"><img ng-src="{{phone.imageUrl}}"></a>
            <a href="#/phones/{{phone.id}}">{{phone.name}}</a>
            <p>{{phone.snippet}}</p>
          </li>
        </ul>
...
{% endhighlight %}

这些链接将来会指向每一部电话的详细信息页。不过现在为了产生这些链接，我们在`href`属性里面使用我们早已熟悉的双括号数据绑定。

我们同样为每条记录添加手机图片，只需要使用`ng-src`指令代替`<img>`的`src`属性标签就可以了。如果我们仅仅用一个正常`src`属性来进行绑定（`<img class="diagram" src="{{phone.imageUrl}}">`），浏览器会把AngularJS的`{{ 表达式 }}`标记直接进行字面解释，并且发起一个向非法url`http://localhost:8000/app/{{phone.imageUrl}}`的请求。因为浏览器载入页面时，同时也会请求载入图片，**AngularJS在页面载入完毕时才开始编译**——浏览器请求载入图片时`{{phone.imageUrl}}`还没得到编译！有了这个`ng-src`指令会避免产生这种情况，使用`ng-src`指令防止浏览器产生一个指向非法地址的请求。

### 路由和多视图

在完成这步之后，当你转到app/index.html时，你会被重定向到app/index.html#/phones并且相同的手机列表在浏览器中显示了出来。当你点击一个手机链接时，一个手机详细信息列表也被显示了出来。

在这步之前，应用只给我们的用户提供了一个简单的界面（一张所有手机的列表），并且所有的模板代码位于`index.html`文件中。下一步是增加一个能够显示我们列表中每一部手机详细信息的页面。

为了增加详细信息视图，我们可以拓展`index.html`来同时包含两个视图的模板代码，但是这样会很快给我们带来巨大的麻烦。相反，我们要把`index.html`模板转变成**“布局模板”**。这是我们应用所有视图的通用模板。其他的“局部布局模板”随后根据当前的“路由”被充填入，从而形成一个完整视图展示给用户。

AngularJS中应用的路由通过`$routeProvider`来声明，它是`$route`服务的提供者。这项服务使得控制器、视图模板与当前浏览器的URL可以轻易集成。应用这个特性我们就可以实现深链接，它允许我们使用浏览器的历史(回退或者前进导航)和书签。

#### 关于依赖注入（DI），注入器（Injector）和服务提供者（Providers）

正如从前面你学到的，依赖注入是AngularJS的核心特性，所以你必须要知道一点这家伙是怎么工作的。

当应用引导时，AngularJS会创建一个注入器，我们应用后面所有依赖注入的服务都会需要它。这个注入器自己并不知道`$http`和`$route`是干什么的，实际上除非它在模块定义的时候被配置过，否则它根本都不知道这些服务的存在。注入器唯一的职责是载入指定的服务模块，在这些模块中注册所有定义的服务提供者，并且当需要时给一个指定的函数注入依赖（服务）。这些依赖通过它们的提供者“懒惰式”（需要时才加载）实例化。

提供者是提供（创建）服务实例并且对外提供API接口的对象，它可以被用来控制一个服务的创建和运行时行为。对于`$route`服务来说，`$routeProvider`对外提供了API接口，通过API接口允许你为你的应用定义路由规则。

    我理解angular的依赖注入的含义就是，把一些服务通过简单的API注入到应用中来。这个确实挺强大的。

`app/js/app.js`
{% highlight javascript %}
angular.module('phonecat', []).
  config(['$routeProvider', function($routeProvider) {
  $routeProvider.
      when('/phones', {templateUrl: 'partials/phone-list.html',   controller: PhoneListCtrl}).
      when('/phones/:phoneId', {templateUrl: 'partials/phone-detail.html', controller: PhoneDetailCtrl}).
      otherwise({redirectTo: '/phones'});
}]);
{% endhighlight %}

为了给我们的应用配置路由，我们需要给应用创建一个模块。我们管这个模块叫做`phonecat`，并且通过使用`configAPI`，我们请求把`$routeProvider`注入到我们的配置函数并且使用`$routeProvider.whenAPI`来定义我们的路由规则。

注意到在注入器配置阶段，提供者也可以同时被注入，但是一旦注入器被创建并且开始创建服务实例的时候，他们就不再会被外界所获取到。

我们的路由规则定义如下

> 当URL 映射段为`/phones`时，手机列表视图会被显示出来。为了构造这个视图，AngularJS会使用`phone-list.html`模板和`PhoneListCtrl`控制器。
当URL 映射段为`/phone/:phoneId`时，手机详细信息视图被显示出来。这里`:phoneId`是URL的变量部分。为了构造手机详细视图，AngularJS会使用`phone-detail.html`模板和`PhoneDetailCtrl`控制器。
`$route.otherwise({redirectTo: '/phones'})`语句使得当浏览器地址不能匹配我们任何一个路由规则时，触发重定向到`/phones`。

注意到在第二条路由声明中`:phoneId`参数的使用。`$route`服务使用路由声明`/phones/:phoneId`作为一个匹配当前URL的模板。所有以:符号声明的变量（此处变量为phones）都会被提取，然后存放在`$routeParams`对象中。

为了让我们的应用引导我们新创建的模块，我们同时需要在`app/index.html`的`ngApp`指令的值上指明模块的名字：

{% highlight html %}

<!doctype html>
<html lang="en" ng-app="phonecat">
...
{% endhighlight %}

控制器`app/js/controllers.js`

{% highlight javascript %}
...
function PhoneDetailCtrl($scope, $routeParams) {
  $scope.phoneId = $routeParams.phoneId;
}

//PhoneDetailCtrl.$inject = ['$scope', '$routeParams'];
{% endghighlight %}

`$route`服务通常和`ngView`指令一起使用。`ngView`指令的角色是为当前路由把对应的视图模板载入到布局模板中。

`app/index.html`中：

{% highlight html %}
<html lang="en" ng-app="phonecat">
<head>
...
  <script src="lib/angular/angular.js"></script>
  <script src="js/app.js"></script>
  <script src="js/controllers.js"></script>
</head>
<body>

  <div ng-view></div>

</body>
</html>
{% endhighlight %}

注意，我们把`index.html`模板里面大部分代码移除，我们只放置了一个`<div>`容器，这个`<div>`具有`ng-view`属性。我们删除掉的代码现在被放置在`phone-list.html`模板中：

{% highlight html %}
<div class="container-fluid">
  <div class="row-fluid">
    <div class="span2">
      <!--Sidebar content-->

      Search: <input ng-model="query">
      Sort by:
      <select ng-model="orderProp">
        <option value="name">Alphabetical</option>
        <option value="age">Newest</option>
      </select>

    </div>
    <div class="span10">
      <!--Body content-->

      <ul class="phones">
        <li ng-repeat="phone in phones | filter:query | orderBy:orderProp" class="thumbnail">
          <a href="#/phones/{{phone.id}}" class="thumb"><img ng-src="{{phone.imageUrl}}"></a>
          <a href="#/phones/{{phone.id}}">{{phone.name}}</a>
          <p>{{phone.snippet}}</p>
        </li>
      </ul>

    </div>
  </div>
</div>
{% endhighlight %}

注意到我们的布局模板中没再添加`PhoneListCtrl`或`PhoneDetailCtrl`控制器属性！

接下来，将实现手机详细信息视图，我们将会使用`$http`来获取数据，同时我们也要增添一个`phone-detail.html`视图模板。
这里相当于是给出了写一个页面的整体过程，所以很有参考价值：

+ 数据

`app/phones/`目录包含了每一部手机信息的json文件。

比如`app/phones/nexus-s.json`（样例片段）:

{% highlight json %}
{
  "additionalFeatures": "Contour Display, Near Field Communications (NFC),...",
  "android": {
      "os": "Android 2.3",
      "ui": "Android"
  },
  ...
  "images": [
      "img/phones/nexus-s.0.jpg",
      "img/phones/nexus-s.1.jpg",
      "img/phones/nexus-s.2.jpg",
      "img/phones/nexus-s.3.jpg"
  ],
  "storage": {
      "flash": "16384MB",
      "ram": "512MB"
  }
}
{% endhighlight %}

这些文件中的每一个都用相同的数据结构描述了一部手机的不同属性。我们会在手机详细信息视图中显示这些数据。

+ 控制器

我们使用`$http`服务获取数据。这和之前的手机列表控制器的工作方式是一样的。

`app/js/controllers.js`中：

{% highlight javascript %}
function PhoneDetailCtrl($scope, $routeParams, $http) {
  $http.get('phones/' + $routeParams.phoneId + '.json').success(function(data) {
    $scope.phone = data;
  });
}

//PhoneDetailCtrl.$inject = ['$scope', '$routeParams', '$http'];
{% endhighlight %}

为了构造HTTP请求的URL，我们需要`$route`服务提供的当前路由中抽取`$routeParams.phoneId`。

+ 模板

`phone-detail.html`文件，这里我们使用AngularJS的`{{表达式}}`标记和`ngRepeat`来在视图中渲染数据模型。

{% highlight html %}
<img ng-src="{{phone.images[0]}}" class="phone">

<h1>{{phone.name}}</h1>

<p>{{phone.description}}</p>

<ul class="phone-thumbs">
  <li ng-repeat="img in phone.images">
    <img ng-src="{{img}}">
  </li>
</ul>

<ul class="specs">
  <li>
    <span>Availability and Networks</span>
    <dl>
      <dt>Availability</dt>
      <dd ng-repeat="availability in phone.availability">{{availability}}</dd>
    </dl>
  </li>
    ...
  </li>
    <span>Additional Features</span>
    <dd>{{phone.additionalFeatures}}</dd>
  </li>
</ul>
{% endhighlight %}

### 过滤器


-EOF-
