---
layout: post
title: "Cross Domain Communication"
description: ""
category: 
tags: []
---
{% include JB/setup %}

本文主要介绍前端开发中常用的一些跨域方面的方法和技巧。
{: .countheads }

* ToC
{:toc}

### 什么是跨域交互

现在，稍稍大一点的网站开发，就要面对多域名的情况，如Active面向全球的主站https://www.active.com，以及面向不同产品的站点，如https://endurance.active.com/和https://camps.active.com等等。现在的域名安全标准是不允许某个域的**JavaScript**去访问和控制别外一个域的数据和对象的，包括变量，DOM等等，当然**Cookie**肯定也是不允许的。

显然，多域名架构下的各个站点并非完全独立的。常常由于应用的需要，不同域名的站点之间需要相互沟通信息和相互提供服务。然后，客户端浏览器却总是很小心谨慎地区分和处理不同域名之间的程序和数据，禁止它们之间互相操作，因为浏览器并不知道我的多个域名之间是有关联的，它只能把它们当成不同的网站，因为跨域操作地给访问者带来严重的安全性问题（主要是**Cookie**和**JavaScript**）。

严峻的客观事实告诉我们：安全的跨域通信确实是非常困难的！那么，我们应该怎么办呢？我们先谈谈常见的二种跨域类型。

跨域交互主要有二种类型：

* http://www.active.com和http://endurance.active.com
* http://www.active.com和http://www.baidu.com

对于这二种跨域交互的处理方法是不一样的，第一种是主域名和子域名之间的交互，第二种是两个完全不同的域名之间的交互。

### 常见的处理方式

* document.domain
* Iframe
* Hash
* Cookie
* postMessage
* window.name
* XMLHttpRequest

下面会分别简单介绍一下这些处理跨域交互方式的原理和实例。

#### document.domain

对于第一种主域名和子域名之间的交互，完全可以设置通过设置`document.domain`来保证两个不同域名下面的页面可以进行交互. 例如：

点击`iframe`页面中的按钮，可以设置父页面中元素`#msg`的内容. 如果两个页面没有设置`document.domain`，是无法访问`parent.document`属性的，`sendMessage`函数也就无法达到预期的功能.

{% highlight HTML %}

<html>
<body>
  <script type='text/javascript'>document.domain = 'active.com'; </script>
  <div id='msg'></div>
  <iframe src='http://endurance.active.com/index.html' style='width:100%; height:768px;'></iframe>
</body>
</html>

{% endhighlight %}

{% highlight HTML %}

<html>
<body>
  <script type='text/javascript'>
    document.domain = 'active.com';
    function sendMessage() {
      parent.document.getElementById('msg').innerHTML = new Date();
    }
    </script>
  <input type='button' onclick='sendMessage()' value='Sent Message'></body>
</html>

{% endhighlight %}

#### Iframe

**Iframe**是我们经常使用的一种跨域通信方式，一般是通过**Iframe**中的静态页面来实现交互的。从上面`document.domain`例子中可以看到，如果两个子域名之间的页面要交互的话，最简单的办法就是设置两个页面的`document.domain`为相同的主域，但是有时候考虑到**安全问题**，页面是无法设置`document.domain`的，设置了之后也有可能会影响到其它的功能。

#### Hash

但是如果是http://www.active.com和http://www.baidu.com，这是二个完全不同的地址，上面介绍的方法都不起作用了。
但是我们仍然可以通过一个静态页面传递参数，本质也就是通过**静态页面的hash来传递数据**。

测试前需要自己搭建一个测试用的多域名环境。首先需要在系统的**/etc/hosts**文件中添加二个静态域名解析项：

{% highlight Bash %}

127.0.0.1 active.com
127.0.0.1 passport.com

{% endhighlight %}

**active.com**是我们的主站，而**passport.com**主要用于验证用户身份。假如，**passport.com**所在域的**Cookie**保存了用户登录的状态，而**active.com**需要打开身体验证网站的一个页面，比如使用Iframe来加载，从而才有可能获取到身份认证的信息。但是，浏览器是不会允许我们这样做的。

> 当然，应用网站和身份网站的通信也可以通过应用网站的后台服务器直接与身份认证网站沟通，但是这种使用方式使用比较少，主要是现在服务器一般都是无状态的，所以比较困难。

接下来，我打算从**active.com**域模拟一个服务请求到**passport.com**域，输入参数`Hello World`，**passport.com**接着执行服务，给输入参数拼接一个`starandtina`，得到结果`Hello World starandtina`，然后送回原域**active.com**上显示。
本质上，我们需要的是**跨域通信**而不是**跨域访问**。我们只需要**passport.com**提供服务而已。

先编写一个[`http://active.com:3000/cdc_home.html`](https://raw.githubusercontent.com/starandtina/test/gh-pages/cdc_home.html)页面，它里面会用一个**Iframe**来加载**passport.com**域的页面。

{% highlight HTML %}

<html>
<head>
  <title>CDC Home Page</title>
</head>
<body style='background: #ccc;'>
  <span>Domain: active.com</span>
  <br />
  <span>请求参数：</span>
  <input type='text' id='param' value='Hello World' />
  <input type='button' value='Sent to External Domain' onclick='GoToAborad()' />
  <br />
  <iframe id='abroad' width='400px' height='250px'></iframe> <br />
  <span>最终结果：</span><input type='text' id='result' />
  <script type="text/javascript">
  var aboradFrame = document.querySelector('#abroad').contentWindow;
  var param = document.querySelector('#param');
  var result = document.querySelector('#result');

  function GoToAborad(){
    aboradFrame.location = 'http://passport.com:3000/cdc_abroad.html#' + param.value;
  }
  </script>
</body>
</html>

{% endhighlight %}

如上所示，页面上会有一个按钮点击用来加载外域页面，并且还会有一个接收最终返回结果的输入框。同时你也可以编辑传递给外域的消息，这个消息会以`Hash`的形式传递下去。

接下来需要编写一个**passport.com**域的服务页面[`http://passport.com:3000/cdc_abroad.html`](https://raw.githubusercontent.com/starandtina/test/gh-pages/cdc_abroad.html)，这个页面将使用`location.hash`来接收传递下来的参数数据，然后加以处理。

{% highlight HTML %}

<html>
<head>
  <title>外域框架</title>
</head>
<body style='background: red;'>
  <span>Domain: passport.com</span>
  <br />
  <span>收到参数：</span>
  <input type='text' id='param' value='Hello World' />
  <input type='button' value='Sent to Original Domain' onclick='GoToLocal()' />
  <br />
  <iframe id='local'></iframe>
  <br />
  <span>处理结果：</span>
  <input id='result' type='text' />
  <script type="text/javascript">
  var aboradFrame = document.querySelector('#local').contentWindow;
  var param = document.querySelector('#param');
  var result = document.querySelector('#result');

  param.value = location.hash.substring(1);
  result.value = param.value + ' starandtina';

  function GoToLocal(){
    aboradFrame.location = 'http://active.com:3000/cdc_local.html#' + result.value;
  }
  </script>
</body>
</html>

{% endhighlight %}

上面这个文件不是直接通过浏览器打开的，是我们在主页[`http://active.com:3000/cdc_home.html`](https://raw.githubusercontent.com/starandtina/test/gh-pages/cdc_local.html)中通过**Iframe**加载的。

接下的问题是，如果将处理结果回传至主页呢？

我们的做法是通过点击`Sent to Original Domain`按钮再将结果传递至另外一个属于原域的**Iframe**页面`http://active.com:3000/cdc_local.html`页面。

{% highlight HTML %}

<html>
<head>
  <title>本域框架</title>
</head>
<body style='background: #ccc;'>
  <span>Domain: active.com</span>
  <br />
  <span>收到结果：</span>
  <input type='text' id='result' />
  <input type='button' value='Go To Home' onclick='GoToHome()' />
  <br />
  <script type="text/javascript">
  var result = document.querySelector('#result');

   result.value = location.hash.substring(1);

  function GoToHome(){
    var home = parent.parent;

    home.document.querySelector('#result').value = result.value;
  }
  </script>
</body>
</html>

{% endhighlight %}

此时，我们在外域中又加载了与原域相同域下的一个页面，这个页面与我们的主域有相同的域，尽管中间隔了一个外域。

这个时候，我们通过什么与主域通信呢？相信你通过代码也已经看到了，就是`parent.parent`。虽然域的安全屏障是不允许外域在自己的地盘上捣乱的，但是却总是允许外域过境访问的。因此，不管**Iframe**嵌套层次有多深，其`parent`属性总是可以访问到的，同理，`parent.parent`也是可以访问到的，直到顶层。而当一个域的访问过境至自己域的领土时，它又有权力操作本域的任何元素，调用任何方法，同域的自家人总是相互信任的嘛！

因此，我们在`http://active.com:3000/cdc_local.html`页面中，用`parent.parent`就可以和主域通信。于是当你点击`Go To Home`按钮时，将在主页看到我们处理后的结果。

![CDC Hash](/assets/images/cdc_hash.png)
{: class='image-wrapper'}

就这样，主页的请求从一个域的时空穿越到了另外一个时空，最终又回到自己的时空，并带回了丰盛的成果，实现了跨域通信。

> 事实上，在这个跨域通信过程中，并没有必要嵌套两层**Iframe**，嵌入一个**Iframe**就足够了。因为在外域处理完请求后，如果没有必要保存当前状态，我们完全可以利用当前的环境回传，我们可以不必再创建一个新的**Iframe**，而是直接上当前Iframe跳转回原域即可。

到这里大家想到的可能是问题已经基本都解决了。**active.com**和**passport.com**之间都可以相互调用，还有什么不能处理的呢？其实事情到这里还没有完，如果两个页面要传递的数据量很大怎么办呢？而每个**URl**是有最大长度限制的，遇到越长参数时将会出现错误(一般来说，每个**Iframe**的`src`长度都限制在4096个字节）如果要传递的数据本身就是4096个字节长度，应该怎么办呢？可能有大家会想到多建立几个**Iframe**，每个**Iframe**传递部分数据过去，收到之后再组装一下不就可以了吗？

不错，我们就是采用这种办法. 这样子就可以认为，理论上可以传递任意长度的数据，加上**JSON**的支持，可以将对象序列化之后传递过去. 看起来是很不错的想法. 让我们看看是如何实现的吧。

如果我要传递的数据是**1234567890**，但是每个**Iframe**只能携带2个字节长度的数据，因此我们就需要建立5个**Iframe**来传递这个10个字节的数据。这样子对不对呢? 感觉看起来可以了，不过实际上是有问题的. 数据收到之后应该按照什么顺序在组装呢?因此还需要将分组之后的数据的顺序提供出来。对方收到就组装数据，但是总应该告诉对方总共需要有多少个分组需要组装吧. 因此，还需要将分组的数据传递过去。现在应该才是比较完美的解决方案，即便是第三个分组先到，那也没有问题，因此对方根据`s0002`可以知道将这个分组的数据放到什么位置. 等到接收到5个分组了，把接收到的数据拼接一下，就得到原始数据了.

{% highlight HTML %}

<iframe src="http://passport.com#msg=s0000&5&12"></iframe> 
<iframe src="http://passport.com#msg=s0001&5&34"></iframe> 
<iframe src="http://passport.com#msg=s0002&5&56"></iframe> 
<iframe src="http://passport.com#msg=s0003&5&78"></iframe> 
<iframe src="http://passport.com#msg=s0004&5&90"></iframe>

{% endhighlight %}

#### postMessage

上面介绍的方法虽然可行，但是太麻烦了，而且还要兼容不同的浏览器。Firefox6+，Chrome 1+，IE8+(部分支持)，Opera9.5+，Safari 4.0浏览器已经支持postMessage方法了，具体见[caniuse#postMessage](http://caniuse.com/#search=postMessage)。

具体用法和示例可以参见[MDN](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage)。

#### window.name

工作原理是在A中创建一个iframe，iframe的src指向B的一个文件，这个文件通过脚本设置window.name = data. data就是需要交互的数据. 等iframe加载完毕之后，A中的脚本修改iframe中contentWindow的location为A中的 一个空白页面，此时就可以获取window.name属性了。

![CDC window.name](/assets/images/cdc_window.name.png)
{: class='image-wrapper'}

这种方法的好处是B是无法获取到A的任何信息，`JavaScript`变量，`Cookie`，`DOM`等等，A唯一需要的是和B协商好接口和准备一个静态的空白页面。

一个典型的用例就是[Respond](https://github.com/scottjehl/Respond#cdnx-domain-setup)。
