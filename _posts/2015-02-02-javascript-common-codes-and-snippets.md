---
layout: post
title: "JavaScript Common Codes & Snippets"
description: ""
category: 
tags: []
---
{% include JB/setup %}

本文主要收集平常使用到的一些JavaScript Codes和Snippets。

## Table of Contents

1. [通过闭包构造私有成员](#1)
1. [使用闭包保存变量所在的作用域](#2)
1. [闭包与变量（闭包所能访问的变量，它是可以被改变的）](#3)
1. [创建对象的各种方法(八仙过海，各显神通)](#4)
1. [定义对象方法](#5)
1. [改变目标元素的显示/隐藏状态](#6)
1. [修改目标元素的CSS样式](#7)
1. [使用`className`](#8)
1. [DOM Level 0 中的事件处理模型](#9)
1. [使用`id`属性](#10)
1. [动态添加`className`](#11)
1. [解脱DOM](#12)
1. [强制浏览器Layout/Reflow](#13)
1. [在JavaScript中获取内联CSS样式](#14)
1. [DOM/HTML: 属性那是相当有用的](#15)
1. [简单的动画](#16)
1. [简单的淡入/淡出效果](#17)
1. [禁止某个资源的caching](#18)
1. [DOM/HTML:常用的方法](#19)
1. [从DOM树中移除当前的node](#20)
1. [在另外一个节点前插入一个节点](#21)
1. [克隆一个节点（浅/深克隆）](#22)
1. [使用另外一个节点替换一个子节点](#23)


## <a name='1'>通过闭包构造私有成员</a>

{% highlight JavaScript %}

function foo() {
  var id = 0;
  return function() { return id++; }
}
    
var makeid = x();

var i = makeid();
var j = makeid();

{% endhighlight %}

在函数`foo`中，`id`实际上是私有成员，我们通过且仅能通过闭包函数`makeid()`访问它。

**私有成员**：私有Private成员要由构造器生成。构造器中的普通的var变量和参数都成为私有成员。

{% highlight JavaScript %}

function Container(param) {
  this.member = param;
  var secret = 3;
  var self = this;
}

{% endhighlight %}

这个构造器有三个私有实例变量：`param`, `secret`, 和`self`，它们被附加到了对象上，但它们无法从外部访问，同时它们也无法被这个对象的公共方法所访问。他们只对私有方法和特权方法可见。私有方法则是构造器内部的函数，而特权方法是用`this`在构造器中分配的。

如果想了解更多关于私有成员的实现请参考[Private properties in JavaScript](https://curiosity-driven.org/private-properties-in-javascript).

## <a name='2'>使用闭包保存变量所在的作用域</a>

{% highlight JavaScript %}

function foo() {
  var req = new XMLHttpRequest();
  req.onreadystatechange = function () { /*此处创建闭包*/
    if (req.readyState == 4) {
      ...
    }
  }
}

{% endhighlight %}

尽管`onreadystatechange`是`req`的属性或是方法，但是它并不是由`req`触发的。因为它需要访问它自己，所以解决方案可以是使用一个
全局变量（如window.req）或是使用闭包让这些变量的值始终保存在内存中，让它不至于在`foo()`函数执行完毕就被`GC`回收。

{% highlight JavaScript %}

getClickHandler: function () {
  var me = this;
  return function(e) {
    me.showTip();
    ...
  }
}

{% endhighlight %}

以上编程模式，是经常会使用到的其中一种闭包写法。

## <a name='3'>闭包与变量（闭包所能访问的变量，它是可以被改变的）</a>

闭包是指在建立函数时，绑定了当时作用域下的有效的自由变量。基于此，首先我们给闭包下个定义：

>闭包就是能够读取其他函数内部变量的函数。

闭包最大用处有两个：

* 闭包可以让你可以读取函数内部的自由变量
* 同时闭包也让这些变量的值始终保持在内存中

那么什么是自由变量呢？

>自由变量是相对于函数而言，既不是本地变量也不是参数的变量，它的作用范围基本上是在函数定义的范围中。举个例子：

{% highlight JavaScript %}

function Robot() {
  var createTime = new Date();

  this.getAge = function () { // 這邊建立了Closure
    var now = new Date();
    var age = now - createTime;
    return age;
  }
}

var robot = new Robot();
alert(robot.getAge());

{% endhighlight %}

单单看`getAge`函数的话，`createTime`并没有多大的意义，对`getAge`而言，`createTime`是一个自由变量，同时它也是`Robot`函数的本地私有成员变量，但是在`getAge`所引用的函数中被关闭了，从而`createTime`的生命周期得以延续，即使它是使用`var`声明的，只要`getAge`中所引用的`Function`对象还存在，`createTime`自由变量也就能继续存活。

关于闭包，[Crockford](http://javascript.crockford.com/private.html)曾经说过:

>一个内部的函数总是可以访问这个函数外部的变量和参数，甚至在外部的函数返回之后。

{% highlight JavaScript %}

var myAlerts = [];

for (var i = 0; i < 2; i++) {
  myAlerts.push(
    function inner() {
      alert(i);
    });
}

myAlerts[0](); // 2
myAlerts[1](); // 2

{% endhighlight %}

第一眼看过去，JavaScript新手会认为`alert(i)`会弹出`inner`函数定义时逐步增长的`i`值，也就是分别显示`0`，`1`。这是我们最常犯的一种错误。`inner`函数是在全局上下文中创建的，从而它的`scope chain`是静态地绑定到全局上下的。当调用`inner`函数时，它会去找寻标识符`i`，而标识符的搜寻是沿着`scope chain`来的。所以它会去`inner`的`scope chain`上搜救变量`i`，但是，在调用`inner`函数的时候，变量`i`的值已经变为`2`，所以每次调用`inner`函数的结果都会是一样的。

JavaScript的一个很重要的特性就是它的解析器使用的是`Lexical Scoping`，意味着所以内部函数都是静态地绑定到创建它们的父上下文中的。闭包只能取得包含函数中任何变量的最后一个值。也就是说，闭包所保存的是整个变量对象，而不是某个特殊的变量。

上面的例子中，所有的`myAlerts`中保存的函数中能会引用到`2`，也就是被关闭的变量`i`的最后一个值（循环的计数器`i`在循环中的内部函数中被关闭了，它的生命周期得以延续，即使它是使用var声明的，只要`myAlerts`中所引用的`Function`对象还存在，`i`也就能继续存活）。

但是我们通过[使用闭包保存变量所在的作用域](#2)解决。

{% highlight JavaScript %}

var myAlerts = [];

for (var i = 0; i < 2; i++) {
  myAlerts.push(
    (function inner(i) {
      return function () {
        alert(i);
      }
    })(i));
}

myAlerts[0](); // 0
myAlerts[1](); // 1


{% endhighlight %}

## <a name='4'>创建对象的各种方法(八仙过海，各显神通)</a>

### 自定义构造器函数

{% highlight JavaScript %}

var obj = new function() {
  /* stuff */
};
  
{% endhighlight %}

### 对象字面量

{% highlight JavaScript %}

var obj = {
  /* stuff */
};

{% endhighlight %}

### Object.create

`Object.create`是[ECMAScript 5](http://es5.github.io/#x15.2.3.5)引入的，它是`new`的一个变体，它可以让你基于一个原型对象来创建一个新的对象。

{% highlight JavaScript %}

var prototypeDef = {
  protoBar: "protoBar",
  protoLog: function () {
    console.log(this.protoBar);
  }
};

var propertiesDef = {
  instanceBar: {
    value: "instanceBar"
  },
  instanceLog: {
    value: function () {
      console.log(this.instanceBar);
    }
  }
}

var foo = Object.create(prototypeDef, propertiesDef);
foo.protoLog(); // logs "protoBar" 
foo.instanceLog(); //logs "instanceBar"

{% endhighlight %}

这里有几点需要注意：

* `Object.creaet`中第二个参数`properties`中的值会覆盖第一个参数`prototype`中的同名属性
* 如果`Object.creaet`中第二个参数`properties`的类型是数组或是对象，那么通过`Object.create`创建的对象都会共享它
* 需要使用`isPrototypeOf`来检查对象的`prototype`对象而不是`instanceOf`

我们可以解决第二个问题，首先初始化`propertyArrayValue_`为`null`，然后当你添加元素时才初始化该数组。

{% highlight JavaScript %}

var prototypeDef = {
  protoArray: [],
};

var propertiesDef = {
  propertyArrayValue_: {
    value: null,
    writable: true
  },
  propertyArray: {
    get: function () {
      if (!this.propertyArrayValue_) {
        this.propertyArrayValue_ = [];
      }
      return this.propertyArrayValue_;
    }
  }
};

var foo = Object.create(prototypeDef, propertiesDef);
var bar = Object.create(prototypeDef, propertiesDef);
foo.protoArray.push("foobar");
console.log(bar.protoArray); //logs ["foobar"] 
foo.propertyArray.push("foobar"); 
console.log(bar.propertyArray); //logs []

{% endhighlight %}


## <a name='5'>定义对象方法</a>

{% highlight JavaScript %}

function MyObj() {}

MyObj.prototype = {
  foo: function () {
      alert(this);
  },
    ...etc..
};

var my1 = new MyObj();
my1.foo();

{% endhighlight %}

## <a name='6'>改变目标元素的显示/隐藏状态</a>

{% highlight JavaScript %}

T.dom.toggle = function (element) {
  element = T.dom.g(element);
  element.style.display = element.style.display == "none" ? "" : "none";

  return element;
};

{% endhighlight %}

使用`display:''`，与之相对的就是`display:block`。

设置`elem.style.display = ''`仅仅设置或是移除了**内联样式**。如果用户自定义的样式存在并且已经应用到了该元素上，那么一旦内联样式被移除了，该元素仍然会使用自定义样式来渲染。（这是因为`elem.style.display`已经移除了高优先级的内联样式，但是自定义的样式并没有受到影响，仍然会起作用。）因此，

1. 在通过`style.display=...`改变目标元素的显示或隐藏状态时，不要同时使用非内联样式和内联样式（只使用内联样式）
1. 或者，如果是使用`className`样式或是同时使用内联/非内联样式，可以通过`className`以及`display`属性来完成
1. 又或者，避免使用`elem.style.display = ''`这种形式，而使用`elem.style.display = 'none'|'block'`的这种形式

## <a name='7'>修改目标元素的CSS样式</a>

{% highlight JavaScript %}

//inline style
elem.style.xxx = ...
//classname
elem.className  = ...

{% endhighlight %}

修改内联样式或是修改`className`即可。

## <a name='8'>使用`className`</a>

{% highlight JavaScript %}

var elem = $('#id');
elem.className = 'foo';

{% endhighlight %}

使用`className`，而不是`class`，那是因为在ECMAScript 5中class是一个[Future Reserved Word](http://es5.github.io/#x7.6.1.2)，而在[ECMAScript 6](https://people.mozilla.org/~jorendorff/es6-draft.html#sec-reserved-words)中`class`已经是一个关键词了。

## <a name='9'>DOM Level 0 中的事件处理模型</a>

>事实上，并不存在DOM Level 0标准，它只存在于DOM的历史长河中。DOM Level 0通常被认为是Internet Explorer 4.0 and Netscape Navigator 4.0中所支持的`DHTML`。

DOM Level 0中的事件处理模型由Netscape Navigator引入，有二种主要的类型：

### 内联模型

在内联事件模型中，事件处理器是作为HTML元素的属性添加的。如下所示：

{% highlight HTML %}

<p>Hey <a href="http://www.example.com" onclick="triggerAlert('Joe'); return false;">Joe</a>!</p>

{% endhighlight %}

如果想了解更多，请参考[这里](http://www.quirksmode.org/js/events_early.html)。

### 传统模型

在传统事件模型中，事件处理器是通过JavaScript脚本添加或删除的。与内联模型相同的是，每一个事件一次只能绑定一个事件处理器。

{% highlight JavaScript %}

element.onclick = doSomething; //添加事件处理器
element.onclick = null; // 移除事件处理器

{% endhighlight %}

如果想了解更多，请参考[这里](http://www.quirksmode.org/js/events_tradmod.html)。

## <a name='10'>使用`id`属性</a>

在使用`id`属性时，文档中所有的元素必须都有唯一的`id`。对于多个元素使用相同的id，在IE中会导致各种种样的问题。

## <a name='11'>动态添加`className`</a>

{% highlight JavaScript %}

var table = document.getElementById('myTable');
var rows = table.getElementsByTagName('tr');
for (var i = 0; i < rows.length; i++) {
  if (i % 2) {
    rows[i].className += " even";
  }
}

{% endhighlight %}

在上面的代码中，展示了给一个table添加class以实现斑马线的效果。对于`onmouseover/onmouseout`的JS事件处理函数也可以通过这种方式添加。

## <a name='12'>解脱DOM</a>

{% highlight JavaScript %}

$('foo').related = $('bar');

{% endhighlight %}

给DOM对象添加指向其它相关联的DOM对象的属性，这种方式不会导致IE中的内存溢出，那是因为所有的操作都是在`DOM`内完成的。

更多请参考[IE的Memory Leak](http://javascript.crockford.com/memory/leak.html)。


## <a name='13'>强制浏览器Layout/Reflow</a>

### 浏览器渲染

首先有个问题：浏览器是如何渲染一个Web页面的？
从整体上来看，下面是渲染引擎在取得内容之后的基本流程：

{% highlight HTML %}

解析html以构建dom树 -> 构建render树 -> 布局render树 -> 绘制render树

{% endhighlight %}

1. 首先，基于从服务端返回的HTML字符串构建**DOM树**
2. 加载并解析样式（包括外部CSS文件以及内联样式），从而形成**[CSSOM(CSS Object Model)](http://dev.w3.org/csswg/cssom/)**(CSSOM提供了一套API来操作CSS)
3. 基于DOM和CSSOM构建别个一棵**Render树**，它包含一系列需要被渲染的对象（Webkit称这些对象为`Renderer`或`Render object`，而Gecko叫`frame`），但是**Render树**并不会包含一些不可见的DOM元素（包含`<head>`元素或是拥有`display:none`样式的元素）。每个'Renderer'都包含与之对应的`DOM`对象和计算完成的样式信息（如颜色，大小等等），它们将按照正确的顺序显示到屏幕上
4. 针对每一个`Renderer`，还需要计算它在屏幕上的确切坐标，这一步叫做`layout`
5. 最后就是`painting`，即遍历**render树**，并使用UI后端层在浏览器窗口中绘制每个节点

### 重绘(Repaint/Redraw)

修改元素样式（如`background-color`, `border-color`, `visibility`）时，如果并不影响元素在页面中的位置，那么浏览器只是会使用新的样式信息重绘该元素。

### Layout/Reflow

另外一些改变会影响文档的内容，结构或是元素位置，那么就会发生重绘（Webkit叫**Layout**，Gecko叫**Reflow**）。通常如下的一些修改会触发重新定位或回流：

* DOM操作：添加/删除/修改/隐藏元素(`display:none`)，移动元素或是修改元素出现顺序
* 内容变更，如表单元素值的修改
* 修改或是计算CSS属性
* 添加或是删除样式表
* 修改**class**属性
* 操作浏览器窗口，如resizing或是scrolling
* 激活**pseudo-class**，如`:hover`, `:checked`, `:first-child`等等

看一个简单来自Google Speedtracer的例子：

{% highlight JavaScript %}

// This line invalidates layout.
elementA.className = 'foo';

// This line requires layout to be up to date.
var aWidth = elementA.offsetWidth;

// Invalidates layout again.
elementB.className = 'bar';

// Requires layout to be up to date
var bWidth = elementB.offsetWidth;

{% endhighlight %}

通常来说，浏览器会尽可能地把重绘限制在更新的元素的那块区域。比如某个元素的重绘只会影响它后面的元素。
所以，我们可以通过批量读取和批量设置来解决。

{% highlight JavaScript %}

// This line invalidates layout.
elementA.className = 'foo';
elementB.className = 'bar';

// This line requires layout to be up to date.
var aWidth = elementA.offsetWidth
// Second Layout pass not needed.
var bWidth = elementB.offsetWidth;

{% endhighlight %}

参考：

* [How Browses Work](http://taligarsiel.com/Projects/howbrowserswork1.htm)
* [Speedtracer Examples](https://developers.google.com/web-toolkit/speedtracer/speed-tracer-examples)


## <a name='14'>在JavaScript中获取内联CSS样式</a>

我们可以通过`x.style`来访问内联样式，如下所示。另外，内联样式会覆盖掉其它的样式，除非使用`!important`来提高指定样式规则的权重。

{% highlight JavaScript %}

element.style.color

{% endhighlight %}

`color` 只在下列两种情况下是有效的：

1. 它是在`element`的内联样式定义上设置的
2. 它已经通过JavaScript中赋过值，如下所示：

{% highlight JavaScript %}

element.style.color ＝ 'red';

{% endhighlight %}

否则，`color` 都是无效的。

## <a name='15'>DOM/HTML: 属性那是相当有用的</a>

{% highlight JavaScript %}

<a href='javascript:showImg()'>
  <img title='my image' name='sunset'>
</a>

{% endhighlight %}

比如，在一个简单的slide展示中，鼠标滑过一个链接时，需要根据一些信息在图片显示区域展示相应的一张图片。那么img标签的title和name属性就可以帮你很容易的达到目的。
当然，其它一些自定义的属性也可以使用的。

另外一个例子是：

{% highlight HTML %}

<input type=text pattern='^\w+$' required=true />

{% endhighlight %}

页面加载时，JS代码可以遍历所有的表单字段，并且如果pattern和required属性存在的话，那么就添加一个validate函数，该函数主要负责在表单提交前，根据正则表达式来验证文本框中的内容。

## <a name='16'>简单的动画</a>

{% highlight JavaScript %}

---| 
-----| 
--------| 
-----------|

{% endhighlight %}

一个进度条的简单动画实现：我们可以通过逐渐增加`div`或`img`的宽度来实现。

{% highlight JavaScript %}

function progress() {
  var img = $('img');
  if(img.width < 200) {
    img.width += 5;
    // Don't autosize height along with width
    img.height = 5;
  } else {
    img.width = origwidth;
  }
}

setInterval('progress()', 500);

{% endhighlight %}

## <a name='17'>简单的淡入/淡出效果</a>

淡入/淡出效果一般用于从页面中添加/删除某个元素时。

{% highlight JavaScript %}

function fade(el) {
  var b = 155;
  var el = $('div');

  function f() {
    el.style.background = 'rgb(255,255,' + (b += 4) + ')';
    if (b < 255) {
      setTimeout(f, 40);
    }
  };
  f();
}

fade();

{% endhighlight %}

更多的可以参考[这里](http://peter.michaux.ca/articles/javascript-animations-from-scratch "JavaScript Animations from Scratch")。

## <a name='18'>禁止某个资源的Caching</a>

{% highlight JavaScript %}

document.write("< img src='foo.jpg?" + Math.random() + "' />"); 

{% endhighlight %}

我们可以修改**URL**，添加了一个伪随机数，这是一种常用的手段，常常用于禁止浏览器缓存图片。

## <a name='19'>DOM/HTML:常用的方法</a>

更多的信息请参考[quirksmode](http://quirksmode.org/ "http://www.quirksmode.org/dom/w3c_core.html#gettingelements")。

* document.createElement('div'): 创建一个`DIV`元素节点
* document.createTextNode(): 创建一个文本节点
* document.createDocumentFragment()：创建一个空的DOM文档碎片。我们可以利用这个API，先创建一个DOM文档碎片，然后在它上面添加新的元素节点，之后将它作为一个整体插入DOM中，以减少[Layout/Reflow](#13).
 
* document.getElementById()
* document.getElementsByTagName('h1')：返回文档中带有指定标签名的元素对象的集合。如果把特殊字符串*****传递给 `getElementsByTagName()` 方法，它将返回包含文档中所有子元素的集合，元素排列的顺序就是它们在文档中的顺序
* element.getElementsByTagName('h1')：返回指定元素`element`中带有指定标签名的元素对象的集合
* document.getElementsByClassName('class')：返回包含指定**class**的元素集合。目前[IE8不支持它](http://caniuse.com/#search=getElementsByClassName)。

我们常用的一些JavaScript库，如[jQuery](http://api.jquery.com/category/selectors/), [Prototype](http://prototypejs.org/doc/latest/dom/dollar-dollar/)等等，已经实现了通过CSS2/CSS3选择器返回元素集合，所以除了这些原生的方法外，我们还可以有更多的选择。

### 节点属性

#### [node.nodeType](http://www.w3.org/TR/DOM-Level-2-Core/core.html#ID-1950641247)

`nodeType'包含一个表示元素类型的数字。我们经常用到的如下所示：

1. Node.ELEMENT_NODE == 1
1. Node.ATTRIBUTE_NODE == 2
1. Node.TEXT_NODE == 3
1. Node.CDATA_SECTION_NODE == 3
1. Node.ENTITY_NODE == 6
1. Node.DOCUMENT_NODE == 9
1. Node.DOCUMENT_FRAGMENT_NODE == 11

#### node.nodeValue

对文本节点来说，`nodeValue`表示真正的文本。value of text for a Text node (useful for #text). Null for most other nodes (including element nodes) devedge link 

{% highlight HTML %}

<p>I am a JavaScript hacker.</p> 

{% endhighlight %}

{% highlight JavaScript %}

var x = [the paragraph];
var text = x.firstChild.nodeValue;

{% endhighlight %}

而对于属性节点来说，`nodeValue`则表示的是属性值。
除此之外，对于`document`和其它元素节点来说，[它会返回null](https://developer.mozilla.org/en-US/docs/Web/API/Node.nodeValue)。

#### node.nodeName

`nodeName`是最有用的一个属性。与它的名字一样，它包含了节点的名字。元素节点的名字始终与标签名相同，属性节点的名字始终与属性名字相同，文本节点的名字始终是`#text`，而文档节点的名字始终是`#document`。

只是有一点我们需要注意：对于HTML元素节点来说，不管你在HTML中写的是大写或是小写，`nodeName`都会返回大写的标签名。

#### node.getAttribute(name)

返回节点上你需要查询的属性的值。

####　node.setAttribute(name, value)

设置节点上某个指定的属性的新的值。

{% highlight HTML %}

<img src='sample.png' id='test' />

{% endhighlight %}

{% highlight JavaScript %}

var imgEl = document.getElementById('test'); 
alert(imgEl.getAttribute('src'));
imgEl.setAttribute('src', 'sample.png');

{% endhighlight %}

### 节点上的操作

#### 搜索节点

1. Children: firstChild, lastChild, childNodes
2. Siblings: nextSibling, previousSibling
3. Parents: parentNode

#### 添加/删除 nodes

1. removeChild(node), appendChild(node)
2. insertBefore(newNode, referenceNode)
3. replaceChild(replacingNode, replacedNode)

## <a name='20'>从DOM树中移除当前的node</a>

{% highlight JavaScript %}

var node = ..some_element..
node.parentNode.removeChild(node);

{% endhighlight %}

## <a name='21'>在另外一个节点前插入一个节点</a>

{% highlight JavaScript %}

var x = document.getElementsByTagName('p')[0]; 
var y = document.getElementsByTagName('h1')[0]; 
x.parentNode.insertBefore(x,y);

{% endhighlight %}

### 返回值

`insertBefore()`返回的是被插入节点的一个引用。

{% highlight JavaScript %}

var x = y.insertBefore(z, a);

{% endhighlight %}

那么现在`x`就包含对`z`的一个引用。

## <a name='22'>克隆一个节点（浅/深克隆）</a>

{% highlight JavaScript %}

var node = ...
var newNode = node.cloneNode(true|false); // true=deep copy, else shallow

{% endhighlight %}

需要注意的是，克隆节点时，并不会同时克隆事件处理函数。

## <a name='23'>使用另外一个节点替换一个子节点</a>

`replaceChild()`方法可以允许你将一个节点替换为另外一个节点。如果被插入的节点已经在DOM中了，那么它首先会从当前的DOM中位置移除掉。并且插入的节点和被替换的节点都会保持它们的所有子节点不变。

{% highlight JavaScript %}

var x = document.getElementsByTagName('h1')[0];
var y = document.getElementsByTagName('p')[1];
x.parentNode.replaceChild(x, y);

{% endhighlight %}

### 返回值

`replaceChild()`返回的是被替换的节点的一个引用。

{% highlight JavaScript %}

var x = y.replaceChild(z, a);

{% endhighlight %}

那么现在`x`就包含对`a`的一个引用。
