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

个人觉得这个题目很有意思，跟[ECMAScript](http://es5.github.io/)底层实现原理有关系，值得趁机再次温故而知新，希望能把它给讲个简单明白。
{: .countheads }

* ToC
{:toc}

如果你是第一次看到如下的表达式，可能首先会让你觉得一头雾水，无从下手。

{% highlight JavaScript %}

++[[]][+[]]+[+[]]

{% endhighlight %}

所以，为了简单清楚起见，我们可以先把它分成二部分，然后逐个击破，如下所示：

{% highlight JavaScript %}

(++[[]][+[]]) + ([+[]])

(++[[]][+[]]) // Left side

([+[]])      // Right side

{% endhighlight %}

### [+[]](#expression-right)

首先，我们先来看看右侧部分`+[]`。总的来说，它的运算步骤如下：

1. 把`[]`赋予`expr`临时变量
2. 在`expr`上调用[`GetValue`](http://es5.github.io/#x8.7.1)方法，它会返回它所指向的对象，在这里就是`expr`
3. 在第二步的执行结果上调用`ToNumber($2)`
4. 之后，`ToNumber`会调用`ToPrimitive`，并传入`Number`作为`hint`参数值，即`ToPrimitive(input argument, hint Number)`
5. 紧接着，`ToPrimitive`会调用`expr`对象的内部方法`[[DefaultValue]]`并带上`Number`作为`hint`值，即`[[DefaultValue]] (hint)`
6. 由于我们传入的`hint`为`Number`，那么`[[DefaultValue]]`首先会先调用`expr`对象上的`valueOf`方法，并将当前`expr`对象作为`this`值
7. 如果第六步的中结果为原生数据类型，那么就把该值赋予`primValue`临时变量，否则会继续调用`expr`对象上的`toString`方法，并将当前对象作为`this`值
8. 如果第七步的中结果为原生数据类型，那么就把该值赋予`primValue`临时变量，否则抛出`TypeError`异常
9. 调用`ToNumber(primValue)`并返回结果

虽然上面步骤很多，但是一目了然，再清楚不过了。

但是，这里仍然需要对`ToPrimitive`这个核心函数多做一下解释。`ToPrimitive`主要是用于将任意类型的值转换为对应的原生类型的值。如果`ToPrimitive`函数的输入已经是原生类型值，那么会原样返回该值，不会做任何类型转换。但是，如果该输入值是非原生数据类型，那么它就会通过调用内部方法`[[DefaultValue]]`以便找到与该对象相对应的默认值。

其中，`[[DefaultValue]]`方法是每一个对象都有的一个内部方法。`[[DefaultValue]]`可以接受一个可选参数`hint`，`hint`必须是`Number`或是`String`。

> 如果可选参数`hint`没有提供，那么它的默认值即是`Number`。但是这里有一个例外，如果对象是`Date`类型的，那么`hint`的默认值就会是`String`。 

如果可选参数`hint`已经确定好了，那么`[[DefaultValue]]`会根据`hint`参数依次选择调用`toString`和`valueOf`，直到找到该对象的原生数据类型值为止，这里就是我们为什么需要`hint`的原因，因为它决定了调用这二个方法的顺序。如果`hint`为`Number`，那么`valueOf`方法会首先被调用，但是如果它是`String`，那么`toString`方法会首先被调用。运行过程如下：

1. 如果第一个方法存在，且是`callable`，那么就执行它并返回它的执行结果，否则跳到第三步
2. 如果第一步的结果是原生数据类型值，那么直接返回它
3. 如果第二个方法存在，且是`callable`，那么就执行它并返回它的执行结果，否则跳到第五步
4. 如果第三步的结果是原生数据类型值，那么直接返回它
5. 抛出`TypeError`异常

由上我们可以看出，`[[DefaultValue]]`的返回结果必定会是原生数据类型，如若不是，则会抛出`TypeError`异常。

所以，对于`+[]`来说，首先会调用`[].valueOf`方法，它的返回仍然是`[]`，由于它不是原生数据类型，所以会接着再调用`[].toString`，此时返回的是空空字符串`''`，之后再调用`ToNumber('')`，即[`+[]`计算的第九步](#expression-right)，最后返回运算结果`0`。

那么，到此为止，右侧部分`[+[]]`的结果就是`[0]`。

### ++[[]][+[]](#expression-left)

从前面对右侧表达式`[+[]]`的分析，我们已经知道了如下的结论：

{% highlight JavaScript %}

[+[]] // [0]

{% endhighlight %}

那么，`++[[]][+[]]`就可以进一步拆解为`++[[]][0]`。

我们再来看看`++[[]][0]`是如何一步一步求值的：

1. 由于`[]`优先级最高，所以会先执行右侧的[Property Accessors](http://es5.github.io/#x11.2.1)，即`[[]][0]`
2. 第一步执行的返回结果是类型为[`Reference`](http://es5.github.io/#x8.7)的空数组，它的`base`是`[[]]`，而它的`referenced name`是`0`
3. 在第二步的结果上执行[`Prefix Increment Operator`](http://es5.github.io/#x11.4.4)，即`++`操作

我们假设`[[]][0]`的执行结果为`v`，那么`++`操作的执行步骤如下：

1. 把`v`赋予临时变量`expr`
2. 在`expr`上调用`GetValue`方法，它会返回它所指向的对象，即`v`
3. 在第二步执行结果的基础上调用`ToNumber($2)`，并将它赋予`oldValue`
3. 在`oldValue`的基础上加1，即[`oldValue + 1`](http://es5.github.io/#x11.6.1)，并将结果赋予`newValue`
4. 调用`PutValue(v, newValue)`
5. 最后返回`newValue`

在这里，我们使用`+`操作符来执行`oldValue + 1`操作，默认它会首先将`oldValue`转换为原生数据类型再执行`+`操作。由于`oldValue`的值为`0`，本身已经是原生数据类型了，所以此处不会做类型转换操作。

{% highlight JavaScript %}

oldValue     // 0
oldValue + 1 // 

{% endhighlight %}

### ++[[]][+[]]+[+[]]

现在我们已经得到了左侧和右侧的值，从而就简化为了如下表达式求值：

{% highlight JavaScript %}

1 + [0]

{% endhighlight %}

我们先说说[`+`操作符的运算步骤](#The-Addition-Operator)：

1. 先计算左侧值
2. 再计算右侧值
3. 在第一步和第二步的结果上调用`ToPrimitive`，且没有`hint`值
4. 如果第三步中任何一个结果是`String`类型，则跳到第七步
5. 在第三步的结果上分别调用`ToNumber`方法
6. 返回第五步中两者结果的和
7. 在第三步结果的基础上分别调用`ToString`方法
8. 拼接第七步结果返回的两个字符串值并返回

从而我们按照[`+`操作符的运算步骤](#The-Addition-Operator)进行计算，由于我们没有传入`hint`值，所以`hint`值默认为`Number`（如果它是`Date`类型，则为`String`）。这也就意味着在`[0].valueOf`方法会先调用，而不是`toString`方法。由于`[0].valueOf`方法返回的仍然是`[0]`，它并不是原生数据类型，所以接着调用`[0].toString`方法，得到结果`'0'`，那么就变成了这样：

{% highlight JavaScript %}

1 + '0'

{% endhighlight %}

由于`0`是`String`，则接着会执行字符串拼接操作，即先将`+`两侧的操作数转换为`String`，然后再返回两个字符串拼接的值，即`10`，至此，我们已经知道了`++[[]][+[]]+[+[]] === '10'`的整个求值过程。

### FAQ

#### 为什么`++[[]][0]`返回`1`，而`++[]`会抛`ReferenceError`

首先，执行`++[]`时抛出的`ReferenceError`是在[11.4.4 Prefix Increment Operator](http://es5.github.io/#x11.4.4)中的第五步中执行`PutValue`时抛出的。

执行`PutValue(expr, newValue)`时第一步就会去检测参数`expr`是否是`Reference`类型，如果不是，则会抛出`ReferenceError`。

知道了这点就可以很好解释了，因为从[`++[[]][0]`执行步骤](#expression-left)我们可以知道，`[[]][0]`其实是一种`MemberExpression [ Expression ] `，它返回的是一个`Reference`类型，而[`[]`](http://es5.github.io/#x11.1.4)不是，则抛异常。

### 参考
{: #references }

1. [What's a valid left-hand-side expression in JavaScript grammar?](http://stackoverflow.com/questions/3709866/whats-a-valid-left-hand-side-expression-in-javascript-grammar)
