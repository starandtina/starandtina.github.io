---
layout: post
title: "JavaScript Common Code & Snippets"
description: ""
category: 
tags: []
---
{% include JB/setup %}


本文主要用于收集平常使用到的一些js code和snippets。

## 1. 使用闭包构造私有变量

  {% highlight JavaScript %}
  function x() {
      var id = 0;
      return function() { return id++; }
  }
      
  var makeid = x();

  var i = makeid();
  var j = makeid();
  {% endhighlight %}

*私有成员*：私有Private成员要由构造器生成。构造器中的普通的var变量和参数都成为私有成员。

在函数x中，id实际上是私有成员，我们通过且仅能通过闭包函数makeid()访问它。

## 2. 使用闭包保存变量的作用域


  {% highlight JavaScript %}
  foo() {
     var req = xmlhttp();
     req.onreadystatechange = function() { /*closure created here*/
    if (req.readyState == 4) {
        ...
       }
    }
  }
  {% endhighlight %}

尽管`onreadystatechange`是req的属性或是方法，但是它并不是由req触发的。因为它需要访问它自己，所以解决方案可以是使用一个
全局变量（如window.req）或是使用闭包延伸它的作用域，让它不至于在foo()函数执行完毕就被GC回收。

  {% highlight JavaScript %}
  getClickHandler : function() {
    var me = this;
    return function(e) {
      me.showTip();
      ...
    }
  }
  {% endhighlight %}

以上编程模式，是Holmes中经常会使用到的其中一种闭包写法。


## 3. 闭包与变量（闭包所访问的变量，它是可以被改变的）


  {% highlight JavaScript %}
  x = elements('a')
  for (var i = 0; i < 10; i++)
    x[i].onclick = function() { /*...use x[i] here...*/ }
    }
  {% endhighlight %}

闭包只能取得包含函数中任何变量的最后一个值。也就是说，闭包所保存的是整个变量对象，而不是某个特殊的变量。

上面的例子中，所有的`onclick`函数中能会引用到x[10]，也就是被关闭的变量 *i* 的最后一个值（循环的计数器 *i* 在循环中的内部函数中被关闭了，它的生命周期得以延续，即使它是使用var声明的，只要`onclick`所引用的Function对象还存在，*i* 也就能继续存活）。


## 4. 创建对象的各种方法(八仙过海，各显神通)

  
  {% highlight JavaScript %}
  A) new and function definition combined (自定义构造器函数)
  var obj =   new function() {
    /* stuff */
  }
    
  B) object literal
  var obj = {
    /* stuff */
  }
  {% endhighlight %}

## 5. 定义对象方法


  {% highlight JavaScript %}
  function MyObj() {}
  MyObj.prototype = {
    foo: function () {
      alert(this);
      }    
    ...etc..
    }

  var my1 = new MyObj();
  my1.foo();
  {% endhighlight %}

## 6. 改变目标元素的显示/隐藏状态


  {% highlight JavaScript %}
  baidu.dom.toggle = function (element) {
    element = baidu.dom.g(element);
    element.style.display = element.style.display == "none" ? "" : "none";

    return element;
  };
  {% endhighlight %}

使用`display:''`，与之相对的就是`display:block`。

设置`elem.style.display = ''`仅仅设置或是移除了*内置样式*。如果用户自定义的样式存在并且已经应用到了该元素上，那么一旦内联样式被移除了，该元素仍然会使用自定义样式来渲染。（这是因为`elem.style.display`已经移除了高优先级的内联样式，但是自定义的样式并没有受到影响，仍然会起作用。）因此，

  1. 在通过`style.display=...`改变目标元素的显示或隐藏状态时，不要同时使用非内联样式和内联样式（只使用内联样式）；
  2. 或者，如果是使用className样式或是同时使用内联/非内联样式，可以通过修改样式表className中的display属性来完成；
  3. 又或者，避免使用`elem.style.display = ''`这种形式，而使用`elem.style.display = 'none'|'block'`的这种形式。


## 7. 修改目标元素的CSS样式

  
  {% highlight JavaScript %}
  //inline style
  elem.style.xxx = ...
  //classname
  elem.className  = ...
  {% endhighlight %}

内联样式或是className都可以使用。


## 8. 使用 *className*


  {% highlight JavaScript %}
  elem.className = 'foo';
  {% endhighlight %}

使用`className`，而不是`class`（那是因为class可能会成为一个 reserved word）


## 9. DOM 0 Event handlers are methods

  {% highlight JavaScript %}
  <form>
  <button onclick="myfunc()"></button>
  </form>
  <script>
  alert(button.onclick);
  </script>
  {% endhighlight %}

经典的事件处理器（如button.onclick = ...）都是相应的 DOM button 对象的方法。


## 10. 使用 *id* 属性


在使用*id*属性时，所有的元素必须都有唯一的*id*。对于多个元素使用相同的id，在IE中会导致各种种样的问题。__todo__


## 11. Programmatic assignment of classes

  {% highlight JavaScript %}
  var table = document.getElementById('mytable');
  var rows  = table.getElementsByTagName('tr');
  for (var i = 0; i < rows.length; i++) {
    if(i%2) {
        rows[i].className += " even";
        }
    }
  {% endhighlight %}

在上面的代码中，展示了给一个table添加class以实现斑马线的效果。对于`onmouseover/onmouseout`的JS事件处理函数也可以通过这种方式添加。


## 12. Ease of DOM

  {% highlight JavaScript %}
  $('foo').related = $('bar');
  {% endhighlight %}

给DOM对象添加指向其它相关联的DOM对象的属性（这种方式不会导致IE中的内存溢出，那是因为所有的操作都是在DOM内完成的）

>Memory Leaks: Microsoft's Internet Explorer contains a number of leaks, the worst of which is an interaction with JScript. When a DOM object contains a reference to a JavaScript object (such an event handling function), and when that JavaScript object contains a reference to that DOM object, then a cyclic structure is formed. This is not in itself a problem. At such time as there are no other references to the DOM object and the event handler, then the garbage collector (an automatic memory resource manager) will reclaim them both, allowing their space to be reallocated. The JavaScript garbage collector understands about cycles and is not confused by them. Unfortunately, IE's DOM is not managed by JScript. It has its own memory manager that does not understand about cycles and so gets very confused. As a result, when cycles occur, memory reclamation does not occur. The memory that is not reclaimed is said to have leaked. Over time, this can result in memory starvation. In a memory space full of used cells, the browser starves to death.


## 13. 强制CSS layout


  {% highlight JavaScript %}
  window.resizeBy(0,1)
  window.resizeBy(0, -1)
  {% endhighlight %}

上述代码可以强制css relayout和渲染更新。


## 14. 在js中获取css样式


  {% highlight JavaScript %}
  $('x').style.foo
  {% endhighlight %}

`foo` 只在下列两种情况下是有效的：


  1. 它是在'x'的内联样式定义上设置的；
  2. 它已经在js中赋过值。


否则，`foo` 都是无效的。

## 15. DOM/HTML: 属性那是相当有用的


  {% highlight JavaScript %}
  <a href="javascript:showImg()">
    <img title="my image" name='sunset'>
  </a>
  {% endhighlight %}

比如，在一个简单的slide展示中，鼠标滑过一个链接时，需要根据一些信息在图片显示区域展示相应的一张图片。那么img标签的title和name属性就可以帮你很容易的达到目的。
当然，其它一些自定义的属性也可以使用的。


另外一个例子是：


  {% highlight HTML %}
  <input type=text pattern="^\w+$" required=true>
  {% endhighlight %}

页面加载时，JS代码可以遍历所有的表单字段，并且如果pattern和required属性存在的话，那么就添加一个validate函数，该函数主要负责在表单提交前，根据正则表达式来验证文本框中的内容。


## 16. 简单的动画

  {% highlight JavaScript %}
      ---| 
          -----| 
          --------| 
          -----------|

  {% endhighlight %}

一个进度条的简单动画实现：我们可以通过逐渐增加div或img的宽度来实现。

  {% highlight JavaScript %}
  function progress() { 
  var img = ..the img..
  if (img.width < 200) {
      img.width += 5;
      //to not autosize height along with width
      img.height = 5; 
  }
  else
      img.width = origwidth;
  }

  setInterval('progress()', 500);
  {% endhighlight %}

## 17. 简单的淡入/淡出效果

淡入/淡出效果一般用于从页面中添加/删除某个item时。

  {% highlight JavaScript %}
  function yellowFade(el) {
    var b = 155;
    function f() {
    el.style.background = 'rgb(255,255,'+ (b+=4) +')';
    if (b < 255) {
      setTimeout(f, 40);
    }
    };
    f();
  }
  {% endhighlight %}

可以参考这个[post](http://peter.michaux.ca/articles/javascript-animations-from-scratch "JavaScript Animations from Scratch")


## 18. 禁止某个资源的caching

  {% highlight JavaScript %}
  document.write("< img src='foo.jpg?" + Math.random() + "' />"); 
  {% endhighlight %}

我们修改了URL，添加了一个随机数，这是一种常用的手段，常常用于禁止图片的浏览器缓存。


## 19. DOM/HTML:常用的方法

参见[quirksmode](http://quirksmode.org/ "http://www.quirksmode.org/dom/w3c_core.html#gettingelements")。

  document.createElement('div')
    Creates a element node (div in this example)
  document.createTextNode()
    Creates a text node
  document.createDocumentFragment()
    Creates an empty DOM tree (useful for appending/creating elements too, and then transferring the entire tree to the actual document in one go)
      
  document.getElementById()
  document.getElementsByTagName("h1")
    Gets all tags of the specified type (h1 in this example). getElementsByTagName("*") gets all child elements of the specified elements/document. 
  element.getElementsByTagName("h1")
    Gets all tags of the specified type (h1 in this example) that are descendents of that element
  document.getElementsByClassName("myclass")
    Gets all elements that have the specified class name. This is not standard across browsers as far as I know.

    JS libraries like prototype, jquery etc., implement a full CSS2/CSS3 selector mechanism, so not only can we get elements by classname but many more CSS3 ways as well. 

### 节点属性

#### node.nodeType

1. Node.ELEMENT_NODE == 1
2. Node.ATTRIBUTE_NODE == 2
3. Node.TEXT_NODE == 3
4. Node.ENTITY_NODE == 6
5. Node.DOCUMENT_NODE == 9
6. Node.DOCUMENT_FRAGMENT_NODE == 11

#### node.nodeValue

value of text for a Text node (useful for #text). Null for most other nodes (including element nodes) devedge link 

#### node.nodeName

mainly useful for the element tag name or "#text" for text nodes.

1. Element ==> Element.tagName ("DIV", "H1", etc, UPPERCASE)
2. Text => "#text"

#### node.getAttribute()

Gets an attribute for a node, such as getAttribute("href") 

####　node.setAttribute()

Sets an attribute for a node, such as node.setAttribute("href", "http://bar.com");

### 节点方法

#### Finding nodes

1. Children: firstChild, lastChild, childNodes
2. Siblings: nextSibling, previousSibling
3. Parents: parentNode

#### Adding/Removing nodes

1. removeChild(), appendChild()
2. insertBefore(newnode, someOtherChild)

## 20. 从DOM树中移除当前的node

  {% highlight JavaScript %}
  var node = ..some_element..
  node.parentNode.removeChild(node);
  {% endhighlight %}

## 21. 在另外一个节点前插入一个节点

  {% highlight JavaScript %}
  var list; 
  var newnode;
  list.insertBefore(newnode, list.firstChild);
  {% endhighlight %}

## 22. 克隆一个节点（浅/深克隆）

  {% highlight JavaScript %}
  var node = ...
  var newnode = node.cloneNode(true|false); true=deep copy, else shallow
  {% endhighlight %}

## 23. 使用另外一个节点替换一个子节点

  {% highlight JavaScript %}
  var list;
  var newnode;
  list.replaceChild(newnode, list.oldchild)
  {% endhighlight %}
