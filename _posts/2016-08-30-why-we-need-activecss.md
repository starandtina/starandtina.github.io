---
layout: post
title: "Why we need Active.css"
description: ""
category: 
tags: []
---
{% include JB/setup %}

* ToC
{:toc}


当我们在开发Web App时，需要尽可能地保证用户检验的一致性（包括外观和感觉），那么很常见的做法就是开发一套属于自己的Style Guide，它有自己的模式并且可以跨产品重用，同时，它可以让开发者轻而易举地重用这些标准模式并且让开发者知道如何去应用它。

这就是[Active.css](http://active.surge.sh/)所做的，它面向的是前端工程师或是其它想要应用这套样式规则的开发者，这里我特别想提到的是我们是如何构建它和和维护它的。


[Active.css](http://active.surge.sh/)是[活跃网络](http://activenetwork.com/)内部的一套CSS框架和一系列的Guidelines。它包括基础的全部样式（Typography，Icon Fonts等等），组件（Button, Alert, Tooltip等等），以及我们写HTML和CSS时的一系列准则，并且这一套CSS框架和准则已经在活跃网络实践了很多年了。

### 活跃网络开发

[Active.css](http://active.surge.sh/)是活跃网络的基础样式库，由一群像你们一样对前端(JS/HTML/CSS)有热情的一帮前端工程师鼓捣出来的。

### 开源

准备在**1.0.0**之后开源并且可以在**MIT**许可证下自由使用，同时[Active.css](http://active.surge.sh/)也是基于一系列的开源软件构建的，包括**LESS**, **PostCSS**, **Jekyll**等等。

### 构建工具

我们基于**[npm-scripts](https://docs.npmjs.com/misc/scripts)**和**[PostCSS](http://postcss.org/)**来编译我们的**LESS**源文件（去掉繁重的Grunt, Gulp或是Webpack），同时使用[cssnano](https://github.com/ben-eb/cssnano)来优化我们的CSS(autoprefixer, minifier等等)，同时使用[Parker](https://github.com/katiefenn/parker)来作为CSS分析工具，从而帮我们了解编译后的CSS的复杂度。

