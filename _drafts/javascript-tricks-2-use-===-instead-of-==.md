---
layout: post
title: 'JavaScript tricks 2: Use === instead of =='
description: ""
category: ['frontend', 'JavaScript', 'tricks']
tags: ['Frontend', 'JavaScript', 'Tricks', 'Best Practices']
---
{% include JB/setup %}

本文是关于**JavaScript**的一些陷阱和最佳实践中的一部分。

# 尽量使用`===`而不是`==`

## `===` && `==`

在`JavaScript`中，有二种比较的方式：

* [严格判等运算符](http://es5.github.io/#x11.9.4)：只考虑类型相同的值是否相等，如果不同，直接返回`false`
* [普通判等运算符](http://es5.github.io/#x11.9.1)：在做相等比较前，会进行一系列的类型转换

### 严格判等运算符`===`

[ES5 11.9.6](http://es5.github.io/#x11.9.6)描述了它的具体比较算法。以`x === y`为例：

1. 首先，如果`x`和`y`的类型不同，则直接返回`false`
2. `undefined` === `undefined`
3. `null` === `null`
4. 如果类型是`Number`
    1. 如果`x`或`y`为`NaN`，则返回`false`
    2. 如果`x`和`y`的值相等，则返回`true`
    3. 如果`x`为`+0`且`y`为`-0`，则返回`false`，反之亦然
    4. 其它情况返回`false`
5. 如果类型为`String`，那么很明显，必须两者包含相同的字符串才会返回`true`，否则返回`false`
6. 如果类型为`Number`，与`String`类型情况一样，必须两者同时为`true`或`false`才会返回`true`，否则返回`false`
7. 如果两者都是`object`（包括数组，函数），那么只有两者都指向同一个对象时，才会返回`true`，否则返回`false`


{% highlight JavaScript %}

NaN === NaN // false
+0 === -0 // true

var foo = function() {}
foo === foo // true

{% endhighlight %}

### 普通判等运算符`==`

[ES5 11.9.3](http://es5.github.io/#x11.9.3)描述了它的具体比较算法。以`x == y`为例：

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

{% endhighlight %}

### `==`会带来的问题

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
''

{% endhighlight %}

`true == 2`为`true`是因为`true`会先被转化为**1**，然后再执行`1 == 2`，所以返回  

## 什么时候可以使用`==`
