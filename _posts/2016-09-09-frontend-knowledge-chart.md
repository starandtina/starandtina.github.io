---
layout: post
title: "Frontend Knowledge Chart"
description: ""
category: 
tags: []
---
{% include JB/setup %}

* ToC
{:toc}

Frontend Knowledge Chart

# HTML & CSS

## HTML常用的一些Tips

### Text

#### Paragraphs

大多数情况下我们都会用段落，它是HTML的重要组成部分，而在HTML中有一个元素就是为它而存在的：`<p>`。

>千万别使用`<br>`标签用于分开整篇数落，因为断行标签不是用于这种情况的。

**避免:**

{% highlight HTML %}
Cupcake ipsum dolor sit. Amet chupa chups chupa chups sesame snaps. Ice cream pie jelly
beans muffin donut marzipan oat cake.

<br>

Gummi bears tart cotton candy icing. Muffin bear claw carrot cake jelly jujubes pudding
chocolate cake cheesecake toffee.
{% endhighlight %}

**推荐:**

对于断行标签的一个合理使用场景是，举例来说，在诗或是歌曲的押韵处断开：

{% highlight HTML %}
<p>So close, no matter how far<br>
Couldn’t be much more from the hearth<br>
Forever trusting who we are<br>
And nothing else matters</p>
{% endhighlight %}

### Headings

标题标签，从`<h1>`到`<h6>`，其实是有一个暗含的层级关系的，从1（最重要）到6（不重要）逐层递减.

为了正确的处理语义，顺序的选择对于你来说正确的标题层级，而不是因为浏览器渲染标题时的大小而决定。你可以并且也应该用CSS来控制文本大小，同时选择一个合适的级别。

**避免**:

{% highlight HTML %}
<article>
    <h1>Monkey Island</h1>
    <h4>Look behind you! A three-headed monkey!</h4>
    <!-- ... -->
</article>
{% endhighlight %}

**推荐**:

{% highlight HTML %}
<article>
    <h1>Monkey Island</h1>
    <h2>Look behind you! A three-headed monkey!</h2>
    <!-- ... -->
</article>
{% endhighlight %}

另外需要考虑的一件事情就是如何创建与标题对应的子标题或是标语。W3C规范推荐使用普通的文本标签而是更低一层级的标题。

**避免**:

{% highlight HTML %}
<header>
    <h1>Star Wars VII</h1>
    <h2>The Force Awakens</h2>
</header>
{% endhighlight %}

**推荐**:

{% highlight HTML %}
<header>
    <h1>Star Wars VII</h1>
    <p>The Force Awakens</p>
</header>
{% endhighlight %}

### Forms

#### Placeholders

大家都知道表单元素`<input>`中有一个叫`placeholder`的属性，它规定描述文本区域预期值的简短提示，该提示会在文本区域为空时显示，当字段获得焦点时消失。很明显，`Placeholder`是针对为表单元素提供正确示例用的。

不幸的是，我们常常将这些`placeholder`当作`<label>`元素来使用，让别人知道这个表单元素是什么而不是做为一个合法输入值的一个示例。这种做法不支持[`accessibel`](https://developer.mozilla.org/en-US/docs/Web/Accessibility)，所以你应该避免它。

**避免：**

{% highlight HTML %}
<input type="email" placeholder="Your e-mail" name="mail">
{% endhighlight %}

**推荐：**

{% highlight HTML %}
<label>
    Your e-mail:
    <input type="email" placeholder="darth.vader@empire.gov" name="mail">
</label>
{% endhighlight %}

### 移动设备中的键盘

对于使用移动设备（如电话或是平板）浏览网站的用户来说，提供输入提示是至关重要的一个功能。只要我们能够给`<input>`元素选择正确的`type`，那么我们其实可以很轻易地就做到这一点。

举例来说，`type='number'`会让移动设备显示数字键盘而不是通常的字母键盘。对于`type='email'`或是`type='tel'`来说也是一样的道理。

**避免：**

{% highlight HTML %}
<label>Phone number: <input type="text" name="mobile"></label>
{% endhighlight %}

**推荐：**

{% highlight HTML %}
<label>Phone number: <input type="tel" name="mobile"></label>
{% endhighlight %}

下面是一张对比图：左边是当使用`type='text'`时的键盘显示；右边是`type='tel'`时的键盘显示。

![keyboard_compare.png](https://hacks.mozilla.org/files/2016/08/keyboard_compare.png)
{: class='image-wrapper'}

## Images

该说说**SVG**了吧！现在你不仅可以像下面这样在`<img>`标签中这样使用**SVG**：

{% highlight HTML %}
<img src="acolyte_cartoon.svg" alt="acolyte">
{% endhighlight %}

同时，你也可以在你的Web应用或是网站中使用**SVG sprites**实现矢量图标，而不是使用**Web Font**- 它一直是一种hack，有时可能你并不能得到你想要的结果。这是因为浏览器把**Web Font**当成文本对待，而不是图片。同时也有其它方面的问题，如广告拦截器会禁止下载**Web Font**。

>[Github从2016年2月23号开始已经不再支持**Icon Font**的Octicons](https://github.com/blog/2112-delivering-octicons-with-svg)。

如果你想了解更多，可以看看[Sarah Semark的这篇关于*为什么使用SVG的icons会比使用Web Font的更好*的演讲](http://wordpress.tv/2016/05/28/sarah-semark-stop-using-icon-fonts-love-svg/)。你也可以在[CSS-Tricks](https://css-tricks.com/svg-sprites-use-better-icon-fonts/)上读到更多关于这项技术的。

其实**SVG sprites**与[**CSS sprites**](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Images/Implementing_image_sprites_in_CSS)很相似。它也是把所有的SVG资源合并为一个图片文件。对于SVG来说，每一个资源是被包含在`<symbol>`标签中的，像下面这样：

{% highlight HTML %}
<svg>
    <symbol id="social-twitter" viewBox="...">
        <!-- actual image data here -->
    </symbol>
</svg>
{% endhighlight %}

之后，这个图标可以通过`<svg>`标签在你的HTML文件中使用，像下面这个例子一样，我们指向SVG文件中**symbol**的ID：

{% highlight HTML %}
<svg class="social-icon">
    <use xlink:href="icons.svg#social-twitter" />
</svg>
{% endhighlight %}

不得不承认，创建这些**SVG spritesheet**的过程是很枯燥的。好吧，这就是这些自动化工具，如[gulp-svgstore](https://github.com/w0rm/gulp-svgstore)存在的理由，它会自动化帮你做这些并根据你提供的单个的资源文件创建好一个CSS样式表。

记住一件事，既然我们开始使用`<svg>`而不是`<img`>标签来包含我们的图片资源，那么我们就可以使用CSS来应用样式。所有你可以对**Web Font icons**做的，同样你也可以对**SVG icons**做。

{% highlight CSS %}
.social-icon {
    fill: #000;
    transition: all 0.2s;
}

.social-icon:hover {
    fill: #00f;
}
{% endhighlight %}

但是，同时也有些CSS限制：当你像这样使用SVG时（通过`<use>`链接到`<symbol>`），那么图片是被注入到[Shadow DOM](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Shadow_DOM)，那么我们就失去了某些CSS的功能。在这种情况下，我们并不能知道SVG中的哪些元素我们需要应用样式，并且一些CSS属性（像fill）中会应到到原本这些属性值为`undefined`的元素上。但是，对于**Web Font icons**来说，你同样不能做到这些。

## 基础

包含了HTML以及CSS的基本知识。

### 知识点

- HTML常用元素。
- HTML元素间嵌套关系。
- HTML元素语义性。
- CSS常用属性。
- CSS选择器。
- CSS选择器优先级。
- 两类盒模型的概念。
- 常见布局的实现。

### 常见问题

- h1~h6的区别是什么？ul、ol的区别呢？

      <h1> - <h6> 标签可定义标题
      <h1> 定义最大的标题。<h6> 定义最小的标题
      <ol> 标签定义有序列表
      <ul> 标签定义无序列表

- 有用过dl、dt、dd这三个元素吗？其表达的语义是什么？

      <dl> 标签定义了定义列表（definition list）。
      <dl> 标签用于结合 <dt> （定义列表中的项目）和 <dd> （描述列表中的项目）。

- 如何实现定宽、自适应的两列、三列布局？

    - [two column with absolute positioned](http://jsfiddle.net/starandtina/w17j12hu/1/)
    - [two column with negative margins](http://jsfiddle.net/starandtina/w17j12hu/2/)
    - [two column with float](http://jsfiddle.net/starandtina/w17j12hu/3/)

    - [three column with absolute positioned](http://jsfiddle.net/starandtina/w17j12hu/1/)
    - [three column with negative margins](http://jsfiddle.net/starandtina/w17j12hu/5/)
    - [three column with float](http://jsfiddle.net/starandtina/w17j12hu/6/)
    - [three column with BFC](http://jsfiddle.net/starandtina/6xhcjzdm/)

- float属性的的作用是什么？如何清除浮动？

>float: 浮动的框可以向左或向右移动，直到它的外边缘碰到包含框或另一个浮动框的边框为止。由于浮动框不在文档的普通流中，所以文档的普通流中的块框表现得就像浮动框不存在一样。

那么如何清除浮动：

1. Clear: clear 属性规定元素的哪一侧不允许其他浮动元素;
2. 空的DIV: <div style="clear: both;"></div>;
3. BFC: 设置父元素 overflow: hidden或display: table触发BFC；
4. CSS pseudo selector (:after);
5. 使用 br标签和其自身的 html属性
6. ...

- [position属性的取值有哪些？各值的效果如何？](http://www.w3school.com.cn/css/css_positioning.asp)
  - static: 元素框正常生成。块级元素生成一个矩形框，作为文档流的一部分，行内元素则会创建一个或多个行框，置于其父元素中。
  - relative: 元素框偏移某个距离。元素仍保持其未定位前的形状，它原本所占的空间仍保留。
  - absolute: 元素框从文档流完全删除，并相对于其包含块定位。包含块可能是文档中的另一个元素或者是初始包含块。元素原先在正常文档流中所占的空间会关闭，就好像元素原来不存在一样。元素定位后生成一个块级框，而不论原来它在正常流中生成何种类型的框。
  - fixed: 元素框的表现类似于将 position 设置为 absolute，不过其包含块是视窗本身。
- [display属性的取值有哪些？各值的效果如何？](http://www.w3school.com.cn/cssref/pr_class_display.asp)
- display:inline-block的特点？

inline-block后的元素创建了一个行级的块容器，该元素内部（内容）被格式化成一个块元素，同时元素本身则被格式化成一个行内元素。

- [form元素的常用属性有哪些？](http://www.w3school.com.cn/tags/tag_form.asp)
- [input元素](http://www.w3school.com.cn/tags/att_input_type.asp)不给定type属性的情况下，会是什么样的？

如果没有type属性，默认会是`text`。

- input type="image"元素的作用是什么？

定义图像形式的提交按钮。

- 标准盒模型是怎么样的？怪异盒模型呢？

[Quirks mode and strict mode](http://www.quirksmode.org/css/quirksmode.html)

- 如何控制浏览器进入怪异模式或者标准模式？


我们知道`<!DOCTYPE>`声明对浏览器来说就是一个指令，它会告诉浏览器当前页面是用适用于哪一个版本的HTML。

什么是**doctype**: 

  - http://www.w3schools.com/tags/tag_doctype.asp
  - http://www.oreillynet.com/pub/a/network/2000/04/14/doctype/index.html
  - https://developer.mozilla.org/en-US/docs/Mozilla's_DOCTYPE_sniffing
  - https://developer.mozilla.org/en-US/docs/Mozilla_Quirks_Mode_Behavior
  - Browser comparison: which doctype triggers which mode?
  - Jukka K. Korpela’s Quirks Mode features page.

选择使用哪种模式是需要一个触发器的，而这个触发器就在**doctype switching**中。根据标准规范来说，任何一个(X)HTML文档必须有一个`doctype`，由它来告诉大家当前这个(X)HTML文档正在使用哪一种模式。


  - 在标准化实现之前的老页面是没有`doctype`的，或者是故意忽略它，因此**没有`doctype`**也就意味着怪异模式，它会根据旧的规则解析和显示页面
  - 相对应的，如果Web开发人员知道包含一个`doctype`，他或者知道自己在做什么。因为大多数的`doctype`会触发标准模式，也就是根据新的符合规范的规则来解析并显示页面
  - 任何新的或是未知的`doctype`会触发标准模式
  - 有一个问题是，有些页面确实是在怪异模式下但是却还是有`doctype`，因些每个浏览器都会有自己的一个会[触发怪异模式的`doctype`列表](https://hsivonen.fi/doctype/)

  

- 有使用过哪些CSS选择器？

    - CSS 元素选择器
    - CSS 选择器分组
    - CSS 类选择器详解
    - CSS ID 选择器详解
    - CSS 属性选择器详解
    - CSS 后代选择器
    - CSS 子元素选择器
    - CSS 相邻兄弟选择器
    - CSS 伪类
    - CSS 伪元素

## 兼容

考察包括Internet Explorer 6-8、Firefox 3.5+、Chrome 6+在内的各浏览器在HTML以及CSS方面的兼容性知识。

### 知识点

- HTML部分元素（如ul）在不同浏览器下的默认样式的差异。
- 部分HTML元素的使用上存在的差异，如引入flash的方式。
- 旧浏览器（如Internet Explorer 6）对部分CSS属性（如display:inline-block）支持程度。
- 部分CSS属性产生的问题（如双倍边距、3px边距等），以及如何修复。
- 旧浏览器不支持部分CSS属性（如position:fixed）的修复方案。
- 各浏览器使用的CSS Hack形式。

### 常见问题

- 有使用或制作过reset.css吗？请问该css中要包含哪些内容?

          reset.css 主要是用于重致浏览器的默认样式，在多个浏览器环境中保持一致性，以便可以方便地开发cross-browser应用并且让所有浏览器都能以最新的标准渲染所有元素。
          CSS 2.1 User Agent Style Sheet Defaults
          defaults for IE6 - IE9
          normalize.css

- 谈谈项目中遇到过的布局相关的兼容性问题，以及通过什么方式解决的？
- 浅谈一下IE中的hasLayout属性的作用，以及如何触发该属性。
- display:inline-block是一个非常好用的样式，但是IE6对其支持有限，能谈谈IE6是如何理解该样式的吗？如何在IE6下模拟出该样式的效果呢？
- 谈谈width和height样式在IE6下与其他浏览器的区别？
- 谈谈IE6对!imoprtant的样式声明的解析规则？
- 对于IE6不支持alpha透明的png的问题，是如何解决的？使用AlphaImageLoader会产生哪些问题？有没有解决的方法呢？
- 如何全浏览器兼容地设置某个元素的透明度为30%？


## 性能

考察页面重构的知识中与性能有关的各知识点，其考察以点状知识为主。

性能不仅仅是浏览器执行、解析的相关数值，也包括如何让用户更快地看到内容等方面的考量。

### 知识点

- CSS选择器的优化。
- HTML元素层面的优化。
- 流式布局与table布局之间的区别。


### 常见问题

- HTML元素到CSS选择器的匹配过程是如何进行的？
- 能否谈谈在制作CSS选择器时，要注意一些什么问题，保证性能的最优化？
- 为什么现在倡导流式布局，而不使用table布局？两者在性能方面是否有区别？

实践1

实现一个二级的菜单，其中一级菜单横向排列，当鼠标移动到一级菜单的某个菜单项上时，显示出该项关联的二级菜单。要求写出相关的HTML和CSS，并解释具体原理，并遵循以下规定：

- 不考虑浏览器兼容性问题，以现有最新标准为基础。
- 不得使用任何javascript或其他脚本，仅使用HTML和CSS。
- 二级菜单的展开方式使用动画可获得额外的认同。

实践2

实现一个3列布局，要求使用以下给出的HTML结构：

<div id="main"><!-- 内容 --></div>
<div id="nav"><!-- 内容 --></div>
<div id="sidebar"><!-- 内容 --></div>
<div id="footer"><!-- 内容 --></div>

并符合如下要求：

- body元素的padding和margin为0。
- #nav在左边，#main在中间，#sidebar在右边。
- #footer独立在下方，占满整行，其中内容居中。
- #nav宽度为15%，#sidebar宽度为20%，#main占用剩余空间。
- #main中的内容与#nav和#sidebar间隔10px。该条不要求兼容IE6。
- 不要求所有浏览器完全相同，但必须达到可用的程度。

http://jsfiddle.net/starandtina/f7kng0td/

# JavaScript及BOM、DOM

基础

考核javascript语言相关的基础知识、ECMA262标准相关知识，以及浏览器提供的BOM模型的知识，和w3标准下的DOM相关知识。

### 知识点

- javascript中的原生类型、内置类型。
- javascript提供的基本类型的相关函数。
- javascript中的对象类型的判定方式。
- 正则表达式相关知识。
- prototype的作用、类型创建、继承。
- ECMA基本语法，如分号插入规则、逗号运算符、括号的多义性等。
- BOM模型提供的对象，如location对象的组成、top/self/window/parent/opener的关系和区别等。
- DOM模型的相关函数、对象。
- DOM元素的创建、修改、删除等操作。
- 常用框架的熟悉程度。
- AJAX的运作过程。


### 常见问题

- javascript有哪些原生类型、哪些内置类型？null和undefined有什么区别？

      JavaScript 有 5 种原始类型（primitive type），即 Undefined、Null、Boolean、Number 和 String。
     undefined 与 null的历史由来

      undefined 与 null 区别：

    - null

        - null是一个对象，但是为空。因为是对象，所以 typeof null  返回 'object' 。
        - null 是 JavaScript 保留关键字。
        - null 参与数值运算时其值会自动转换为 0 ，因此，下列表达式计算后会得到正确的数值：

　　　　      表达式：123 + null　　　　结果值：123
　　　　      表达式：123 * null　　　　结果值：0

    - undefined

        - undefined是全局对象（window）的一个特殊属性，其值是未定义的。但 typeof undefined 返回 'undefined' 。
        - 虽然undefined是有特殊含义的，但它确实是一个属性，而且是全局对象（window）的属性。
        - 尽管undefined是有特殊含义的属性，但却不是JavaScript的保留关键字。
        - undefined参与任何数值计算时，其结果一定是NaN。
        - NaN是全局对象（window）的另一个特殊属性，Infinity也是。这些特殊属性都不是JavaScript的保留关键字！

提高undefined性能
　　当我们在程序中使用undefined值时，实际上使用的是window对象的undefined属性。
　　同样，当我们定义一个变量但未赋予其初始值，例如：
　　　　var aValue;
　　这时，JavaScript在所谓的预编译时会将其初始值设置为对window.undefined属性的引用，
　　于是，当我们将一个变量或值与undefined比较时，实际上是与window对象的undefined属性比较。这个比较过程中，JavaScript会搜索window对象名叫‘undefined'的属性，然后再比较两个操作数的引用指针是否相同。
　　由于window对象的属性值是非常多的，在每一次与undefined的比较中，搜索window对象的undefined属性都会花费时间。在需要频繁与undefined进行比较的函数中，这可能会是一个性能问题点。因此，在这种情况下，我们可以自行定义一个局部的undefined变量，来加快对undefined的比较速度。例如：

    function anyFunc()
    {
        var undefined;          //自定义局部undefined变量

        if(x == undefined)      //作用域上的引用比较


        while(y != undefined)   //作用域上的引用比较

    };

 　　其中，定义undefined局部变量时，其初始值会是对window.undefined属性值的引用。新定义的局部undefined变量存在与该函数的作用域上。在随后的比较操作中，JavaScript代码的书写方式没有任何的改变，但比较速度却很快。因为作用域上的变量数量会远远少于window对象的属性，搜索变量的速度会极大提高。
　　这就是许多前端JS框架为什么常常要自己定义一个局部undefined变量的原因！

- 当访问一个没有用var关键字定义过的对象时，会发生什么？

       - 在非strict模式下，没有使用使用var关键字定义变量，那么就相当于在全局对象上创建了新的属性，并不是变量，使用var是定义变量的唯一方法。
       - 查找identifier时，首先通过scope chain查找，一旦找到并且如果它是一个object，如果object.x，那么之后就会通过prototype chain查找x.

- Object、Array、RegExp的字面表达方式分别是什么？

       - Object: {}
       - Array: []
       - RegExp: //

- 如何判定一个对象是字符串？判断中的每一个分支在什么情况下成立？
- 请解释以下代码的返回，以及其中每个分支存在的原因：(function(){ return this || (1,eval)('this') })();。
- 请解释一下正则表达式中的3个标记位（g|y|m）的作用。
- 在javascript中，通过什么方法能创建一个“类”，以及创建一个“类”的“子类”呢？
- eval('var a = 3;');和(0, eval)('var a = 3;');有什么区别？
- document.defaultView是什么？
- 谈谈Node和Element的区别？Node有哪几种？
- 有哪些函数可以用来获取一个或多个DOM元素？
- 有用过jQuery吗？能说说jQuery的选择器有哪些吗？请问#section~ul>li:gt(0):not(:empty)这个选择器表示什么？

兼容

考察各浏览器中脚本编写时的兼容性处理，主要在于DOM相关属性的兼容性等。

### 知识点

- DOM元素各属性设置时的兼容性问题。
- DOM元素事件的兼容性。
- 事件注册、注销、触发，事件对象的兼容性。
- 原生数据类型的部分兼容问题，如for...in循环输出的顺序。


### 常见问题

- 请问如何给一个元素设置float:left样式？
- 如何取得一个元素的内联style属性的字符串，如'color:red;font-size:16px;'？
- 各浏览器是怎么注册、注销、触发事件的？addEventListener的第三个参数有什么作用？捕获和冒泡的概念？
- 事件Event对象在各浏览器下有哪些兼容性的问题，如何处理？
- delete关键字有什么作用？在各浏览器之下有什么差别吗？
- 如何判断浏览器是处在标准模式的呢？
- 有遇上过position:fixed的需求吗？你是如何检测一个浏览器是否支持该定位方式的？对于不支持的浏览器，有什么好的修复方案吗？
- 是否遇上过IE6下背景图片频繁闪烁的问题？能说说原因以及解决方案吗？
- 能谈谈UA检测和特性检测的概念吗？你是如何在两者之间进行取舍的？
- 如何让AJAX请求可以使用“后退”和“前进”按钮？

性能

考察在DOM操作等方面的性能测量、优化等知识，重在考察一个分析的过程，教条主义的结论并不适用。

### 知识点

- DOM操作的基本优化原则。
- reflow和repaint的概念，如何减少这两个操作。
- Sizzle等框架的选择器的优化。


### 常见问题

- 请问我们经常听到的“javascript”执行很慢通常是因为什么引起的？如何能进行针对性的优化？
- 能谈谈reflow和repaint这两个概念吗？
- 在编写javascript的时候，有什么关于性能方面需要注意的问题吗？
- 往往为了性能，会损失代码的优雅性和规范性，那么在这方面，你是怎么取舍的呢？
- 有用过Google Closure Compiler吗？对它的高级模式有没有什么研究？
- 如果让你设计一个类似Google Closure Compiler的编译工具，你认为可以对javascript进行怎么样的优化呢？

实践1

使用原生的javascript，实现一个ajax请求的函数：

function ajax(options) { };

要求options中含有以下内容：

url
请求的地址。
method
请求用的HTTP Method。
data
可选参数，发送请求时的数据。
success
可选参数，请求成功时的回调函数，形式：function(xhr) {}。
error
可选参数，请求失败时的回调函数，形式：function(xhr) {}。
codeXxx
可选参数，Xxx为一个数字，当响应中的HTTP状态码为Xxx时触发，形式：function(xhr) {}。

实践2

使用原生的javascript，实现一个函数：

function unique(array) { }

并符合以下要求：

- 当函数只有一个参数时，该参数为数组。
- 当函数有多个参数时，所有的参数合起来为一个数组。
- 函数去除数组中的重复元素，并返回一个仅包含相互不重复的元素的新的数组。
- 使用严格等于来判定两个元素是否重复。
- 考虑时间复杂度。

实践3

使用原生的javascript，实现一个获取某个名字的cookie值的函数：

function getCookie(name) { }

要求不存在名字为name的cookie项时，返回空字符串。

实践4

使用原生的javascript，实现一个jsonp函数：

function jsonp(url, callback) { }

仅要求处理服务器正确返回的情况，在服务器正确返回时，可调用callback函数。

同时，与服务器约定，url中可添加callback=xxx作为回调函数名称，服务器会根据此query值生成脚本。

其他部分

# HTTP协议

考查HTTP协议的认知程度和基本知识。

### 知识点

- HTTP协议的概念，HTTP请求、响应的组成。
- 常见HTTP Method表达的意思，不仅仅是GET和POST。
- HTTP状态码的表意区间，常见HTTP状态码表达的意义。
- 浏览器与服务器对cookie的交互方式。
- 浏览器对外部资源，如样式表、脚本文件、图片、flash的加载顺序和限制。
- 基于资源瀑布图的分析和优化。


### 常见问题

- 请说说HTTP协议中的请求及响应的组成部分。
- 请简单阐述一下，当使用一个input type="file"进行文件上传时，怎么从请求流中找出该文件的内容和相关信息？
- 请指出HTTP状态码中的200、302、304、404、500、503表示的意义。
- HTTP状态码301和302有什么区别？
- 在HTTP头中，关于缓存的有哪几个？分别起什么作用？
- 在一次AJAX请求中，怎么判断请求已经结束，并且服务器成功执行了请求并给出了正确的响应？
- 为什么使用CTRL+F5刷新浏览器可以避免本地缓存呢？
- 服务器是如何得到浏览器中的cookie的？又如何向浏览器设置某个cookie呢？
- 为什么要将样式放在关部，而将脚本放在页面底部呢？
- 将一个外部样式表通过link元素引入，则该元素在“页面的head的底部”与“页面的body的头部”有什么区别？

# 安全相关

考查前端及系统安全方面的知识，需要对各种攻击手段和防范措施有基本的了解。

### 知识点

- HTML转义符号、脚本转义符号。
- 常见的XSS漏洞，及其避免方式。
- 常见的其他安全攻击方式，及其应对措施。
- 与flash交互过程中的安全事项。


### 常见问题

- 有一段用户输入的文字，现在要通过一定的方法显示在一个id为output的div元素中，请问如何做？
- 请问什么是XSS漏洞？从你的角度看，HTML片段中有哪些关键字可能导致输出后会执行一段javascript脚本？
- 有没有听过中间人攻击？请问对于中间人攻击，有没有什么有效的防范措施呢？
- 如何防止flash在页面中执行javascript呢？

新技术

考核对HTML5及CSS3、Javascript 1.6的相关知识的吸收程度，借以考察对前端技术的关注度及兴趣。

### 知识点

- HTML5新增元素的语义。
- HTML5表单的增加。
- HTML5中Geolocation、LocalStorage、IndexDB、WebSQL、WebSocket等技术的概念。
- CSS3的选择器。
- box-shadow、text-shadow等CSS3新增样式。
- CSS3 Transform和CSS3 Animation的应用。
- dataURI的概念。
- javascript 1.6的新增功能。


### 常见问题

- 对HTML5有所了解吗？能说说HTML5新增加的元素有哪一些吗？
- 能谈谈就你来看，HTML5的意义以及其设计的原则吗？
- HTML5中对表单进行了不少的增强，能谈谈你对这方面的了解吗？
- HTML5扩展了不少浏览器对象，包括地理位置、本地存储等，能谈谈你的了解吗？
- 有了解过box-shadow这个样式吗？请问box-shadow设置的阴影处于标准盒模型的哪一层中？
- 现在需要在页面上画一个直径为30px的正圆形，请例举所有可能的方案。
- CSS3中你最喜欢的特性是什么？为什么呢？
- 能说说dataURI的概念吗？其具体的编码格式是什么？你认为怎么样的情况下适于使用dataURI？
- 使用javascript 1.6，是否可以给对象定义一个属性，并且使得这个属性无法被for...in语句枚举出来？
- 有接触过javascript的strict模式吗？请问怎么进入strict模式，strict模式与普通模式有什么区别？我们为什么要使用strict模式呢？

实践1

现有一个页面的代码如下：

<!DOCTYPE html>
<html>
    <head>
        <!-- head内容 -->
    </head>
    <body>
        <p id="message"><%= Request.QueryString["errorMessage"] %></p>
    </body>
</html>

该系统使用Session存放登录用户的信息，使用cookie中的session_id项来对应服务器的Session。

请设计一个攻击方案，使攻击者可以在不使用社会工程学的前提下，获得用户的帐号并成功登录到系统中。

实践2

现有一个简单的购物系统，有以下数据结构：

- Product : { id, category, name, price }
- User : { id, name, password }
- BuyRecord : { productId, userId, price, time }

有以下的功能需求：

- 用户可以登录。
- 可以按照category浏览Product。
- 可以按照id查看单个Product的详细信息。
- 可以购买单个Product，此操作会形成一个BuyRecord。
- 可以查看自己的BuyRecord。

请根据上面的信息，请设计以下功能的url结构以及相应的HTTP Method，url中的参数使用{xxx}的形式表示：

功能

Method

URL

用户登录界面

用户登录提交地址

根据category查看Product列表

查看Product详细信息

购买Product

用户查看自己的BuyRecord

要求尽可能使每个URL语义化，便于用户的记忆和分享。
