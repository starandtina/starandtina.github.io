---
layout: post
title: "Problems iterating over arrays and objects in JavaScript"
description: "Problems iterating over arrays and objects in JavaScript"
category: ['Front-End']
tags: ['Front-End', 'JavaScript', 'Iterator', 'js-dev']
---
{% include JB/setup %}

本文介绍在**JavaScript**中遍历数组或对象时常用的一些方法，以及使用**for**循环时的*Tricks*和*Best Practices*.
{: .countheads }

* ToC
{:toc}

### 遍历

基本上，数组和对象的遍历会有如下三种方案：

* **for**循环：包括**for**, **for...in**, **for...of**以及**for each...in**
* **ES5**中提供的数组遍历方法，如[Array.prototype.forEach](#[ES5: Array.prototype.forEach])
* 通过`Object.keys`或`Object.getOwnPropertyNames`遍历对象属性：一般与前面二种配合使用


本文只介绍第一部分，即**for**循环。

### for

#### [语法](#ES5.1: 12.6.3 The for Statement)

{% highlight JavaScript %}

for ( ExpressionNoInopt ; Expressionopt ; Expressionopt) Statement
for ( var VariableDeclarationListNoIn ; Expressionopt ; Expressionopt ) Statement

{% endhighlight %}

#### 用法

{% highlight JavaScript %}

var sum = 0;
for (var i = 0; i < 10; i++) {
  sum += i;
}

{% endhighlight %}

这是**JavaScript**中最传统也是最常使用的遍历数组的方法，这里有一点需要注意，就是使用`var`定义的变量的作用域始终是所包含的*scope*，除非使用*ES2015+*提供的*let*。

> ES2015新增了*let*命令，用来声明变量。它的用法类似于*var*，但是所声明的变量，只在*let*命令所在的代码块内有效。

下面是使用`let`后的代码，大家可以看到`i`只在*for block*内有效，在

{% highlight JavaScript %}

var sum = 0;
for (let i = 0; i < 10; i++) {
  sum += i;
}

console.log(i); // ReferenceError: i is not defined

{% endhighlight %}

如果浏览器不支持*let*，可以使用[Babel](https://babeljs.io/repl/#?experimental=false&evaluate=true&loose=true&spec=false&playground=true&code=%0Avar%20sum%20%3D%200%3B%0Afor%20(let%20i%20%3D%200%3B%20i%20%3C%2010%3B%20i%2B%2B)%20%7B%0A%20%20sum%20%2B%3D%20i%3B%0A%7D%0A%0Aconsole.log(i)%3B%20%2F%2FReferenceError%3A%20i%20is%20not%20defined)将你的*ES2015+*代码编译为与*ES5*兼容的代码，如上面这段代码，使用*Babel*后就会被编译为：

{% highlight JavaScript %}

'use strict';

var sum = 0;
for (var _i = 0; _i < 10; _i++) {
  sum += _i;
}

console.log(i); //ReferenceError: i is not defined

{% endhighlight %}

从上面代码，可以看出，在*for*循环中，变量`i`编译后变为`_i`了，那自然在代码块外面引用`i`时会报错了。

### for...in

#### [语法](#ES5.1: 12.6.4 The for-in Statement)

{% highlight JavaScript %}

for ( LeftHandSideExpression in Expression ) Statement 
for ( var VariableDeclarationNoIn in Expression ) Statement

{% endhighlight %}

#### 用法

{% highlight JavaScript %}

var steps = {
  one: 'one',
  two: 'two',
  three: 'three'
};

var countdown = "";

for (var step in steps) {
  countdown += steps[step] + "\n";
}

console.log(countdown);

{% endhighlight %}

* 只遍历可枚举的对象属性（即`enumerable`为`true`），同时还包括继承属性
* 可以用于遍历数组，但是**并不提倡这样做**。因为它不仅会遍历数组索引，同时也会遍历数组的属性值（如通过`arr.foo`添加的`foo`属性）以及原型链上可枚举属性
* 与*for*一样，使用`var`定义的变量的作用域始终是所包含的*scope*
* 在遍历过程中，同时可以删除属性值，但不建议这样做，因为这会改变原始的数组

#### 陷阱1

> 遍历数组索引，但是同时也会遍历数组的属性值以及原型链上可枚举属性

{% highlight JavaScript %}

Array.prototype.foo = 'foo';

var steps = ['one', 'two', 'three'];
var countdown = "";

steps.bar = 'bar';

for (var step in steps) {
  countdown += steps[step] + "\n";
}

console.log(countdown);
// one
// two
// three
// bar
// foo

{% endhighlight %}

#### 陷阱2

> 遍历对象时，对象属性值的遍历顺序是不保证的。

[ES3标准](#ES3: 12.6.4 The for-in Statement)规定**for...in**语句的属性遍历的顺序是**由对象定义时属性的书写顺序决定的**。

>The mechanics of enumerating the properties (step 5 in the first algorithm, step 6 in the second) is implementation dependent. The order of enumeration is defined by the object. ...

而在[ES5标准](#ES5.1: 12.6.4 The for-in Statement)中对 **for...in**语句的遍历机制又做了调整，**属性遍历的顺序是没有被规定的**，也就是说并没有规定对**JavaScript**的`Object`类型中的属性的存储顺序。

> The mechanics and order of enumerating the properties (step 6.a in the first algorithm, step 7.a in the second) is not specified. Properties of the object being enumerated may be deleted during enumeration....

对于对象遍历来说，这个并不是什么大问题，因为它本来就是无序的。但是你并不太可能想遍历数组时也是无序的，因为这会让你得到意想不到的结果。

<iframe width="100%" height="300" src="//jsfiddle.net/starandtina/6yrfqrfa/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

#### 跳过继承属性

遍历对象时，大多数时候我们并不想遍历继承属性，这时可以在遍历时通过`hasOwnProperty()`方法跳过继承属性。

{% highlight JavaScript %}

for (var step in steps) {
  if (steps.hasOwnProperty(step)) {
    console.log(step);
  }
}

{% endhighlight %}

### for...of

[ES2015](#ES2015: for...of)中引入了Iterator的概念，它为你迭代集合中的每一个元素提供了更大的便利。Iterator属于一种接口规格，任何数据结构只要部署这个接口，就可以完成遍历操作，即依次处理该结构的所有成员。它的作用有两个，

* 一是为各种数据结构，提供一个统一的、简便的接口
* 二是使得数据结构的成员能够按某种次序排列

在ES2015中，遍历操作特指**for...of**循环，即Iterator接口主要供**for...of**消费。**for...of**与其它的遍历一样，除了它是专门用于与*Iterables*对象（部署了Iterator的对象，`Array`或是`String`都是Iterables）一起工作的。

{% highlight JavaScript %}

let values = [1, 2, 3];

for (let i of values) {
    console.log(i);
}

{% endhighlight %}

关于Iterator和Iterable部分，可以参考[ECMAScript 6 入门](#[5])。

ES2015中，一个数据结构只要部署了`Symbol.iterator`方法，就被视为具有*iterator*接口，就可以用**for...of**循环遍历它的成员。也就是说，**for...of**循环内部调用的是数据结构的`Symbol.iterator`方法。

**for...of**循环可以使用的范围包括数组、`Set`和`Map`结构、某些类似数组的对象（比如`arguments`对象、`DOM NodeList`对象）、`Generator`对象，以及字符串。

{% highlight JavaScript %}

const arr = ['red', 'green', 'blue'];
let iterator  = arr[Symbol.iterator]();

for(let v of arr) {
    console.log(v); // red green blue
}

for(let v of iterator) {
    console.log(v); // red green blue
}

{% endhighlight %}

**JavaScript**原有的**for...in**循环，只能获得对象的属性名，不能直接获取属性值。而**for...of**循环，允许遍历获得键值。

{% highlight JavaScript %}

var arr = ["a", "b", "c", "d"];

for (a in arr) {
  console.log(a); // 0 1 2 3
}

for (a of arr) {
  console.log(a); // a b c d
}

{% endhighlight %}

上面代码表明，**for...in**循环读取属性名，**for...of**循环读取属性值。如果要通过**for...of**循环，获取数组的索引，可以借助数组实例的[`entries()`方法和`keys()`方法](#6)。

### for each...in

只有[Firefox提供](#for each...in)，用于遍历对象属性值，**不建议使用**。

#### 用法

{% highlight JavaScript %}

var sum = 0;
var obj = {prop1: 5, prop2: 13, prop3: 8};

for each (var item in obj) {
  sum += item;
}

console.log(sum); // logs "26", which is 5+13+8

{% endhighlight %}

### 总结

#### 数组遍历

由于数组遍历有上述的这些问题，也许你永远也不会用**for...in**去遍历数组了。那么我们有如下的替代选择：

* 简单的**for**遍历
* **ES5**中提供的数组遍历方法
* 永远不要使用**for...in**或是**for each...in**

#### 对象遍历

* 组合使用**for...in**和`hasOwnProperty()`
* 组合使用`Object.keys()`或是`Object.getOwnPropertyNames()`和`Array.prototype.forEach`遍历方法

{% highlight JavaScript %}

var obj = {
  first: "John",
  last: "Doe"
};
// Visit non-inherited enumerable keys
Object.keys(obj).forEach(function (key) {
  console.log(key);
});

{% endhighlight %}

### 参考

* {: ID='[ES5: Array.prototype.forEach]'}[ES5: Array.prototype.forEach](http://kangax.github.io/compat-table/es5/#Array.prototype.forEach)
* {: id='ES5.1: 12.6.3 The for Statement'}[ES5.1: 12.6.3 The for Statement](http://es5.github.io/#x12.6.3)
* {: id='ES3: 12.6.4 The for-in Statement'}[ES3: 12.6.4 The for-in Statement](http://bclary.com/2004/11/07/#a-12.6.4)
* {: id='ES5.1: 12.6.4 The for-in Statement'}[ES5.1: 12.6.4 The for-in Statement](http://es5.github.io/#x12.6.4)
* {: id='for each...in'}[for each...in](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for_each...in?redirectlocale=en-US&redirectslug=JavaScript%2FReference%2FStatements%2Ffor_each...in)
* {: id='ES2015: for...of'}[ES2015: for...of](http://kangax.github.io/compat-table/es6/#for..of_loops)
* {: id='[5]'}[ECMAScript 6 入门](http://es6.ruanyifeng.com/)
* {: id='[6]'}[Array.prototype methods](http://kangax.github.io/compat-table/es6/#Array.prototype_methods)
