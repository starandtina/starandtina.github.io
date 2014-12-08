---
layout: post
title: "Introduction to NATS"
description: ""
category: 
tags: []
---
{% include JB/setup %}

# Table of Contents

* [Introduction to NATS](#nats-intro)
* [NATS Application Protocol](#nats-protocol)
* [NATS Life Circle](#nats-life-circle)
  * [NATS Sample Code](#nats-life-circle-sample)
  * [NATS生命周期时序图](#nats-life-circle-diagram)
* [NATS主题匹配](#nats-subject)
  * [NATS主题特性](#nats-subject-features)
  * [NATS主题数据结构](#nats-subject-data-structure)
  * [NATS主题匹配算法](#nats-subject-matching-algo)
* [NATS与其它CloudFoundry组件](#nats-cf)
  * [NATS与Router](#nats-router)
  * [NATS与HealthManager](#nats-hm)
  * [NATS与DEA、CloudController](#nats-dea-cc)
* [总结](#conclusion)

## <a name='nats-intro'>Introduction to NATS</a>

在CloudFoundry系统中，包含许多内部组件，如CloudController、DEA、Router等等，如果某个组件(如DEA)需要去订阅其它组件的消息，那么DEA需要找到其它每一个组件并在其上注册事件，如果需要在每一个组件上注册多个事件，那么每个事件都需要执行一次上述过程（找到该组件并注册事件），这将增大系统的复杂度，并且使系统更加耦合，从而降低了系统的可扩展性。而NATS则解决了CloudFoundry的内部组件的通讯问题。

NATS是一个轻量级的分布式的消息订阅发布系统，CloudFounery使用NATS作为内部组件的通讯系统，从而进行基于主题的消息订阅和发布。NATS是基于EventMachine，由Ruby实现的，它由NATS客户端和服务端组成。客户端负责向服务端发送指令(订阅主题，发布主题等等)，而服务端则接收并处理来自客户端的指令，并做出响应。目前，NATS客户端同时也支持node.js、Go、Java以及Java-Spring的实现。

## <a name='nats-protocol'>NATS Application Protocol</a>

NATS是一个典型的网络应用程序，也就是说NATS客户端和服务器端是通过计算机网络通信的，NATS在此基础之上定义了属于自己的一套应用层协议。如下所示：

* SUB：订阅主题事件；
* PUB：发布主题事件；
* UNSUB：取消主题事件的订阅；
* MSG：发送消息给主题订阅者；
* PING：使Server发送一个响应消息；
* PONG：Server发回的一个响应，与PING配对；
* CONNECT：重新配置verbose和pedantic连接选项，同时会做用户验证(如果需要的话)；
* INFO：获取Sever信息，包括id, host, port等等

## <a name='nats-life-circle'>NATS生命周期</a>

### <a name='nats-life-circle-sample'>NATS示例代码</a>

以下面的NATS客户端和服务端代码为例，简单介绍一下NATS的整个生命周期。

NATS Server:

<p style='text-align:center'><img alt='NATS Sample Server' src='/assets/images/nats_sample_server.png' /></p>
<p style='text-align:center'>图1 NATS Sample Server</p>

NATS Client:

<p style='text-align:center'><img alt='NATS Sample Client' src='/assets/images/nats_sample_client.png' /></p>
<p style='text-align:center'>图2 NATS Sample Client</p>

### <a name='nats-life-circle-diagram'>NATS生命周期时序图</a>

下面是NATS的完整的生命周期时序图。

<p style='text-align:center'><img alt='NATS Life Circle' src='/assets/images/nats_life_circle.png' /></p>
<p style='text-align:center'>图3 NATS 生命周期</p>

1. 首先NATS Client需要建立与Server端的TCP连接，之后NATS Server会发送一个“INFO”消息至客户端，其中会包含Server端的详细信息，包括NATS Server的ID，host, port, version，以及是否需要用户验证，是否支持SSL等等；

2. 之后NATS Client会发送一个“CONNECT”消息至Server端，其中包含客户端的详细信息，Server端会根据配置来验证该客户端； 如果通过，会返回一个“+OK”消息；

3. 此时，TCP连接已经成功建立，Client会发送一个“PING”消息给Server，而Server会直接返回“PONG”消息给Client，之后Client可以订阅或是分发事件；

4. NATS Client订阅一个主题为“A.B.C”的消息，它的Subject Identifier是2（每次订阅时该ID会自增+1）；

5. 之后NATS Client会发布一个主题为“A.B.C”的消息，消息长度为8，内容为“I‘m Yuan”；

6. 最后，Server端会根据主题匹配算法，找到相应的订阅方，然后发送一个“MSG”消息给订阅方，并且带上主题“A.B.C”和主题ID“2”；

7. 此时，在NATS Client端，就可以根据主题ID找到相应的订阅者，执行相应的callback。
                                      
## <a name='nats-subject'>NATS主题匹配</a>

当一个NATS Client向NATS Server订阅一个消息后，Server会保留该订阅者的信息，包括当前主题，主题ID等等（具体参见图4），之后会根据主题匹配算法找到相应订阅者，并发布消息。

### <a name='nats-subject-features'>NATS主题特性</a>

* 主题是由许多token组成的，并以`.`分隔，如`A.B.C`；
* 支持两种通配符
    * *：可以匹配任意层级中的任意token，如`A.*.C`；
    * \>：可以匹配所有当前层级之后的token，如`A.>`；
* 支持简单的订阅者缓存策略
  * 如果有新的插入或删除，则清空缓存；
  * 如果超过CACHE_SIZE（默认是4096），则随机删除一个

例子如下所示：

|    插入主题     |      匹配主题     | 不匹配主题 |
| :-------------: | :-------------: | :-------------: |
|A.B.C|A.B.C|A.D.C|
|A.*.C|A.B.C/A.D.C/A.*.C|A.B.D|
|A.B.>|A.B.C/A.B.C.D.E.F.G|A.C.D.E.F.G|
{: .neat }

### <a name='nats-subject-matching-data-structure'>NATS主题数据结构</a>

在NATS服务器端内部，是由`sublist.rb`负责主题匹配工作的，它是由SublistLevel和SublistNode两个结构体组成的树形结构，如图4所示。SublistLevel主要维护的是token（包括二个特殊的通配符）至SublistNode节点的映射，而SublistNode主要保存的是订阅者信息，以及与下级SublistLevel的关系。Subscriber保存的是订阅方的信息，包含当前的Connection，主题，主题ID等等。

<p style='text-align:center'><img alt='NATS Subject Structure' src='/assets/images/nats_subject_structure.png' /></p>
<p style='text-align:center'>图4 Sublist、Subscriber 结构</p>

### <a name='nats-subject-matching-algo'>NATS主题匹配算法</a>

当NATS Server收到基于主题“A.B”的PUB消息后，它会把“A.B”主题拆分成二个token，然后将每个token中的内容从Sublist的Root Level开始从上至下开始进行匹配。如果每个Token都能匹配到节点，那么最后一个Token对应的SublistNode就是这个主题的订阅者。

<p style='text-align:center'><img alt='NATS Subject Matching Algorithms' src='/assets/images/nats_subject_matching_algo.png' /></p>
<p style='text-align:center'>图5 Sublist Workflow</p>

## <a name='nats-cf'>NATS与其它CloudFoundry组件</a>

### <a name='nats-router'>NATS与Router</a>

Router组件主要是对所有的请求进行路由，包括管理请求和App应用请求，其它组件如果想让Router路由请求由需要先向Router注册。过程如下：

1. Router启动时，会订阅`router.register`和`router.unreigster`这两个channel，等待其它组件向这二个channel发送消息；
2. 同时，Router会向`router.start`channel发送消息，其它需要向Router注册的组件(如DEA)在启动时都需要订阅`router.start`这个channel，一旦接收到该消息，都需要将需要注册的信息（DEA会包括host, port, uris, tags, app等等）向`router.register` channel发送；
3. Router接收到`router.register`消息后，会立即更新路由消息；同时，如果Router收到`router.unregister`消息，那么它会立即删除该路由信息；

另外，Router还订阅了`router.greet`这个channel，主要用于如果某个组件比NATS启动晚时，从而没有收到`router.start`消息时，也可以让该组件向Router注册路由信息，DEA也采用了这种方式来保证路由注册。

Router同时每隔一段时间(默认是0，所以不会发送)也会通过`router.active_apps`这个channel向其它组件发送当前active的app id信息。

### <a name='nats-hm'>NATS与HealthManager</a>

HealthManager组件主要是从DEA拿到app的各类运行信息，然后进行统计、分析和报警，如果某个instance出现异常，HM可以通过NATS通知CC启用或停止该instance。

HM订阅的DEA状态相关channel:

* dea.heartbeat：从DEA发来的心跳信息；
* droplet.exited：如果DEA关闭了app instance或是instance crash；
* droplet.updated：当DEA更新了app instance的信息

HM订阅的app状态查询相关的channel：

* healthmanager.status：查询app instance当前状态(flapping或crashed)下app instance信息；
* healthmanager.health：查询app instance当前的健康状况信息(是否running)；
* healthmanager.droplet：查询所有app instaince的运行信息

HM不仅统计应用的运行信息，它也会分析应用的状态。与此相关的channel：

* health.start：HM可以通过该channel让CC重启app instance；
* health.stop：HM可以通过该channel让CC关闭app instance

### <a name='nats-dea-cc'>NATS与DEA、CloudController</a>

DEA是一个安装在DEA VM node上的ruby的agent，由它来管理App的整个生命周期，包括stage, start, stop等等。
具体的流程可以参见How Application Are Staged。
与DEA相关的最重要的二件事是Staging app和Run app，都是由CloudFoundry通过NATS来触发的。

* Staging app
  * staging.locate：如果收到该channel的任何消息，DEA的Stager会默认发布一个`staging.advertise`消息；
  * staging.advertise：DEA的Stager会通过该channel向其它组件(特别是CC)广播它们自己capacity(包括available_memory)；
  * staging.XYZ.start：CC可以通过该channel来stage apps
* Run app
  * dea.locate：如果收到该channel的任何消息，DEA会默认发布一个`dea.advertise`消息；
  * dea.advertise：DEA会通过该channel向其它组件(特别是CC)广播它们自己capacity(包括available_memory)；
  * dea.XYZ.start：CC通过该channel来启动apps

NATS与DEA、CloudController主要交互工作流程如下：

* CC中的stager_pool和dea_pool启动时，分别订阅了`staging.advertise`和`dea.advertise`这二个channel，CC会根据该channel所提供的信息时刻更新这二个pool；
* 当有stage或是run app的请求时，CC中的dea_client就会去查询这两个pool，找到合适的dea，然后根据该dea的id向对应的`staging.dea_id.start`或`dea.dea_id.start`的channel发送消息
* DEA收到上述消息后，执行相应的任务(staging或run)
  
## <a name='conclusion'>总结</a>

对于CloudFoundry来说，NATS的作用是必不可少的。NATS让CF系统中的各组件各司其职，各个组件之间可以不需要知道互相的存在，只是通过简单的消息订阅发布来完成异步消息的通信和协同工作，彻底让系统解耦。同时NATS也大大的提高了系统的可维护性和可扩展性，这让我们增加新的组件也更加简单，更加高效。
