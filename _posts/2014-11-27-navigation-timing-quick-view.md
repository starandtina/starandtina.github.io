---
layout: post
title: "Navigation Timing Quick View"
description: "Quick start guide for Navigation Timing API."
category: ["JavaScript", "js-dev", "js-perf"]
tags: ["JavaScript", "WPO"]
---
{% include JB/setup %}


# Navigation Timing

* [Navigation Timing](http://www.w3.org/TR/navigation-timing/)
* [Navigation Timing 2](http://www.w3.org/TR/navigation-timing-2/)
* [Measuring Page Load Speed with Navigation Timing](http://www。html5rocks。com/en/tutorials/webperformance/basics/)

Navigation Timing是一套JavaScript API，主要用于衡量文档加载性能。

Navigation Timing 2目前还是工作草案，它是做为第一版的一个补充，但是新增了如下内容：

* 支持Performance Timeline；
* 支持High Resolution Time；
* 提供link negotiation时间信息；
* 提供prerender时间信息

# PerformanceTiming Interface

在第1版中，我们可以通过`window.performance.timing`获取`PerformanceTiming`对象。
在第2版中，我们也可以通过[Performance Timeline](http://www.w3.org/TR/performance-timeline/#sec-window.performance-attribute)提供的API访问。

## navigationStart

浏览器完成卸载前一个文档的时间(也就是准备加载新页面的那个起始时间)。如果没有前一个文档，那么就返回`timing.fetchStart`的值。

## unloadEventStart

如果前一个文档，和当前文档同源，返回前一个文档发生`unload`事件前的时间。如果没有前一个文档，或不同源，则返回0。

## unloadEventEnd

如果前一个文档和当前文档同源，返回前一个文档发生`unload`事件的时间；如果没有前一个文档，或不同源，则返回0。

如果发生了HTTP重定向，或者类似的事情，并且从导航开始中间的每次重定向，并不都和当前文档同域的话，则返回0。

## redirectStart

如果发生了HTTP重定向，或者类似的事情，并且，从导航开始，中间的每次重定向都和当前文档同域的话，就返回开始重定向的`timing.fetchStart`的值。其他情况则返回0。

## redirectEnd

如果发生了HTTP重定向，或者类似的事情。并且，从导航开始，中间的每次重定向都和当前文档同域的话，就返回最后一次重定向，接收到最后一个字节数据后的那个时间。其他情况则返回0。

## fetchStart

如果一个新的资源(这里是指当前文档)获取被发起，或类似的事情发生，则`fetchStart`必须返回用户代理开始检查其相关缓存的那个时间，其他情况则返回开始获取该资源的时间。

## domainLookupStart

返回用户代理对当前文档所属域进行DNS查询开始的时间。如果此请求没有DNS查询过程，如长连接，资源cache，甚至是本地资源等。那么就返回 **fetchStart**的值。

## domainLookupEnd

返回用户代理对结束对当前文档所属域进行DNS查询的时间。如果此请求没有DNS查询过程，如长连接，资源cache，甚至是本地资源等。那么就返回 **fetchStart**的值。

## connectStart

返回用户代理向服务器服务器请求文档，开始建立连接的那个时间，如果此连接是一个长连接，又或者直接从缓存中获取资源（即没有与服务器建立连接）。则返回domainLookupEnd的值。

## connectEnd

返回用户代理向服务器服务器请求文档，建立连接成功后(注意，不是断开连接的时间。)的那个时间。如果此连接是一个长连接，又或直接从缓存中获取资源 （即没有与服务器建立连接），则返回**domainLookupEnd**的值。

## secureConnectionStart

可选特性。用户代理如果没有对应的属性，就要把这个设置为`undefined`。如果有这个属性，并且是HTTPS协议，那么就要返回开始SSL握手的那个时间。如果不是HTTPS， 那么就返回0。

## requestStart

返回从服务器、缓存、本地资源等，开始请求文档的时间。

如果请求中途，连接断开了，并且用户代理进行了重连，并重新请求了资源，那么requestStart就必须为这个新请求所对应的时间。

**注意**: 此接口并不包含一个表示请求结束的属性，如**requestEnd**，原因有两点:

  * 用户代理所能确定的请求的结束，并不能代表正确的网络传输中的结束时间，所以设计这个属性并没什么用处。
  * 由于HTTP层封装，从而导致一些用户代理如果要判断请求结束时间的代价非常昂贵。

## responseStart

返回用户代理从服务器、缓存、本地资源中，接收到第一个字节数据的时间。

## responseEnd

返回用户代理接收到最后一个字符的时间，和当前连接被关闭的时间中，最早的那个。同样，文档可能来自服务器、缓存、或本地资源。

>此值的读取应该是在我们可以确保真的是Response结束以后，比如`window.onload`。因为可能会有`chunked`输出的情况,那么脚本执行并获取该值时，响应还没有结束，这就会导致获取时间不准确。

## domLoading

返回用户代理把其文档的[current document readiness](http://www.w3.org/html/wg/drafts/html/master/dom.html#current-document-readiness)设置为[loading](http://www.w3.org/html/wg/drafts/html/master/dom.html#current-document-readiness)的时候。

>这是整个过程开始的时间戳，浏览器开始解析 HTML 文档第一批收到的字节 document。另外，current document readiness 其实就是`document。readyState` API对应的状态。

## domInteractive

返回用户代理把其文档的[current document readiness](http://www.w3.org/html/wg/drafts/html/master/dom.html#current-document-readiness)设置为["interactive"](http://www.w3.org/html/wg/drafts/html/master/syntax.html#the-end)的时候。

>标记 DOM 准备就绪，即浏览器完成解析并且所有 HTML 和 DOM 构建完毕的时间点

从标准来说，domReady的状态为"interactive"时，意味着文档解析结束了。因为标准中描述DOM树创建结束后第一件事，就是把[current document readiness](http://www.w3.org/html/wg/drafts/html/master/dom.html#current-document-readiness)设置为"interactive"。

[HTML 5.1 8.2.6](http://www.w3.org/html/wg/drafts/html/master/syntax.html#the-end)中第一步。

## domContentLoadedEventStart

返回文档发生`DOMContentLoaded`事件的时间。

>标记 DOM 准备就绪并且没有样式表阻碍 JavaScript 执行的时间点 - 意味着我们可以开始构建呈现树了。

[HTML 5.1 8.2.6](http://www.w3.org/html/wg/drafts/html/master/syntax.html#the-end)中第4步。

DOMContentLoad和 DOMInteractive 之间差了以下两个步骤。

  * 先确保所有[open elements](http://www.w3.org/html/wg/drafts/html/master/syntax.html#stack-of-open-elements)出栈
  * 然后去看看[文档结束解析后待运行的script list](http://www.w3.org/html/wg/drafts/html/master/scripting-1。html#list-of-scripts-that-will-execute-when-the-document-has-finished-parsing)中是否有需要运行的脚本，如果有则执行，一直到这个列表为空了。再触发`DOMContentLoad`事件。

>需要注意的是这个待运行脚本列表。在不同浏览器中可能会有所不同。大致会包括`document.write`写入文档流的脚本，以及`defer`的脚本。所以从这里我们应该知道**`defer`的脚本的执行是推迟`DOMContentLoad`事件触发的**。很多JavaScript 框架等待此事件发生后，才会开始执行它们自己的逻辑，比如JQuery。因此，浏览器会通过捕获 _EventStart_ 和 _EventEnd_ 时间戳，跟踪执行逻辑所需的时间。

## domContentLoadedEventEnd

文档的`DOMContentLoaded`事件的结束时间。

> 所谓事件结束的时间，是指如果DOMContentLoaded事件被开发者注册了回调事件，那么这个事件的 _Event End_ 时间减去 _Event Start_ 的时间，那么就可以跟踪回调执行逻辑所需的时间。

> 当然部分浏览器实现可能会有2-3ms的误差。但是这个时间，基本可以忽略不计。类似的情况还有后面的`loadEventStart`和`loadEventEnd`。即`window.onload`所有回调所消耗的时间。

## domComplete

返回用户代理把其文档的[current document readiness](http://www.w3.org/html/wg/drafts/html/master/dom.html#current-document-readiness)设置为["complete"](http://www.w3.org/html/wg/drafts/html/master/syntax.html#the-end)的时候。

>标记网页及其所有附属资源都已经准备就绪的时间，顾名思义，所有的处理完成，网页上所有资源（包括图片，Web Fonts等）都已下载完成 - 即加载旋转图标停止旋转。

[HTML 5.1 8.2.6](http://www.w3.org/html/wg/drafts/html/master/syntax.html#the-end)中第7步。

## loadEventStart

文档触发load事件的时间。如果load事件没有触发，那么该接口就返回0。
 
>作为每个网页加载的最后一步，浏览器会触发onLoad事件，以便触发附加的应用逻辑。

## loadEventEnd

文档触发load事件结束后的时间。如果load事件没有触发，那么该接口就返回0。


# 指标值

我们现在已经知道Navigation Timing所提供的API，下面是一些常用的Use Cases:

|指标|指标值|
| :-------------: | :------------- |
|DNS|domainLookupEnd - domainLookupStart|
|页面等待响应的耗费时间|responseEnd – fetchStart  |
|页面DOM渲染的耗费时间(domReady)|domContentLoadedEventEnd – fetchStart  |
|页面Load完成的耗费时间(onLoad)|loadEventEnd  – fetchStart  |
{: .neat }

# Navigation Timing API Overview

[Timing Overview Chart](http://www.w3.org/TR/navigation-timing/timing-overview.png)

