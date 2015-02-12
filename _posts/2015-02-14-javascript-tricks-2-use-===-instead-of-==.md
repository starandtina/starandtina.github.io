---
layout: post
title: 'JavaScript tricks 2: Use === instead of =='
description: ""
category: ['frontend', 'JavaScript', 'tricks']
tags: ['Frontend', 'JavaScript', 'Tricks', 'Best Practices']
---
{% include JB/setup %}

本文是关于**JavaScript**的陷阱和最佳实践中的一部分。

## 尽量使用`===`而不是`==`
{: .countheads }

* ToC
{:toc}

### `===` && `==`

在`JavaScript`中，有二种判断值是否相等的等值运算符：

* [严格判等运算符](http://es5.github.io/#x11.9.4)：只考虑类型相同的值是否相等，如果不同，直接返回`false`
* [普通判等运算符](http://es5.github.io/#x11.9.1)：在做相等比较前，会进行一系列的类型转换

#### 严格判等运算符`===`

[ES5 11.9.6](http://es5.github.io/#x11.9.6)描述了`===`的具体算法。我们以`x === y`为例：

1. 首先，如果`x`和`y`的类型不同，返回`false`
1. 如果类型为`Undefined`或`Null`，返回`true`
1. 如果类型是`Number`
    1. 如果`x`或`y`为`NaN`，返回`false`
    1. 如果`x`和`y`的数值相等，返回`true`
    1. 如果`x`为`+0`且`y`为`-0`，返回`true`，反之亦然
    1. 其它情况返回`false`
1. 如果类型为`String`，那么很明显，必须两者包含相同的字符串才会返回`true`，否则返回`false`
1. 如果类型为`Number`，与`String`类型情况一样，必须两者同时为`true`或`false`才会返回`true`，否则返回`false`
1. 如果`x`和`y`两者都指向同一个对象时，返回`true`，否则返回`false`

{% highlight JavaScript %}

NaN === NaN // false
+0 === -0 // true

var foo = function() {}
foo === foo // true

{% endhighlight %}

>注意：`NaN`并不等于它自己，即`NaN === NaN`返回`false`。所以我们必须使用其它方法来判断某个值是否为`NaN`。

#### 普通判等运算符`==`

[ES5 11.9.3](http://es5.github.io/#x11.9.3)描述了`==`的具体算法。我们以`x == y`为例：

1. 如果`x`和`y`的类型相同，则使用[`===`算法](http://es5.github.io/#x11.9.6)比对
1. 如果`x`为`null`且`y`为`undefined`，则返回`true`，反之亦然
1. 如果其中一个是`Number`类型，另外一个是`String`类型，那么需要先把`String`类型的转换为`Number`，然后再执行`==`
1. 如果其中一个是`Boolean`类型，另外一个是非`Boolean`类型，那么需要先把`Boolean`类型转换为`Number`类型，然后再执行`==`
1. 如果其中一个是`String`或是`Number`类型，另外一个是`Object`类型，那么需要先把`Object`类型转换为**primitive**的类型，然后再执行`==`
1. 其它情况返回`false`

{% highlight JavaScript %}

undefined == null // true

false == 0 // true
true == 1 // true
true == '1' // true
true == 'foo' // false

'' == 0  //true
'  ' == 0 // true

'foo' == new String('foo') // true

[] == 0 // true

'abc' == new String('abc') // true

{% endhighlight %}

### `==`带来的问题

从上面的分析，我们可以看到如果使用`==`，可能会带来一些问题：

1. `==`做判等时，如果两者类型不同，则需要做大量的类型转换，而这些类型转换规则并不是一眼就能看出来的，如果你稍不留意，结果就可能与你想像的大不相同
1. 由于`==`可以允许任意的类型做判断，所以一些类型错误就有可能难以发现

比如：

{% highlight JavaScript %}

true == 1 // true
true == 2 // false 与常识相反

true == '1' // true
true == '2' // false 与常识相反

0 == '' // true
'\r\n\t 111 \t' == 111 // true

{% endhighlight %}

`true == 2`为`true`是因为`true`会先被转化为**1**，然后再执行`1 == 2`，所以返回`false`。
大家可能会觉得最后一个很奇怪，为什么上面代码中最后一个例子`'\r\n\t 111 \t' == 111`会返回`true`，那是因为`'\r\n\t 111 \t'`在执行`==`前，必须先转换为`Number`，而这样会[删除掉数值前后的空白符](#[1])。

### 什么时候可以使用`==`

我们经常给**JavaScript**初学者的其中一个建议就是尽量使用`===`而不是`==`，毫无疑问，这是完全正确的，特别是对一个初学者来说。但是有些情况下，我们可能会觉得使用`==`更合乎常理，但是他们其实不是看上去那样美的。

>虽然你的代码只写了一次，但是它会被你和其他人读多次，所以代码的可读性是非常重要的。

#### 与`undefined`和`null`比较

如果你看完了上面的分析，那么你就会知道，使用`==`时，`undefined`和`null`之间是互等的，但是与其它值判等时都是`false`。所以，我们经常会用下在的代码来判断一个值是否为`undefined`或`null`：

{% highlight JavaScript %}

if (foo == null) {
  ...
}

{% endhighlight %}

乍一看，这样判断代码简短，但是可读性很弱，无法完全表达作者原有的意思。如果一个**JavaScript**初学者看到这段代码，他可能会以为你只是在检测`null`，但是如果是一个经验丰富的前端工程师的话，他可能会认为你写错，其实你是想用`===`。如下所示：

{% highlight JavaScript %}

if (foo === undefined || foo === null) { 
  ... 
}

{% endhighlight %}

如果你还想更精简一点，那么可以这样写：

{% highlight JavaScript %}

if (!foo) {
  ....
}

{% endhighlight %}

>在**JavaScript**中, `false`, `null`, `0`, `''`, `undefined`和`NaN`都被当做`false`值。

#### 比较字符串和数字

在处理与后端交互的**Ajax**请求返回结果时，我们经常需要去处理返回的以字符串形式展现的数值，如`id: '111111'`。这个时候，我们可能经常会这样去做比较：

{% highlight JavaScript %}

if (id == 111111) {
  ....
}

{% endhighlight %}

这个时候，我们是去检查`id`是`111111`或是`'111111'`。
但是，如果别人来看你的代码时，他会认为`id`是`Number`类型的。那么，为什么不直接告诉读你代码的人它是我们需要的是`Number`类型的呢？如果不是，我们需要先进行显示的类型转换。

{% highlight JavaScript %}

if (Number(id) == 111111) {
  ....
}

{% endhighlight %}

#### 比较对象和原生数据类型

由于`==`支持类型不同的值的比较的，所以你也用它来比较对象和原生数据类型值。

{% highlight JavaScript %}

true == new Boolean(true)

{% endhighlight %}

如果你看到下面的这段代码，你会认为写这段代码的人是想做什么？

{% highlight JavaScript %}

str == 'foo'

{% endhighlight %}

* `str`可能是一个`String`类型的包装对象，你是想让一个对象和字符串进行比较吗？如果是，你得了解这个对象是[如何转换为原生数据类型的](#[2])。
* 在比较之前你想让`str`先转换为字符串吗？如果是，你可以让你的代码更清楚点

{% highlight JavaScript %}

String(str) == 'foo'

{% endhighlight %}

* 又或如果`str`是一个`String`类型的包装对象，你想得到对象的原始值，然后再做比较？

{% highlight JavaScript %}

str.valueOf() == 'foo'

{% endhighlight %}

#### 如果你知道你比较的对象类型

比如，当我们[以函数的形式调用`String`构造函数](#[3])时，它会自动完成类型转换，如下代码所示：

{% highlight JavaScript %}

if (String(foo) == 'foo') {
  ...
}

{% endhighlight %}

你知道`String(foo)`返回的肯定会是`String`类型，所以这里使用`==`是没有任何问题的，因为我们可以保证这里不可能会有任何的类型转换的。
但是，我们仍然可以不用这样做：

* 一致性：在这里使用`==`，并没有给我们带来任何收益，那么我们为什么不使用更简单的`===`呢?
* 简单性和性能：一般来说，`===`是判断是否相等最简单的方法，那是因为它并不会对它的操作数进行类型转换。虽然性能测试结果在所有的**JavaScript**引擎中不尽相同，但对大多数浏览器来说，[`===`与`==`性能差不多，甚至更快](#[4])。

### 总结

虽然许多种情况下，看似使用`==`不伤大雅，但是*不使用`==`*已经是一条不成文的规则，不仅仅是对新手而已。

>代码中少一些花哨，那么就更容易理解和维护。

### 参考
{: #references }

1. {: id='[1]' }[ECMAScript 5: ToNumber](http://es5.github.io/#x9.3.1)
2. {: id='[2]' }[ECMAScript 5: ToPrimitive](http://es5.github.io/#x9.1)
3. {: id='[3]' }[The String Constructor Called as a Function](http://es5.github.io/#x15.5.1)
4. {: id='[4]' }[JSPerf: Equality vs Strict Equality](http://jsperf.com/equality-vs-strict-equality)
