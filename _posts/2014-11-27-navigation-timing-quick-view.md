---
layout: post
title: "Navigation Timing Quick View"
description: ""
category: ["JavaScript", "js-dev", "js-perf"]
tags: []
---
{% include JB/setup %}


# Navigation Timing

* [Navigation Timing](http://www.w3.org/TR/navigation-timing/)
* [Navigation Timing 2](http://www.w3.org/TR/navigation-timing-2/)
* [Measuring Page Load Speed with Navigation Timing](http://www.html5rocks.com/en/tutorials/webperformance/basics/)

Navigation Timing是一套JavaScript API，主要用于衡量文档加载性能。

Navigation Timing 2目前还是工作草案，它是做为第一版的一个补充，但是新增了如下内容：

* 支持Performance Timeline；
* 支持High Resolution Time；
* 提供link negotiation时间信息；
* 提供prerender时间信息

# PerformanceTiming Interface

在第1版中，我们可以通过`window.performance.timing`获取`PerformanceTiming`对象。
在第2版中，我们也可以通过[Performance Timeline](http://www.w3.org/TR/performance-timeline/#sec-window.performance-attribute)提供的API访问。

* navigationStart

浏览器完成卸载前一个文档的时间(也就是准备加载新页面的那个起始时间)。如果没有前一个文档，那么就返回 timing.fetchStart的值。

* unloadEventStart 

如果前一个文档，和当前文档同源，返回前一个文档发生unload事件前的时间。如果没有前一个文档，或不同源，则返回0。

* unloadEventEnd 

如果前一个文档和当前文档同源，返回前一个文档发生unload事件的时间；如果没有前一个文档，或不同源，则返回0。

如果发生了HTTP重定向，或者类似的事情，并且从导航开始中间的每次重定向，并不都和当前文档同域的话，则返回0。 

* redirectStart

如果发生了HTTP重定向，或者类似的事情，并且，从导航开始，中间的每次重定向都和当前文档同域的话，就返回开始重定向的timing.fetchStart的值。其他情况则返回0。

* redirectEnd

如果发生了HTTP重定向，或者类似的事情。并且，从导航开始，中间的每次重定向都和当前文档同域的话，就返回最后一次重定向，接收到最后一个字节数据后的那个时间。其他情况则返回0。

* fetchStart

如果一个新的资源(这里是指当前文档)获取被发起,或类似的事情发生,则 fetchStart必须返回用户代理开始检查其相关缓存的那个时间,其他情况则返回开始获取该资源的时间。

* domainLookupStart

返回用户代理对当前文档所属域进行DNS查询开始的时间. 如果此请求没有DNS查询过程,如长连接，资源cache,甚至是本地资源等. 那么就返回 fetchStart的值。

* domainLookupEnd

返回用户代理对结束对当前文档所属域进行DNS查询的时间.如果此请求没有DNS查询过程,如长连接，资源cache,甚至是本地资源等. 那么就返回 fetchStart的值。

* connectStart

返回用户代理向服务器服务器请求文档，开始建立连接的那个时间,如果此连接是一个长连接,又或者直接从缓存中获取资源（即没有与服务器建立连接）.则返回domainLookupEnd的值。

* connectEnd

返回用户代理向服务器服务器请求文档，建立连接成功后(注意，不是断开连接的时间.)的那个时间.如果此连接是一个长连接，又或直接从缓存中获取资源 （即没有与服务器建立连接）,则返回domainLookupEnd的值。

* secureConnectionStart

可选特性.用户代理如果没有对应的东东,就要把这个设置为undefined.如果有这个东东,并且是HTTPS协议,那么就要返回开始SSL握手的那个时间. 如果不是HTTPS, 那么就返回0。

* requestStart

返回从服务器、缓存、本地资源等,开始请求文档的时间. 

如果请求中途,连接断开了,并且用户代理进行了重连，并重新请求了资源,那么requestStart就必须为这个新请求所对应的时间。

`performance.timing`并不包含一个单表请求结束的"requestEnd"接口. 原因有两点:

1. 用户代理所能确定的请求的结束,并不能代表正确的网络栓书中的结束时间. 所以设计这个属性并没什么用处.
1. 一些用户代理，如果要封装一个代表HTTP层面的，请求结束时间的接口,成本会非常高昂.


* responseStart

返回用户代理从服务器、缓存、本地资源中，接收到第一个字节数据的时间.
 
 
* responseEnd

返回用户代理接收到最后一个字符的时间，和当前连接被关闭的时间中，更早的那个. 同样,文档可能来自服务器、缓存、或本地资源.
补充: 此值的读取应该是在我们可以确保真的是Response结束以后. 比如window.onload.  因为考虑到chunked输出的情况. 那么我们脚本执行，并获取该值时，响应还没有结束. 这就会导致获取时间不准确.
 

* domLoading 

返回用户代理把其文档的 "current document readiness" 设置为 "loading"的时候.
(current document readiness 其实就是document.readyState API对应的状态.)

[参考](http://www.w3.org/html/wg/drafts/html/master/dom.html#current-document-readiness)

* domInteractive

返回用户代理把其文档的 "current document readiness" 设置为 "interactive"的时候.

从标准来说,domReady的状态为"interactive"时,意味着,文档解析结束了. 因为标准中描述, DOM树创建结束后第一件事，就是把 "current document readiness" 设置为"interactive"

[参考](http://www.w3.org/html/wg/drafts/html/master/syntax.html#the-end)中第一步.

* domContentLoadedEventStart

返回文档发生 DOMContentLoaded事件的时间.

[参考](http://www.w3.org/html/wg/drafts/html/master/syntax.html#the-end)中第4步.

DOMContentLoad和 DOMInteractive 之间差了两个步骤. 
其中之一是, 所有open elements出栈 ，然后去看看待运行的script list中是否有需要运行的脚本，如果有则执行，一直到这个列表为空了.再触发DOMContentLoad.  需要主的是这个待运行脚本列表.有些可能在不同浏览器中，被加入进去的行为可能不同. 比如 document.write写入文档流的脚本，以及script deferr 的脚本.. 所以我们应该知道deferr的脚本也是要他推迟domContentLoaded的，也就是我们最常用的所谓domReady.(至少html5的规范是如此.)

* domContentLoadedEventEnd

文档的DOMContentLoaded 事件的结束时间.

> 所谓事件结束的时间，是指，如果DOMContentLoaded事件被开发者注册了回调事件.那么这个事件的End时间减去Start的时间.就会是这个回调执行的大概时间. 

> 当然部分浏览器实现可能会有2-3ms的误差. 但是这个时间，基本可以忽略不计. 类似的情况还有后面的.loadEventStart,End. 即 window.onload 所有回调所消耗的时间.

* domComplete

返回用户代理把其文档的 "current document readiness" 设置为 "complete"的时候.

[参考](http://www.w3.org/html/wg/drafts/html/master/syntax.html#the-end)中第7步.

* loadEventStart

文档触发load事件的时间. 如果load事件没有触发,那么该接口就返回0.
 
 
* loadEventEnd

文档触发load事件结束后的时间. 如果load事件没有触发,那么该接口就返回0.


# 指标值

* headtime
* tti(传输时间间隔)
* domready
* onload


|指标值|指标算法|
| :-------------: | :------------- |
|DNS|domainLookupEnd - domainLookupStart|
|页面等待响应的耗费时间|responseEnd – fetchStart  |
|页面DOM渲染的耗费时间(domready)|domContentLoadedEventEnd – fetchStart  |
|页面Load完成的耗费时间(onload)|loadEventEnd  – fetchStart  |
{: .neat }

# timing overview

[timing overview](http://www.w3.org/TR/navigation-timing/timing-overview.png)

