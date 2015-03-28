---
layout: post
title: 从HybridApp技术调研扩展
category: 前端技术
tags: [Hybrid,codova,ionic,jquery mobile,前端MVC,Angular]
---

### 三大类App比较

#### native app

native app 是调用了系统自带的的api开发而成，所以不同的系统就有不同的语言进行开发，如android 是采用了java语言，ios是使用了oc，所以要是开发同一款应用想在android和ios的机上运行还真不容易，要找来会java和oc的程序员，而且在同一系统上开发出来的程序，还要区分不同的版本，这就是为什么native app的成本高的原因了。但是native app运行的速度和用户的体验是最好的，所以为什么native app这么重要就是这个原因，用户至上，钱算什么。还有native app在上各大应用商店的审核还是比较好通过的。

#### web app

web app 它就是我们平常看到的网页，是由hmtl css javascript 组成，现在称之为html5技术，最近也是相当的火爆，连微软这样固执的老头在ie9也开始支持html5技术，现在的大头都很重视html5技术，也积极的发展。由html5做出来的app在在效果上也是相当的漂亮，它的显示效果可以做到和native app的一样，但是体验是比较差的，因为它不能和各个系统平台进行通信，因此操作就比较少了，实现的功能也是很少的，下载的流量比较大，只能作信息的展示和传递，但是它有一个很明显的优点就是一次性开发，有支持html5的浏览器的平台都能使用，所以制作的费用和技术比较低，如果你是 用来展示信息的话倒可以使用web app，而且现在android和ios两大系统都支持将web app的快捷方式添加到桌面，所以现在web app的入口相当方便，不用时时都要输入很长的域名地址。在ios中你一旦添加了快捷方式后，从桌面中打开safari的url bar 和bottom都会隐藏掉，只要加入相应的html代码，就完全类似native app，效果非常好。或许在不久的将来，android和ios都会开放给html5就直接操纵的相关api。现在apple已经在行动，还有web os的诞生，我们将期待html5横扫时代的到来。

#### hybrid app

hybrid app 其实就是一种折中的方法，通过用native api 开发出一些接口让js调用就使到他们完美的结合，使到html5运行在一个虚拟的浏览器中，而且还可以操纵相关的设备。如今各种开发框架应运而生，如现在被apache收录的phonegap，版本更新的如火箭般快，发展势头很猛，它们的产生使到我们找到了一个跨平台的开发技术，开发人员只要会html5技术就完全能胜任开发工作，相当的完美，但是hybrid app还是有很多的缺点，如它运行的速度真的不如native app，界面的各种效果有时候很虚，确实达不到一个完美的app，最痛恨的是apple是不会让你的app通过审核，也就是说hybrid app苹果是不认可的，你将不能在apple app store中见到你的app，他会建议你用oc开发app，无解，除非你能骗过审核人员。android app store就比较容易通过。你在使用这种技术去开发app的时候就要注意了，这是经验所得。

### codova/phonegap 

#### 流程：

1. install codova shell tool
android sdk环境变量

    export PATH=${PATH}:/Development/adt-bundle/sdk/platform-tools:/Development/adt-bundle/sdk/tools
    $ source ~/.bash_profile
 
2. create app & add platform

    $ cordova create hello com.example.hello HelloWorld
    $ cd hello
    $ cordova platform add android	//另外也需要cordova platform add iOS
    $ cordova build	//这个会把所有platform的都build
    $ cordova platforms ls

然后导入eclipse，注意一点：build得到的hello/platform/android，在import的时候不要导入到workspace。

3. build project 
在hello/www中

    $ cordova build

4. configure a emulator

5. deploy to emulator || deploy to device

6. 在hello/platform/ios中双击helloworld.xcodeproj，它也一样的可以deploy to emulator。

#### 系统原理

<http://app.phonegap.com>

这个把整个系统介绍的很清楚。其实说的意思就是，可以在服务器统一修改，然后各个平台的app都统一修改了。
我理解：图中的desktop app指的是web后台，mobile app指的就是app壳。

#### 技术架构

phonegap其实就是可以理解为一个中间件.
用啥前端框架都行，cordova只是个壳
前端框架随便选，ionic,mui,jquery mobile
比如，写一个移动站点，调用几个什么值得买的ajax api，潜到cordova。

### ionic

ionic中连模拟器都可以在shell中实现，太爽了。

$ ionic start myApp tabs	//blank or sidemenu
$ cd myApp
$ ionic platform add ios
$ cordova plugin add org.apache.cordova.device

$ ionic build ios
$ ionic emulate ios

#### 技术架构

ionic是ui，angular是mvc，这两者加在一起就有了页面，然后用cordova加层壳成为app。

<http://ionicframework.com/docs/guide/starting.html>
这个例子，可以看出ionic+angular+cordova是如何结合工作的：ionic提供了一些控件，然后要想控件运转起来就需要angular，最后要发布就需要cordova。

### 实践

如果用ionic，有ionic creator提供component／template／export。
这里有个例子，一步步说明怎么使用：http://thejackalofjavascript.com/ionic-creator-beta/
这个例子非常清楚，可以很清晰看出怎么做前端＋后端系统。
后端是一个能提供rest api的服务器，然后前端是ionic/www（这里纠正一下原来的一个错误认识，www中是前端系统，并不是server。），用来调用后端的rest服务。
这里有个angular写spa的例子，也可以帮助理解。

如果用jquery mobile，可以看看jquery mobile bootstrap template：http://www.sitepoint.com/jquery-mobile-bootstraps/
另外也有themeroller可以自定义theme。

### Angular

Angular没有提供完善的UI，没有提供CSS样式套件，也没有对移动平台进行直接支持。所以，如果你使用Angular，你一定需要其它东西来配合。例如，如果需要UI，你需要使用jQueryUI，或者自己封装UI组件；如果需要CSS样式，你可以选择bootstrap或者LESS；如果需要支持移动平台，还是需要你自己 去开发。

jQuery主要是用来操作DOM的，如果单单说jQuery的话就是这样一个功能，它的插件也比较多，大家也都各自专注一个功能，可以说jQuery体系是跟着前端页面从静态到动态崛起的一个产物，他的作用就是消除各浏览器的差异，简化和丰富DOM的API，简单易用。

而AngularJS, Ember.js, Backbone则是比较新的产物，他们的产生不是为了再页面上实现各种特效，而是为了构建更重量级的webapp，这种app通常只有一个页面，通常拥有丰富的数据和交互，业务逻辑耦合深，跟传统的web页面还是有比较大的差异的。他们通常把数据和逻辑还有展现之类的东西做了分离，可以更方便做出复杂的单页面应用。


### 选型

#### 初步

cordova肯定是确定了作为中间层。

对于开发框架方面：
ionic + angularjs
jquery mobile

问了一下meikidd，taobao的做法是：
前端用自己的框架，app是web+native，一些活动页面用web，并不是全部用web。因为转场／瀑布流这样的效果web很难模拟出native的效果，对于他们的app来说，难点在于html5和native之间的交互协议。
对于我这种全web然后套层壳的app，中间层用cordova。对于前端他的意思是用jquery mobile蛮好用，不过缺点在于它的ui不好看，如果想效果好可能需要设计师重新设计一套控件样式。ionic没有用过，不过angular有点重，而且对代码侵入性比较大，如果未来要换框架的话可能只能重新写了，而jquery mobile就扩展性好，很多新的框架都会比较支持它。不过如果搞单页应用的话，ionic会蛮爽，因为如果是单页应用，最好是选择支持路由／视图／转场这些特性的。如果是所有场景都在一个页面里面，只有一个webview，那还是选angular，因为它可以做很多事情。（native支持每开一个页面就新开一个webview，这样页面内逻辑就简单了，native需要处理的就多一点）
再就是虽然jquery mobile比较大，但是可以随着发布包打包到手机中安装，一般也不会更新，就算更新就更新安装包。（另外就是其实jquery mobile的库文件都是经过cdn的，应该蛮快。）

### 最终

fyi，技术选型确定下来了：讨论的结果是用jquery mobile来作为前端框架，将静态资源都发布在mobile端，而动态部分通过server端提供。另外ui的部分可以选用bootstrap等框架（如果做出来模版味太重，可以考虑再优化一下）。（虽然angular做单页应用有天然优势，不过毕竟上手没有jquery快而且感觉更适合做更大型的系统，对代码侵入性比较大如果未来想把angular换掉那基本上只能重新开发了，另外感觉ionic的document不是非常详细，ui也不太有优势，另外对于它宣称的性能优势这方面估计并不能对咱们应用带来多少实际好处）。

### 通过spring mvc理一下系统架构

spring mvc，把系统架构又理顺了很多：
先说后台系统：用于发布api
dao-service-api三层，在dao和service都是object，在api层中做的事情是：每个request根据uri进行mapping到一个function，然后进行gson builder来讲从service获取的object转化为json，这其中需要用到的方式是每个class type都对应一个serializer。
总的来说，后台系统和原始的ssh系统很像，只是不需要再使用struts了，而改用spring mvc，这样的好处是可以直接发布为json api，也就是rest的方式。摒弃了原来struts.xml那种比较土的方式。需要注意的是在后台系统中其实没有用到spring mvc中v的部分，而只用到的是uri映射以及返回结果封装为json这个功能，按照我的理解似乎还没有使用到真正mvc的功能，所以说这块说使用了spring mvc不如说使用了spring，因为确实使用的是spring，不过因为这是后台系统，并没有怎么用到mvc的功能，mvc按我理解应该是在后台提供数据之后前端来展示视图时才会用到。

再来说说前端系统：做的事情是调用rest api获取json数据，然后在页面中呈现出来。
这块如果拿spring mvc＋jsp来做的话，这样的：httpclient.get(uri)获取到json数据然后放入到model，这个model是处于spring mvc管辖内的m，然后jsp中就可以直接用点式语法获取model中的数据。

然后最外层就是浏览器，在微信公众号中也就是微信内置的浏览器。

需要注意的是，现在我需要做的事情是，后台已经会提供给我api，我需要做的就是前端展示这部分，也就是说我现在需要做的就是选一个mvc框架，用angular的话，那就是正好可以用到这个功能。

### 单页面应用

这个概念很像百度的ER框架

单页 Web 应用概述
传统的 Web 应用通过加载整个 Web 页面来实现与用户的交互。当用户点击一个链接或提交一个表单时，浏览器将向服务器请求一个全新的页面。这涉及到获取新页面的数据、卸载旧的页面和绘制新的页面。按照这种方式进行用户交互势必影响到 Web 应用的性能。并且，由于网络延时的存在，页面与页面之间的切换很可能会不流畅，从而进一步影响用户体验。
SPA 能够缓解传统的 Web 应用存在的问题。有关 SPA 的详细介绍可以访问 参考资源 以获得更多的信息。SPA 规定在一个页面的生命周期中不会有页面的重新加载，所有的用户交互和页面内容变换将限制在同一个 Web 页面中。换言之，SPA 用 Web 页面的局部变更来代替整个页面的重新加载，从而提高了用户体验和整体性能。
目前 SPA 已经有了较为广泛的应用，如 Google Docs、Gmail 等都是单页 Web 应用。
但是，相比于传统的 Web 应用，SPA 也存在着以下问题。
1. SPA 中动态变化的内容可能不会被搜索引擎找到。如果你的 Web 页面的内容希望被搜索引擎找到，那就不应该选择 SPA。
2. 需要特殊处理才能实现浏览器的“后退”和“前进”功能。
3. SPA 需要在一个 Web 页面周期中加载所有需要的 HTML、CSS 等资源。这样有可能造成初次加载时间较长。本文将在“在创建适用于单页 Web 应用的 UI 布局”中介绍缓解该问题的方法。
利用上述的方法构建的二级菜单 Web 应用，可以实现在一个 Web 页面的生命周期内，动态的切换多个子页面。同时，上述实现方法还具有以下优点：
1. 延迟加载。即在初次加载应用时，只加载需要的部分。其他的部分只在用户需要的时候才去加载。这样可以减少初始加载的时间，提高用户访问速度。在上述的实现过程中，所有的子页面都会在“OnLoad”事件触发时向 Firebug 的 Console 面板打印信息。运行本文提供的示例代码可以发现，在初次加载时，仅仅 Header 和一个子页面被加载了。
2. 局部更新。即切换子页面时，仅仅是子页面内的内容变换了，其他区域（例如 Header）并没有重新加载。这样提高了 Web 应用的响应速度。

优点：
1、分离前后端关注点，前端负责界面显示，后端负责数据存储和计算，各司其职，不会把前后端的逻辑混杂在一起；
2、减轻服务器压力，服务器只用出数据就可以，不用管展示逻辑和页面合成，吞吐能力会提高几倍；
3、同一套后端程序代码，不用修改就可以用于Web界面、手机、平板等多种客户端；
缺点：
1、SEO问题，现在可以通过Prerender等技术解决一部分；
2、前进、后退、地址栏等，需要程序进行管理；
3、书签，需要程序来提供支持；

### 前端MVC

几年前，jQuery 是用于构建客户端 JavaScript 应用程序的主流库。然而，随着应用程序中的 JavaScript 的复杂性日益增加，jQuery 成为一项处理复杂性的必要不充分技术。例如，用于待办事项 (to-do) 列表的单页面应用程序可以包含一个紧急待办事项列表、一个完整的待办事项列表、一个当日待办事项列表，以及一个过期待办事项列表。在删除某个待办事项时会怎样？如果任务很紧急但已过期，您可能需要手动编写代码来从视图中的三个或四个不同位置中删除该事项。如果删除某个对象后需要您删除或更改屏幕上显示的其他相关对象，这样复杂性就会变得无法控制。客户端 MVC 框架旨在解决此类问题。

你还没有搞清楚前端路由和服务端路由的区别：
服务端路由：每跳转到不同的URL，都是重新访问服务端，然后服务端返回页面，页面也可以是服务端获取数据，然后和模板组合，返回HTML，也可以是直接返回模板HTML，然后由前端JS再去请求数据，使用前端模板和数据进行组合，生成想要的HTML。
前端路由：每跳转到不同的URL都是使用前端的锚点路由，实际上只是JS根据URL来操作DOM元素，根据每个页面需要的去服务端请求数据，返回数据后和模板进行组合，当然模板有可能是请求服务端返回的，这就是  SPA 单页程序.



下面是我的理解：

传统的ssh是后端struts提供route，因为是后端来渲染页面，将页面渲染了之后返回给前端，也就是说其实jsp文件是放在后台的，完成了model组装之后再给前端，这样前端其实没做很多事情。

但是spa单页应用，也就是baidu union做的方式是：前端进行route，调用不同的后端api。后端需要做的事情只是提供json数据即可，渲染页面的事情都是由前端来完成的。



服务器端的MVC，每次用户干了什么，流程大致是这样的：

客户端发送请求 -> 服务器触发Controller -> 服务器进行Model各种操作 -> 服务器根据Model数据渲染View -> 服务器回复请求，包含了整个View的html -> 客户端重新渲染整个页面，所有的css都又计算了一遍，所有的js都重新执行了一遍，所有的资源都重新请求了一遍（虽然可能已经cache了）

前端MVC的话则是这样的（前端其实大部分是MV+X，不一定有Controller):

客户端根据用户的行为修改客户端Model -> 客户端更新和该Model相关的View -> 客户端Model发送sync请求到服务器，只包含改变了哪些数据 -> 服务器审核数据改动是否合法，只需回复是否修改成功 -> 客户端收到成功，什么都不用做；不成功，则把刚才改的View改回来，然后通知用户。（当然，也可以在客户端的Model里面也加入审核机制，这样对不合法数据的反应更快，但还是得保留服务器端的审核）

比较一下可以看到，前端MVC需要向服务器端传递和接收的数据量都少很多，而前端要做的等待和渲染工作也少了很多。换言之，会提供更快的交互反馈，也意味着更好的用户体验。同时，服务器端的负载也略有降低，因为基本上只需要在数据库上提供一个RESTful API即可。

-EOF-
