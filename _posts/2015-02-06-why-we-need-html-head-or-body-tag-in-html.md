---
layout: post
title: "Why we need &lt;html&gt;, &lt;head&gt; or &lt;body&gt;?"
description: ""
category: 
tags: []
---
{% include JB/setup %}

最近在面试某个职位的前端工程师时，因为对候选者要求不是很高，只需要了解基础的`HTML`和`CSS`就行了，所以经常我会问一个问题：

>要求用`HTML`写一个Hello, World!页面

其实这个问题只是想考察一下面试者是否熟悉基本的`HTML`结构。下面是一个简单示例：

{% highlight HTML %}

<!doctype html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Hello, World!</title>
</head>
<body>Hello, World!</body>
</html>

{% endhighlight %}

但是从上面这个问题，让我想了很多。

>为什么我们需要在一个`HTML`页面中添加`<html>`, `<head>`或`<body>`这三个元素呢？

带着这样的疑问，我做了4个实验，得出来了下面的结果。

* `HTML`从来都不需要`<html>`, `<head>`或`<body>`
* 如果某个元素在`<head>`中不合法，那么浏览器会从这个元素开始自动关闭`<head>`并开始`<body>`

用你自己喜欢的浏览器打开[测试1](http://starandtina.github.io/test/hello_world_without_html_head_body.html)，查看页面的源代码你会发现这三个`HTML`元素根本没出现在源代码中。但是，如果你用浏览器提供的开发者工具查看`DOM`的话，你会发现浏览器其实是自动地插入了三个元素的。

那么问题来了，**浏览器是如何知道应该在哪里插入`<head>`，而又在哪里打开`<body>`的呢？**

[测试2](http://starandtina.github.io/test/hello_world_with_unidentified_tag.html)中包含了另外一个&lt;ufo&gt;元素，这个元素它不是`HTML`标准中的一部分，同样你可以看到在源代码中也没有`<body>`元素，但是浏览器把`<title>`和`<meta>`放在`<head>`中，同时将`<ufo>`放到`<body>`中，这就是你为什么能看到`<ufo>`元素内容的原因。

我们说某个元素在`<head>`中是不合法，也就是说浏览器无法识别它为[Metadata content](http://www.w3.org/TR/html5/dom.html#metadata-content)（`<base>`, `<link>`, `<meta>`, `<noscript>`, `<script>`, `<style>`, `<template>`, `<title>`）。那么任何该元素后面的所有元素都会被放到`<body>`中，如[测试3](http://starandtina.github.io/test/hello_world_with_unidentified_tag_after_doctype.html)所示。

如果你在测试3的基础上添加上`<html>`, `<head>`或`<body>`元素，结果也是一样的，见[测试4](http://starandtina.github.io/test/hello_world_with_html_head_body_unidentified_tag.html)。
