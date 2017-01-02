---
layout: post
title: "Prefer DEFER TO ASYNC"
description: "Prefer DEFER TO ASYNC"
category: ['JavaScript', 'Front-End', 'WPO']
tags: ['JavaScript', 'Front-End', 'WPO']
---
{% include JB/setup %}

前几天读到[Steve Sounder的一篇文章](http://calendar.perfplanet.com/2016/prefer-defer-over-async/)，他推荐优先使用`DEFER`而不是`ASYNC`.

主要原因如下：

1. `ASYNC`脚本会阻塞所有在它之后下载完成的同步脚本
1. `DEFER`脚本总是会跟`ASNYC`脚本同时或是之后执行
1. `DEFER`脚本永远不会阻塞同步脚本，然后`ASYNC`脚本也许会（依赖于下载`ASYNC`脚本有多快）
1. 如果你使用过多的`ASYNC`脚本，不妨把它替换为`DEFER`，如果你的页面的**DOMInteractive**时间很长，也许结果会让你意想不到


另外一些其它有意思的总结：

1. 阻塞渲染的罪魁祸首是**JavaScript**的执行
1. 同步**JavaScript**脚本会阻塞**HTML**的解析
1. `ASYNC`脚本不仅会阻塞渲染，而且还会阻塞其它同步脚本的执行
1. 如果脚本是用于渲染页面中的关键内容的，那么它就必须同步加载该脚本；相反，如果该脚本不是用于关键内容渲染的，那么我们就可以异步加载它（也就是说，也并不是所有脚本都必须是异步加载的）
1. `ASYNC`脚本一旦下载成功就会立即解析和执行，然而，`DEFER`脚本下载成功后并不立即执行，直到**HTML**解析完成（`performance.timing.domInteractive`）
1. `ASYNC`脚本的执行顺序不确定，然而`DEFER`脚本是按顺执行的（依据在DOM中的顺序）
1. `ASYNC`脚本和`DEFER`脚本不会阻塞**HTML**解析，但是他们能阻塞渲染。（这种情况发生在渲染完成前，它们已经被解析和执行并且接管主线程）
1. `DEFER`脚本的执行会推迟**DOMContentLoad**事件触发的，因为它是在**DOMInteractive**和**DOMContentLoaded**之间执行的
1. `ASYNC`会覆盖`DEFER`，所以在大多数的浏览器中同时指定它们二个与只指定`ASYNC`是一样的效果

