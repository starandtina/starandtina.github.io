---
layout: post
title: 'JavaScript tricks 1: Avoid undeclared assignments'
description: ""
category: ['frontend', 'JavaScript', 'tricks']
tags: ['Frontend', 'JavaScript', 'Tricks', 'Best Practices']
---
{% include JB/setup %}

本文是关于**JavaScript**的陷阱和最佳实践中的一部分。

## 给未声明的变量赋值时千万不要忘记`var`关键字
{: .countheads }

给未声明的变量赋值时，JavaScript会自动在全局对象上创建一个新的同名属性（为什么这里不是说全局变量呢？见下面分析），而在浏览器中全局对象就是`window`对象，所以未声明的赋值其实是在`window`对象上创建属性。如下所示：

{% highlight JavaScript %}

function doSomething() { 
  var count = 10;
      title = "Maintainable JavaScript"; // Bad: global
}

{% endhighlight %}

上面这段代码是我们常见的一种错误，作者本意是想使用一个`var`声明二个本地变量，但是却意外的在第一个变量后面插入了一个分号而不是逗号，然后就导致自动地创建一个全局变量`title`。

创建全局变量通常会被认为是一种**Bad Practice**，特别是在一个团队开发环境中，随着code base的越来越大，会引起很多不必要的问题。

### 命名冲突

随着全部变量越来越多，命名冲突的危险就越来越大。从另一方面来说，最易维护的代码就是它所有的变量都是本地的。
通过使用全局对象，可以访问所有其他所有全局对象上预定义的对象、函数和属性，并且所有navtive的JavaScript对象都是在全局对象上预定义的，如果把你自己的变量加入这个scope，那么就很有可能与浏览器的实现或是其它第三方的代码相冲突。

### 代码易碎

如果一个模块或函数依赖与全局变量，那么它就与全局环境紧密偶合。那么如果全局环境发生改变，那么该模块或函数就有可能会发生breaking changes。如下所示：

{% highlight JavaScript %}

function sayHello() {
  console.log(helloMessage);
}

{% endhighlight %}

如果`helloMessage`不存在了，`sayHello`函数会抛异常。也就意味着，任何对全局环境的修改，都有可能导致代码错误。

在函数式编程中，会特别强调函数是要“引用透明”以及不能有“副作用”的。

>**引用透明（Referential transparency）**，指的是函数的运行不依赖于外部变量或"状态"，只依赖于输入的参数，任何时候只要参数相同，引用函数所得到的返回值总是相同的。

>所谓**"副作用"（side effect）**，指的是函数内部与外部互动（最典型的情况，就是修改全局变量的值），产生运算以外的其他结果。
函数式编程强调没有"副作用"，意味着函数要保持独立，所有功能就是返回一个新的值，没有其他行为，尤其是不得修改外部变量的值。

如果`helloMessage`作为函数的输入参数，那么这段代码就会更容易维护，如下所示：

{% highlight JavaScript %}

function sayHello(helloMessage) {
  console.log(helloMessage);
}

{% endhighlight %}

如上代码就不会产生全局依赖，因此也不会被全局环境的改变则影响。延伸开来，也就是说定义函数时，最好让更多的数据相对于函数来说是本地变量，任何来自于函数外的数据都应该通过参数传递进入函数，这样做的话，你就可以将函数和全局环境隔离开，同时允许你对函数和全局做出修改而不会互相影响。

### 可测试性差

最近，我开始为某个大型的SPA应添加前端测试，从最开始的测试框架选型到搭建整个前端测试环境，我发现是那么的困难。
因为虽然我们引入了AMD以及RequireJS作为模块的加载器，但是还是严重依赖于很多全局变量，如Backbone, jQuery, Handlebars, Highcharts等等，以至于添加UT的过程是如此的艰难。

最后导致的结果就是，在加载任何测试代码之前，首先必须要做的一件事就是确保这些全局变量已经加载成功，并且已经正常工作。所以你会看如下的代码：

{% highlight JavaScript %}

window.$ = window.jQuery = $;
window._ = _;
window.Backbone = Backbone;
window.Handlebars = Handlebars;

{% endhighlight %}

从测试的角度我们可以看出，保证你的代码模块或是函数不依赖于外部的全局变量可以使你的代码更容易维护，更容易测试，从而更高效。

### 全局变量还是全局属性？

大多数讲JavaScript的文章甚至是JavaScript的书通常都会这么说：“声明全局变量的方式有两种，一种是使用var关键字（在全局上下文中），另外一种是不用var关键字（在代码的任何地方）”。而这样的描述是错误的，要记住的是：**使用var关键字是声明变量的唯一方式**。

{% highlight JavaScript %}

foo=1

{% endhighlight %} 

上面代码仅仅是在全局对象上创建了新的属性，而不是变量。“不是变量”并不意味着它无法改变，它也是ECMAScript中变量的概念，并且它是全局对象(在浏览器中就是`window`)上的属性。

它们之间的不同点如下：

{% highlight JavaScript %}

alert(a); // undefined 
alert(b); // ReferenceError: "b" is not defined 

b = 10;
var a = 20;

{% endhighlight %}

我们可以看到，在进入全局上下文时并没有任何`b`，因为它不是变量，`b`是在执行代码阶段才出现。但是，在我们这个例子中也不会出现，因为在`b`出现前就发生了错误。

这里关于变量还有非常重要的一点：与简单属性不同的是，变量是不能删除的（即不拥有{DontDelete}标志），这意味着要想通过`delete`操作符来删除一个变量是不可能的，但是不用var关键字创建的全局属性是可以删除的，如下例。

{% highlight JavaScript %}

var GLOBAL_OBJECT = this; 

/* create global property via variable declaration; property has DontDelete */ 
var foo = 1; 

/* create global property via undeclared assignment; property has no DontDelete */ 
bar = 2; 

delete foo; // false 
typeof foo; // "number" 

delete bar; // true 
typeof bar; // "undefined"

{% endhighlight %}

在一些浏览器（如Chrom, Firefox）中的**console**中运行时可能结果会不正确，请参考该例测试[jsfiddle](http://jsfiddle.net/wn5e3u22/)。如果想知道关于`delelte`背后更多的故事，请移步至[Understanding-delete](#[1])。

### IE Bugs

避免未声明赋值的另外一个原因是在`MSHTML DOM`中有一些意思不到的古怪行为。当你在IE中创建一个未声明的赋值时，如果标识符的名字与`DOM`元素中的`id`或是`name`属性相同，那么IE会报错。如下所示：

{% highlight HTML %}

<p id="foo"></p>
<form name="bar" action=""><p></p></form>

<script type="text/javascript">
  try {
    foo = 1;
  }
  catch(e) {
    document.write(e); // TypeError: Object doesn't support this property or method
  }
  try {
    bar = 1;
  }
  catch(e) {
    document.write(e); // ReferenceError: Illegal assignment
  }
</script>

{% endhighlight %}

Note that plain variable declarations in global scope, or explicit assignments have no such problems:
注意，在全局环境中普通的变量声明或是显示的赋值并不会报错：

{% highlight HTML %}

<p id="foo"></p>
<form name="bar" action=""><p></p></form>

<script type="text/javascript">
  var foo = 1; // declares (and initializes) global `foo` variable
  window.foo = 1; // assigns to a "foo" property of `window` object
  this.foo = 1; // assigns to a "foo" property of Global Object
</script>

{% endhighlight %}

### Strict 模式

ECMAScript 5为JavaScript引入了[Strict Mode(严格模式)](#[2])。它的出发点是想让开发者写出更优雅，性能更好的JavaScript代码，从而能更快调试Bugs。

在严格模式下，未声明赋值会直接报错：

{% highlight JavaScript %}

"use strict";

foo = 1; // ReferenceError: foo is not defined

{% endhighlight %}

所以，如果你想让你的代码更**strict**，那么最好的选择是避免未声明赋值。


### 参考
{: #references }

1. {: id='[1]' }[Understanding delete](http://perfectionkills.com/understanding-delete/)
1. {: id='[2]' }[It’s time to start using JavaScript strict mode](http://www.nczonline.net/blog/2012/03/13/its-time-to-start-using-javascript-strict-mode/)
1. {: id='[3]' }[onloadfunction-considered-harmful](http://perfectionkills.com/onloadfunction-considered-harmful)
