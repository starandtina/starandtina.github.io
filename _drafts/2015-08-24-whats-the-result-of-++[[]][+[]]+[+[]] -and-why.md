---
layout: post
title: "What's the result of ++[[]][+[]]+[+[]] and why?"
description: ""
category: 
tags: []
---
{% include JB/setup %}

最近在[Quora](http://www.quora.com/In-JavaScript-why-is-++-+-+-+-10-true)上看到有人在问为什么下面的执行结果为`true`？

{% highlight JavaScript %}

++[[]][+[]]+[+[]] === '10'

{% endhighlight %}

个人觉得这个题目很有意思，跟[ECMAScript](http://es5.github.io/)底层原理很有关系，所以趁机回顾一下，把它给讲个简单明白。
{: .countheads }

* ToC
{:toc}

首先`++[[]][+[]]+[+[]]`会让人看得一头雾水，为了简单起见，我们先把它分成二部分，逐个击破，如下所示：

{% highlight JavaScript %}

(++[[]][+[]]) + ([+[]])

{% endhighlight %}

### [+[]](#section1)

我们先看看右侧部分`+[]`。它的运算步骤如下：

1. 让`expr`的值为[]
2. 在`expr`上调用`GetValue`方法，它会返回它所指向的对象，即`expr`
3. 在第二步的结果上调用`ToNumber($2)`
4. `ToNumber`会调用`ToPrimitive`，并传入`Number`作为`hint`值，即`ToPrimitive(input argument, hint Number)`
5. 紧接着，`ToPrimitive`会调用对象的内部方法`[[DefaultValue]]`并带上`Number`作为`hint`值，即`[[DefaultValue]] (hint)`
6. 由于我们传入的`hint`为`Number`，那么`[[DefaultValue]]`首先会先调用对象上的`valueOf`方法，并将当前对象作为`this`
7. 如果第六步的中结果为原生数据类型，那么就把该值赋予`primValue`，否则会调用对象上的`toString`方法，并将当前对象作为`this`
8. 如果第七步的中结果为原生数据类型，那么就把该值赋予`primValue`，否则抛出`TypeError`异常
9. 调用`ToNumber(primValue)`并返回

虽然上面步骤很多，但是一目了然，但是，这里仍然需要对`ToPrimitive`这个核心函数多做一下解释。`ToPrimitive`主要是用于将任意类型的值转换为相应的原生类型的值。如果`ToPrimitive`函数的输入已经是原生类型值，那么该值会原样返回，不会做出任何类型转换。但是，如果该输入值是非原生数据类型，那么它就会通过调用内部方法`[[DefaultValue]]`以便找到与该对象相对应的默认值。

`[[DefaultValue]]`方法是每一个对象都有的一个内部方法。`[[DefaultValue]]`可以接受一个可选参数`hint`，`hint`必须是`Number`或是`String`。如果`hint`可选参数没有提供，那么它的默认值就是`Number`。但是这里有一个例外，如果对象是`Date`类型的，那么`hint`的默认值就会是`String`。 如果`hint`参数已经确定好了，它会根据`hint`参数依次选择调用`toString`和`valueOf`，直到找到该对象的原生数据类型值。这就是`hint`起作用的地方。如果`hint`为`Number`，那么`valueOf`方法会首先被调用，但是如果它是`String`，那么`toString`方法会首先被调用。运行过程如下：

1. 如果第一个方法存在，且是`callable`，那么就执行它并返回它的执行结果，否则跳到第三步
2. 如果第一步的结果是原生数据类型值，那么直接返回它
3. 如果第二个方法存在，且是`callable`，那么就执行它并返回它的执行结果，否则跳到第五步
4. 如果第三步的结果是原生数据类型值，那么直接返回它
5. 抛出`TypeError`异常

由上我们可以看出，`[[DefaultValue]]`的返回结果必定是原生数据类型，如若不是，则会抛出`TypeError`异常。

所以，对于`+[]`来说，由于先调用`[].valueOf`方法时返回的仍然是`[]`，所以接着会再调用`[].toString`，此时返回的是空空字符串`''`，然后再调用`ToNumber('')`，返回`0`。

那么右侧部分`[+[]]`的结果就是`[0]`。

### ++[[]][+[]]

从前面对右侧的分析，我们已经知道了如下的结论：

{% highlight JavaScript %}

[+[]] // [0]

{% endhighlight %}

那么，`++[[]][+[]]`就等于`++[[]][0]`。我们再来看看`++[[]][0]`是如何一步一步求值的。

1. 由于`[]`优先级最高，所以会先执行右侧的Property Accessors[`[[]][0]`](http://es5.github.io/#x11.2.1)
2. 第一步执行返回结果是类型为[`Reference`](http://es5.github.io/#x8.7)的空数组，它的`base`是`[[]]`，而它的`referenced name`是`0`
3. 在第二步的结果上执行[`Prefix Increment Operator`](http://es5.github.io/#x11.4.4)，即`++`操作

我们假设`[[]][0]`的执行结果为`v`，那么`++`操作一般执行步骤如下：

1. 让`expr`的值为`v`
2. 在`expr`上调用`GetValue`方法，它会返回它所指向的对象，即`v`，并将它赋予`oldValue`
3. 在`oldValue`的基础上加1，即[`oldValue + 1`](http://es5.github.io/#x11.6.1)，并将结果赋予`newValue`
4. 调用`PutValue(v, newValue)`
5. 返回`newValue`

在这里，我们使用`+`操作符来执行`oldValue + 1`操作，默认它会首先将`oldValue`转换为原生数据类型再执行`+`操作。由[+[]](#section1)可得出如下结论：

{% highlight JavaScript %}

oldValue     // 0
oldValue + 1 // 1

{% endhighlight %}

[`+`操作的运算步骤](#The-Addition-Operator)如下：

1. 先计算左侧值
2. 再计算右侧值
3. 在第一步和第二步的结果上调用`ToPrimitive`，且没有`hint`值
4. 如果第三步中任何一个结果是`String`类型，则跳到第七步
5. 在第三步的结果上分别调用`ToNumber`方法
6. 返回第五步中两者结果的和
7. 在第三步结果的基础上分别调用`ToString`方法
8. 拼接第七步结果返回的两个字符串值并返回

### ++[[]][+[]]+[+[]]

现在我们已经得到了左侧和右侧的值，从而就简化为了如下：

{% highlight JavaScript %}

1 + [0]

{% endhighlight %}

从而我们按照[`+`操作的运算步骤](#The-Addition-Operator)进行计算，由于我们没有传入`hint`值，所以`hint`值默认为`Number`（如果它是`Date`类型，则为`String`）。这也就意味着在`[0]`上`valueOf`方法会先调用，而不是`toString`方法。由于`[0].valueOf`方法返回的仍然是`[0]`，它并不是原生数据类型，所以接着调用`[0].toString`方法，得到结果`'0'`，那么就变为了：

{% highlight JavaScript %}

1 + '0'

{% endhighlight %}

由于`0`是`String`，则会执行字符串拼接操作，即先将`+`两侧的操作数转换为`String`，然后再返回两个字符串拼接的值，即`10`。   

