---
layout: post
title: 分布式系统架构笔记
category: 分布式
tags: [分布式]
---

源于曾经的一个技术调研，笔记比较杂，但是很有料：

（1）请求分发：
BVS（baidu virtual server）
类似于UTR
都是做负载均衡、抗攻击这种功能的。

（2）两种模型：
同步模型
消息模型（异步）（队列）

（3）RPC属于一种传输方式，MCPACK属于一种协议。
展开来讲：
RPC关注的是分布式的组件通信。
编码方式有三种：JSON、XML、MCPACK
传输协议有两种：基于TCP之上的NShead+MCPACK，服务划分通过端口。基于HTTP之上的Hession，服务划分通过URL。
服务描述方式有两种：java的接口，IDL

（4）在union系统中的RPC
以holmes为例，有两种接口：
一种是要访问holems的web接口在XML中配server的地址，咱们这边属于client提供interface。
这种是注册为McpackRpcProxyFactoryBean。
第二种是要访问holmes数据接口，咱们这边属于client先将请求参数包装为RequestPackect，然后转为JSON，然后封装为MCPACK（这种协议可以理解为就是将JSON数据转为的key-value那种面相对象的数据结构）。然后通过socket发送，接收到返回信息后再按协议解析ResponsePacket。
另外，union也会作为server去提供RPC服务：
在咱们这作为innerapi，会写bean、service、impl。

（5）RMI是针对TCP/IP，spring Remoting是针对HTTP

（6）web service
SOA
SOAP

（7）REST

（8）架构设计：
组件compent，提供两种接口，web service的XML、REST的JSON
REST的url就可以对应到调用哪个interface。

（9）发布的方式：
组件上层有个web系统，处理权限等，组件和web层之间通过REST通信，HTTP方式。web层向外提供服务。

（10）多服务的架构下：
现状是，各台机器必须部署全部服务，然后上面都连着BVS配置这白名单各种IP。
希望是，各台机器可以部署部分服务，这些服务在注册中心进行注册。

（11）代码的架构：
下层是doris接口
接下来是report的impl，分为abstract和派生类
再上层是interface，REST风格的，比如get()，这里的各种params，参数是配在XML中的，比如type、columns、sum、cid、time、doristable等，以及派生类的class要注册。

DAO->ServiceImpl->service->interface
REST大概是union/report/cpro/getOne或getList

（12）MDD模型驱动开发：
现在有各种业务都有效果报告，而且很类似。
先实现cpro的比如cproServiceImpl，利用工具生成模板，然后比如有generateDao，里面有共用的方法，那么uapDao就可以调用这个共用方法，当然也可以手写。

（13）各种概念做一个对比和理解：
RPC——remote procedure call，是网络协议，语言无关，返回的是数据
RMI——remote method invocation，远程方法调用，Java相关，是EJB的基础，返回的是对象，传输基于的是java的序列化。
比如java.rmi.Naming，
server是Naming.rebind(接口方法)
client是Naming.lookup(接口方法)
RPC和RMI都是进行服务调用的，请求响应这种模式，但是本身不是特别有可比性，因为RPC更偏重一种网络协议，有种比较泛的感觉，RMI就比较具体是java的远程方法调用。
另外spring提供了RMI服务，当然也提供了HTTP的remoting服务，spring是值得信赖的。

JMS——java message service，java相关，作用是将对象从一个JVM转移到另一个JVM，是做信息交换的。
这个和RMI完全不是一码事，RMI是做方法调用的。

SOAP——simple object access protocol，语言无关，XML数据。

两边都是Java的时候，才能使用RMI。
如果两边是异构的，那么只能用web service。

SOA——service oriented architecture，面向服务架构（是以前面向技术架构的进化），语言无关、平台无关。
SOA其实是一种方法论：深入理解业务流程（这是关键）->组件建模（跨技术、面向服务、复用性强、有弹性、消息机制互连、高内聚低耦合）->服务整合
SOA可以基于的底层技术有CORBA（过时）、web service

SOAP是协议，在客户端和web服务端之间通过XML进行传递。

web service是可互操作的分布式应用平台，在异构双方进行HTTP互连。
所使用的远程过程调用协议是RPC
（我的理解：可以看出web service是一种SOA架构的底层互连模型，然后所使用的远程调用方式是RPC，这些概念是交织在一起但是又不是一码事）

SOA构架中的服务调用，大概是这么一回事：
使用WSDL——web service definition language，web服务描述语言，来描述服务，这是基于XML的语言，是由原来的IDL接口描述语言进化来的。
另外用XSD——XML schema definition来做消息通信
使用UDDI来注册和查找服务。
SOAP来作为SOA架构中的传输层，负责服务的调用。
大概的模式是：先通过UDDI查找服务，然后获取WSDL服务描述，然后通过SOAP调用服务。

SOAP可以理解为：HTTP+XML，因此是语言无关的。
由四部分组成：
SOAP封装：消息内容
SOAP编码规则：数据类型
SOAP-RPC：远程调用和应答的协定（这其实就是RPC的概念）
SOAP绑定：底层协议交换

web service的组成：
XML，因此是平台无关的
SOAP，利用HTTP的get/post方式来进行远程调用
服务注册与发现通过UDDI、WSDL这些
（其实看出来这和SOA架构很像，其实web service本来就是作为SOA架构的底层技术）


（14）Spring提供的远程访问和web服务
http://oss.org.cn/ossdocs/framework/spring/zh-cn/remoting.html
主要就是spring那本书里介绍的。

（15）Java远程通讯可选技术及原理 
这有个帖子总结的挺好的http://blog.csdn.net/xuhaipeng/article/details/3983917
这帖子下面的参考文档也很有意义。
先介绍了各种远程通信的应用层协议：传输协议则通常都是可选的，在java领域中知名的有：RMI、XML-RPC、Binary-RPC、SOAP、CORBA、JMS。
然后介绍各种可选实现技术：目前java领域可用于实现远程通讯的框架或library，知名的有：JBoss-Remoting、Spring-Remoting、Hessian、Burlap、XFire(Axis)、ActiveMQ、Mina、Mule、EJB3等等
关于SOAP、RPC和RMI的选型，还有个帖子很好http://blog.csdn.net/xuhaipeng/article/details/3984260

（16）SCA（Service Component Architecture）
http://www.ibm.com/developerworks/cn/webservices/ws-sca/

（17）MDD的一些记录：
业务模型：业务上的抽象

技术模型：
平台无关：接口
平台相关：SSH
代码

实体模型：
DB->DAO->SERVICE->ACTION->JS/JSP
		->API
现有的hibernate有局限，只有ORM。但是各个字段的从前到后的检验以及DB中的约束等等这些东西，又是需要通过js、struts、SQL这些东西来弄。
但是现在想实现的是，写了实体的XML之后（或者通过annotation来实现），可以把以上这些东西都生成出来。

模板：（泛型也可以实现）
柳桐（写模板，通过模板来实现java）（不想写模板怎么办，先写UserDao、UserService这些，然后用工具转化为模板，然后通过这个模板来实现CnDao、CnService这些）（还有就是这个生成出来的是generateService和generateService，可以在手写的类中调用这个自动生成的方法）
博哥（一套代码，动态解析，动态模板，XML是多个，但是模板这有一个，代码不生成）

协议：
MCPACK对内开发，公司内做了压缩，比较高效
web service在某些部门用，这个比较重，数据量太大了
rest用的是json
这些其实都是协议，像mcpack这种是内部协议。

（18）REST
相对于SOAP、XML-RPC来说要简洁一些。
REST基于HTTP、URI
资源定位通过URI
资源操作GET/POST/PUT/DELETE正好对应于HTTP
连接协议是无状态的。

RESTful是采用REST风格的web服务：
数据格式是JSON、XML
基于HTTP协议。

CRUD (Create, Read, Update, Delete)分别对应POST、GET、PUT、DELETE

传统的请求模式和REST模式的请求模式区别：
作用 			传统模式 			REST模式
列举出所有的用户 	GET /users/list 		GET /users
列出ID为1的用户信息 	GET /users/show/id/1 		GET /users/1
插入一个新的用户 	POST /users/add 		POST /users
更新ID为1的用户信息 	POST /users/mdy/id/1 		PUT /users/1
删除ID为1的用户 	POST /users/delete/id/1 	DELETE /users/1

（19）百度商业产品体系服务型组件框架设计与模型驱动开发实践

（20）监控日志
db64的路径：
/home/work/monitor/data/error/2013-09-04
/home/work/monitor/data/performance/2013-09-04

（21）性能优化会议补充：
1.缓存使用量
2.物化视图
3.快捷报告，抓典型

（22）一个rest的实际例子，以及联系实际（这两个帖子很有启发意义）
http://www.ibm.com/developerworks/cn/webservices/0907_rest_soap/
http://www.ibm.com/developerworks/cn/web/wa-aj-multitier/
REST可以用的HTTP的四种方式，但是SOAP只用到POST这种方式
而且REST的接口更直观一些。

使用 RPC 样式架构构建的基于 SOAP 的 Web 服务成为实现 SOA 最常用的方法。RPC 样式的 Web 服务客户端将一个装满数据的信封（包括方法和参数信息）通过 HTTP 发送到服务器。服务器打开信封并使用传入参数执行指定的方法。方法的结果打包到一个信封并作为响应发回客户端。客户端收到响应并打开信封。每个对象都有自己独特的方法以及仅公开一个 URI 的 RPC 样式 Web 服务，URI 表示单个端点。它忽略 HTTP 的大部分特性且仅支持 POST 方法。

使用RESTLET框架来做

现在有了一个大致的方案：
1. bowser client                    -> mvc            -> 业务逻辑层 -> DAO -> DB
2. webservice client rest           -> RESTlet        -> 业务逻辑层 -> DAO -> DB
3. webservice client spring remoting-> springremoting -> 业务逻辑层 -> DAO -> DB

（23）REST与SOAP适用场景
REST的适用场合包括：
1. 有限的带宽和资源 别忘了返回的结构可以采用（由开发者定义的）任何格式。另外，由于REST采用标准的GET、PUT、POST和DELETE动词，因此可被任何浏览器所支持。除此以外，REST还可以使用为目前大多数浏览器支持的XMLHttpRequest对象，这为AJAX增色不少。
2. 完全无状态的操作 对于那些需多步执行的操作，REST并非最佳选择，采用SOAP更合适。但是，如果你需要无状态的CRUD（Create/Read/Update/Delete，创建/读取/更新/删除）操作，那么应采用REST。
3. 缓存考虑 若要利用无状态操作的特性，使得信息可被缓存，那么REST是很好的选择。

SOAP的适用场合包括：
1. 异步处理与调用 如果你的应用需要确保可靠性与安全性，那么请采用SOAP。SOAP 1.2为确保这种操作补充定义了WSRM（WS-Reliable Messaging）等标准。
2. 形式化契约 若提供者/消费者双方必须就交换格式取得一致，那么采用SOAP更合适。SOAP 1.2为这种交互提供了严格的规范。
3. 有状态的操作 如果应用需要上下文信息与对话状态管理，那么应采用SOAP。SOAP 1.2为此补充定义了WS-Security、WS-Transactions和WS-Coordination等标准。相比之下，REST方法要求开发者自己来实现这些框架性工作。

（24）REST服务开发实战
http://www.infoq.com/cn/articles/dt-rest-service

REST和RPC之间的区别。
    REST强调资源有唯一的URI；而RPC更加强调过程（动词），由统一的接口来调用它们。
    REST回归HTTP最初的设计；RPC仅仅只是把HTTP作为传输协议来使用。
    REST是由超文本驱动的；RPC是由方法驱动的。
    REST强调HTTP通信的语义可见性，通过消息头和标准的HTTP方法来体现；RPC把语义封装在	HTTP消息体中。

REST最适合的应用场景是需要对外暴露服务的情况。此时，我们可以充分利用REST的自描述、无状态、唯一标识等特性来提供清晰、友好的API；此外，目前Jesery、RESTEasy等JAX-RS框架也提供了OAuth的支持，基本上能够保证服务安全。
最不适合的应用场景是对性能要求高的系统内部的服务调用，在这种情况下使用REST的话，那么REST所有的特性都会变成拖累。这个时候，还是需要选择更底层的通信协议和方式会更好一些，比如ICE。

利用RESTEasy搭建应用：
http://docs.jboss.org/resteasy/docs/2.0.0.GA/userguide/html_single/index.html

利用jersey搭建应用：
https://jersey.java.net/nonav/documentation/latest/getting-started.html


（25）Web开发技术的发展可以粗略划分成以下几个阶段：
1. 静态内容阶段：在这个最初的阶段，使用Web的主要是一些研究机构。Web由大量的静态HTML文档组成，其中大多是一些学术论文。Web服务器可以被看作是支持超文本的共享文件服务器。
2. CGI程序阶段：在这个阶段，Web服务器增加了一些编程API。通过这些API编写的应用程序，可以向客户端提供一些动态变化的内容。Web服务器与应用程序之间的通信，通过CGI（Common Gateway Interface）协议完成，应用程序被称作CGI程序。
3. 脚本语言阶段：在这个阶段，服务器端出现了ASP、PHP、JSP、ColdFusion等支持session的脚本语言技术，浏览器端出现了Java Applet、JavaScript等技术。使用这些技术，可以提供更加丰富的动态内容。
4. 瘦客户端应用阶段：在这个阶段，在服务器端出现了独立于Web服务器的应用服务器。同时出现了Web MVC开发模式，各种Web MVC开发框架逐渐流行，并且占据了统治地位。基于这些框架开发的Web应用，通常都是瘦客户端应用，因为它们是在服务器端生成全部的动态内容。
5. RIA应用阶段：在这个阶段，出现了多种RIA（Rich Internet Application）技术，大幅改善了Web应用的用户体验。应用最为广泛的RIA技术是DHTML+Ajax。Ajax技术支持在不刷新页面的情况下动态更新页面中的局部内容。同时诞生了大量的Web前端DHTML开发库，例如Prototype、Dojo、ExtJS、jQuery/jQuery UI等等，很多开发库都支持单页面应用（Single Page Application）的开发。其他的RIA技术还有Adobe公司的Flex、微软公司的Silverlight、Sun公司的JavaFX（现在为Oracle公司所有）等等。
6. 移动Web应用阶段：在这个阶段，出现了大量面向移动设备的Web应用开发技术。除了Android、iOS、Windows Phone等操作系统平台原生的开发技术之外，基于HTML5的开发技术也变得非常流行。

（26）REST架构风格最重要的架构约束有6个：
1. 客户-服务器（Client-Server）
通信只能由客户端单方面发起，表现为请求-响应的形式。
2. 无状态（Stateless）
通信的会话状态（Session State）应该全部由客户端负责维护。
3. 缓存（Cache）
响应内容可以在通信链的某处被缓存，以改善网络效率。
4. 统一接口（Uniform Interface）
通信链的组件之间通过统一的接口相互通信，以提高交互的可见性。
5. 分层系统（Layered System）
通过限制组件的行为（即，每个组件只能“看到”与其交互的紧邻层），将架构分解为若干等级的层。
6. 按需代码（Code-On-Demand，可选）
支持通过下载并执行一些代码（例如Java Applet、Flash或JavaScript），对客户端的功能进行扩展。

（27）理解REST的五个关键词：
    资源（Resource）
    资源的表述（Representation）
    状态转移（State Transfer）
    统一接口（Uniform Interface）
    超文本驱动（Hypertext Driven）

什么是资源？
资源是一种看待服务器的方式，即，将服务器看作是由很多离散的资源组成。每个资源是服务器上一个可命名的抽象概念。因为资源是一个抽象的概念，所以它不仅仅能代表服务器文件系统中的一个文件、数据库中的一张表等等具体的东西，可以将资源设计的要多抽象有多抽象，只要想象力允许而且客户端应用开发者能够理解。与面向对象设计类似，资源是以名词为核心来组织的，首先关注的是名词。一个资源可以由一个或多个URI来标识。URI既是资源的名称，也是资源在Web上的地址。对某个资源感兴趣的客户端应用，可以通过资源的URI与其进行交互。

什么是资源的表述？
资源的表述是一段对于资源在某个特定时刻的状态的描述。可以在客户端-服务器端之间转移（交换）。资源的表述可以有多种格式，例如HTML/XML/JSON/纯文本/图片/视频/音频等等。资源的表述格式可以通过协商机制来确定。请求-响应方向的表述通常使用不同的格式。

什么是状态转移？
状态转移（state transfer）与状态机中的状态迁移（state transition）的含义是不同的。状态转移说的是：在客户端和服务器端之间转移（transfer）代表资源状态的表述。通过转移和操作资源的表述，来间接实现操作资源的目的。

什么是统一接口？

REST要求，必须通过统一的接口来对资源执行各种操作。对于每个资源只能执行一组有限的操作。以HTTP/1.1协议为例，HTTP/1.1协议定义了一个操作资源的统一接口，主要包括以下内容：
    7个HTTP方法：GET/POST/PUT/DELETE/PATCH/HEAD/OPTIONS
    HTTP头信息（可自定义）
    HTTP响应状态代码（可自定义）
    一套标准的内容协商机制
    一套标准的缓存机制
    一套标准的客户端身份认证机制
REST还要求，对于资源执行的操作，其操作语义必须由HTTP消息体之前的部分完全表达，不能将操作语义封装在HTTP消息体内部。这样做是为了提高交互的可见性，以便于通信链的中间组件实现缓存、安全审计等等功能。

什么是超文本驱动？
“超文本驱动”又名“将超媒体作为应用状态的引擎”（Hypermedia As The Engine Of Application State，来自Fielding博士论文中的一句话，缩写为HATEOAS）。将Web应用看作是一个由很多状态（应用状态）组成的有限状态机。资源之间通过超链接相互关联，超链接既代表资源之间的关系，也代表可执行的状态迁移。在超媒体之中不仅仅包含数据，还包含了状态迁移的语义。以超媒体作为引擎，驱动Web应用的状态迁移。通过超媒体暴露出服务器所提供的资源，服务器提供了哪些资源是在运行时通过解析超媒体发现的，而不是事先定义的。从面向服务的角度看，超媒体定义了服务器所提供服务的协议。客户端应该依赖的是超媒体的状态迁移语义，而不应该对于是否存在某个URI或URI的某种特殊构造方式作出假设。一切都有可能变化，只有超媒体的状态迁移语义能够长期保持稳定。

一旦读者理解了上述REST的五个关键词，就很容易理解REST风格的架构所具有的6个的主要特征：
    面向资源（Resource Oriented）

    可寻址（Addressability）

    连通性（Connectedness）

    无状态（Statelessness）

    统一接口（Uniform Interface）

    超文本驱动（Hypertext Driven）

这6个特征是REST架构设计优秀程度的判断标准。其中，面向资源是REST最明显的特征，即，REST架构设计是以资源抽象为核心展开的。可寻址说的是：每一个资源在Web之上都有自己的地址。连通性说的是：应该尽量避免设计孤立的资源，除了设计资源本身，还需要设计资源之间的关联关系，并且通过超链接将资源关联起来。无状态、统一接口是REST的两种架构约束，超文本驱动是REST的一个关键词，在前面都已经解释过，就不再赘述了。

（28）从架构风格的抽象高度来看，常见的分布式应用架构风格有三种：
1    分布式对象（Distributed Objects，简称DO）
架构实例有CORBA/RMI/EJB/DCOM/.NET Remoting等等
2    远程过程调用（Remote Procedure Call，简称RPC）
架构实例有SOAP/XML-RPC/Hessian/Flash AMF/DWR等等
3    表述性状态转移（Representational State Transfer，简称REST）
架构实例有HTTP/WebDAV

DO和RPC这两种架构风格在企业应用中非常普遍，而REST则是Web应用的架构风格，它们之间有非常大的差别。

REST与DO的差别在于：
    REST支持抽象（即建模）的工具是资源，DO支持抽象的工具是对象。在不同的编程语言中，对象的定义有很大差别，所以DO风格的架构通常都是与某种编程语言绑定的。跨语言交互即使能实现，实现起来也会非常复杂。而REST中的资源，则完全中立于开发平台和编程语言，可以使用任何编程语言来实现。
    DO中没有统一接口的概念。不同的API，接口设计风格可以完全不同。DO也不支持操作语义对于中间组件的可见性。
    DO中没有使用超文本，响应的内容中只包含对象本身。REST使用了超文本，可以实现更大粒度的交互，交互的效率比DO更高。
    REST支持数据流和管道，DO不支持数据流和管道。
    DO风格通常会带来客户端与服务器端的紧耦合。在三种架构风格之中，DO风格的耦合度是最大的，而REST的风格耦合度是最小的。REST松耦合的源泉来自于统一接口+超文本驱动。

REST与RPC的差别在于：
    REST支持抽象的工具是资源，RPC支持抽象的工具是过程。REST风格的架构建模是以名词为核心的，RPC风格的架构建模是以动词为核心的。简单类比一下，REST是面向对象编程，RPC则是面向过程编程。
    RPC中没有统一接口的概念。不同的API，接口设计风格可以完全不同。RPC也不支持操作语义对于中间组件的可见性。
    RPC中没有使用超文本，响应的内容中只包含消息本身。REST使用了超文本，可以实现更大粒度的交互，交互的效率比RPC更高。
    REST支持数据流和管道，RPC不支持数据流和管道。
    因为使用了平台中立的消息，RPC风格的耦合度比DO风格要小一些，但是RPC风格也常常会带来客户端与服务器端的紧耦合。支持统一接口+超文本驱动的REST风格，可以达到最小的耦合度。

（29）从面向实用的角度来看，REST架构风格可以为Web开发者带来三方面的利益：
    简单性
采用REST架构风格，对于开发、测试、运维人员来说，都会更简单。可以充分利用大量HTTP服务器端和客户端开发库、Web功能测试/性能测试工具、HTTP缓存、HTTP代理服务器、防火墙。这些开发库和基础设施早已成为了日常用品，不需要什么火箭科技（例如神奇昂贵的应用服务器、中间件）就能解决大多数可伸缩性方面的问题。

    可伸缩性
充分利用好通信链各个位置的HTTP缓存组件，可以带来更好的可伸缩性。其实很多时候，在Web前端做性能优化，产生的效果不亚于仅仅在服务器端做性能优化，但是HTTP协议层面的缓存常常被一些资深的架构师完全忽略掉。

    松耦合
统一接口+超文本驱动，带来了最大限度的松耦合。允许服务器端和客户端程序在很大范围内，相对独立地进化。对于设计面向企业内网的API来说，松耦合并不是一个很重要的设计关注点。但是对于设计面向互联网的API来说，松耦合变成了一个必选项，不仅在设计时应该关注，而且应该放在最优先位置。

笔者在前面强调过REST是一种为运行在互联网环境中的Web应用量身定制的架构风格。REST在互联网这个运行环境之中已经占据了统治地位，然而，在企业内网运行环境之中，REST还会面临DO、RPC的巨大挑战。特别是一些对实时性要求很高的应用，REST的表现不如DO和RPC。所以需要针对具体的运行环境来具体问题具体分析。但是，REST可以带来的上述三方面的利益即使在开发企业应用时，仍然是非常有价值的。所以REST在企业应用开发，特别是在SOA架构的开发中，已经得到了越来越大的重视。

以上这些比较宏观的讲述了分布式系统架构以及REST，节选自http://www.infoq.com/cn/articles/understanding-restful-style

（30）利用restlet搭建应用：
http://restlet.org/learn/tutorial/2.1/

first step:
http://www.iteye.com/topic/182843

http://www.infoq.com/cn/articles/restlet-for-restful-service

（31）深入浅出REST
http://www.infoq.com/cn/articles/rest-introduction

这里面讲到接口的例子，比较利于理解：
用RPC的方式，接口会定义为很多很多，比如getXXXForXXX、updateXXX等等
但是用REST的方式，就非常清晰：
interface Resource中定义了GET/POST/PUT/DELETE
实现类通过URI来获取不同的，比如
/orders，它有这四种方法
/orders/{id}，它有这四种方法，不过有些区别比如这个GET就是取到一个订单的详情，上面那个是取得订单列表
/customers，它有这四种方法
/customers/{id}，它有这四种方法
/customers/{id}/orders，它也有这四种方法，GET表示取得这个用户的所有订单
这个例子很有启发意义，可以想到还可以有：
/customers/{id}/orders/{id}
/orders/{id}/customers
/orders/{id}/customers/{id}

另外一些URI的例子也有启发性：
http://example.com/orders/2007/11
http://example.com/products?color=green 

还有超媒体的例子也有启发性：
<order self="http://example.com/customers/1234"> 
   <amount>23</amount> 
   <product ref="http://example.com/products/4554"> 
   <customer ref="http://example.com/customers/1234"> 
</customer> </product></order>

（32）Remote Object、SOA和REST的演讲，笔记：
http://www.infoq.com/cn/presentations/xh-remote-object-soa-rest
企业内网架构现在也在向互联网架构转型，比如使用HTTP这些。
先看下历史remote object
曾经的企业内网倾向于去绑定在TCP上，比如CORBA，面向对象用IDL描述，想大一统的去抹平语言间的差异，就是让IDL可以去映射到各种OO语言的对象上。
当时sun公司弄的EJB2也是CORBA这种思路。也对HTTP的支持不好。
这种陈旧的技术在互联网技术兴起的时代，就注定被淘汰。
SOAP刚开始的时候就是基于HTTP的，这是因为当时是微软是想屏蔽操作系统差异，去和UNIX争取企业网市场。它的技术想法是利用HTTP去发RPC，利用HTTP去请求一个URL，然后响应是用XML表示。
因为毕竟HTTP和XML的优势，让SOAP领先于CORBA和EJB2，更加轻量和开放。
所以当时SOAP曾经在2000年左右风靡一时。
以上是第一个时代，下面到了第二个时代就是SOA架构
因为当大家都可以把自己的服务通过SOAP或者web service的方式暴露出来之后，就可以组成大的面向服务的架构。
这个听起来很好，貌似是可以很便捷的架构起这种服务，但是实际不怎么样。
然后这个时候就有人来卖产品了，说我有产品来帮你解决SOA方案，然后各种各样的企业和组织都进来添乱，结果后来就发展成有超级多web services的标准。
本来兴起的时候只是一个很轻量很简单的一个想法，就是我想通过HTTP将我的服务释放出来。但是后来就变成各种利益争夺的战场。甚至回过头来看看，现在SOA的这些标准已经比原先我们因为觉得非常复杂而被抛弃的CORBA。这种故事还在上演的类似于spring。。。

说完过去看看最近15年
现在企业的业务已经越来越转向互联网。
其实可能业务本身的单据流程并没有变化特别大（可能业务流程变化特别大，但是单据流程变化不太大），只是在过去这些年我们利用各种各样的技术来把纸质的单据映射为数据。可能15年前我们花钱用CORBA实现出来，10年前我们花钱用SOA实现出来。
（这个地方也想到一点，其实技术人员就是需要有各种新技术来延续自己的工作，不然难道都失业没事情做了吗）
那么萌生的想法是，数据流不是变化不太大嘛，那就想怎么来更好的表示出来。同时可以迎合频繁变化的业务流。
再来看看互联网是什么？互联网本质上是一组彼此关联的数据。它本身就应该是个数据平台，而我们之前的技术总想着去用互联网实现业务整合，这是不合适的。
因此在5年前，SOA被抛弃，REST出现。

Representational State Transfer
首先Representational好理解，用不同格式来表示数据嘛比如JSON、XML等等。
那么什么是state transfer？
REST是用来建模数据流程的，它并不是CRUD。
资源也就是数据，数据在数据流上是有些转换的，用四个基本语义和URL就可以直接表征哪个资源在做哪种转换。
就是说，我们现在有一种技术REST，它基于HTTP基于超媒体，它可以用来实现任何一种数据流。

那现在想用REST来实现业务流怎么来做呢，通过mashup的方式整合在一起。
再重复一下，REST并不是一个利用HTTP的CRUD。
rails2其实不是个好的REST实现，3.x就变得好一些。
这块也举了一个超媒体描述的例子，这样整个服务是自描述的：
<?xml version="1.0" encoding="utf-8"?>
<feed xmlns="http://www.w3.org/2005/Atom">
	<title>Messages</title>
	<link href="http://restapi/messages/" rel="self" />
	<id>urn:uuid:xxxxxxxxxxxx</id>
	<updated>2011-03-xx</updated>
	<entry>
		<title>message subject</title>
		<link rel="read" href="http://restapi/messages/[message_id]">
		<link rel="mark_as_read" href="http://restapi/messages/[message_id]/status">
		<link rel="delete" href="http://restapi/messages/[message_id]">
		<id>message_id</id>
		<updated>[sent_time]</updated>
	</entry>
</feed>


（33）和博哥讨论了一下
和博哥聊了一下union未来的组件化服务化架构方案

我的想法是：
服务性组件中有：业务逻辑层->数据持久层->DB&doris
向外暴露的有三种：
	1.咱们自己的浏览器访问可以用MVC通过TOMCAT直接调用，不是远程调用
	2.向企业内网暴露的接口是RPC，可以用spring remoting，或者mcpack json也不错，如果同是java平台的也可以用RMI
	3.向外暴露的接口是REST，通过HTTP的GET之类的来调用

博哥的想法是：
咱们所有对服务的调用都不要在tomcat中做了，就是说服务不要部署在tomcat中，如果还是和以前那样直接就在tomcat中作为一个webapp来调用的话，就不算服务化组件了。
所以设计的思路是这样的：
服务框架比如叫UES，它向外只暴露REST接口，数据格式可以只定为JSON
UES向外网暴露服务通过两种方式：
	1.ER直接通过访问UES向外暴露的域名比如api.union.baidu.com/uri来直接获取JSON
	2.我们自己的tomcat中只有mvc，只做请求URL到REST接口的映射，也通过REST接口去HTTP方式调用UES，这个地方虽然有HTTP的性能开销，但是由于是内网所以应该很快。
UES向内网暴露服务比如向CLB、CRM，就是通过REST接口。

这里涉及到还想做一个基于角色的访问权限管理，这块稍微扯得远一点还是提一下：
如果是外网进来的用户，还是用原来差不多的方式经过filter放入一些user token到memchached中，然后映射为某种role。
如果是内网的调用，通过IP来去判断它是哪种role。

有一个比较重要的事情是，UES部署在哪台IP的哪个端口上，这个是暴露服务很重要的一点，因为毕竟http://ip:port/uri是这样的，如果UES的部署有变动的话是需要能弹性支持的。
这就需要一个zookeeper去做这个IP到功能的映射，使用zookeeper的原因是它也是个树形结构和URI很像，另外它是强一致的。
那么UES的集群架构和zookeeper的集群架构彼此是透明的，不互相影响。
另外对于tomcat来讲，需要一个thread来一直监控zookeeper会不会给它发UES的IP调整信息。
如果UES的IP有切换，那么zookeeper首先会知道这个信息，然后会主动的发给TOMCAT。TOMCAT获取之后就不需要重启就可以拼接一个正确的URL去访问UES。

从代码角度，比如现在有个getReport接口
在这个service中可以有多个get接口，用各种annotation来表示权限以及URI
@path("/union/report/xxx")
@role("clb"/"crm")
getReport(ucid)

@path("/union/report/xxx")
@role("outer")
getReport()
当然这个还需要考虑，但是确实可以用annotation的方式来做这些AOP的东西。

大概的数据流程是这样的：
rest interface -> /xxx/user/{id} -> 从zookeeper获取UES地址然后拼接为完整的http请求 -> UES服务 -> 返回JSON -> 用GSON解析为对象 -> 返回给ER使用

其实我对interface的想法大概是这样的：
interface Resource有GET/POST/PUT/DELETE方法
其他数据都继承Resource，都是这几个方法。

-EOF-
