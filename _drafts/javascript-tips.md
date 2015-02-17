---
layout: post
title: "JavaScript Tips"
description: ""
category: 
tags: []
---
{% include JB/setup %}

本文是关于**JavaScript**的一些常用Tips。
{: .countheads }

* ToC
{:toc}

----

### 结构

#### 空白

{% highlight JavaScript %}

var that = this;

{% endhighlight %}

上面的代码中，等号两端的空白字符是可以去掉的，但我们需要保留它们，不至于让**that=this**看起来像一个token。

在特定的位置加上空格有助于代码的可读性，以下位置*必须*加上空格：

* 除括号外，所有运算符的前后，如`+`, `-`等等
* 用作代码块起始的左大括号`{`前，包括`if`、`else`、`try`、`finally`这些关键字之后，以及**函数定义的参数列表**之后
* 以下关键字之后：`for`、`switch`、`while`
* 对象初始化（`{ ... }`）的每个属性名的冒号`:`后
* 所有逗号`,`后
* 单行的对象初始化（`{ ... }`）左大括号`{`后和右大括号`}`前

更具体点来说，基本代码空白规则如下：

* 左括号的左边需要有空白
* 右括号的右边需要有空白
* 双目或三目运算符的两边需要有空白
* 逗号或分号的右边需要有空白
* 如果有多个空白符时合并成一个
* **function**声明中，在函数形参左括号的左边不需要空白，保持和调用的形式一致
* ...

###### 代码空白示例

{% highlight JavaScript %}

if (num > 0) {
}

for (i = 0; i < len; i++) {
}

function setStyle(dom, name, value) {
}

{% endhighlight %}          

#### 缩进

使用*2*个空格字符作为代码缩进，避免使用*tab*空白字符，因为*tab*空白字符在不同编辑器下可能会有不同的表现，使用空格字符作为缩进能很好的避免这种情况。主流编辑器都能够设置自动对tab进行替换，以方便代码编写。

#### 行字符限制

每行的字符数建议不超过*80*个字符。
过长的程序行应在适当的地方断行，并保持断行后的代码整齐，同时断行不能破坏表达式本身的意义。

##### 在逗号后面断行

{% highlight JavaScript %}

// 断行示例：逗号后面断行
callFunction(
    expression1,
    expression2,
    expression3,
    expression4
);

{% endhighlight %}      

##### 在逻辑运算符前面断行

{% highlight JavaScript %}

// 断行示例：逻辑运算符前面断行
if (condition1
    && condition2
    && condition3
) {
    callFunction();       
}

{% endhighlight %} 

##### 在+号前面断行
  
{% highlight JavaScript %}

// 断行示例：+号前面断行
var str = 'hello world'
          + 'hello world2'
          + 'hello world3';
                          
{% endhighlight %}

*不得*出现以下情况：

{% highlight JavaScript %}

// 最后一个结果很长，但不建议合并条件和第一分支
// 不要这么干
var result = condition ? resultA :
    thisIsAVeryVeryLongResult;

{% endhighlight %}

这种方法会导致语义上的分裂，即*条件和分支*在一行，*另一分支*在一行，没有按逻辑进行组织。

#### 不要省略if、while、for、do的块

对于`if`、`while`、`for`、`do`，应用`{}`将执行体包围，避免产生理解障碍。
主流的编辑器，以回车新起一行时，输入光标通常保留在上一行的缩进位置，所以编写时容易造成这样的情况：

###### 容易造成理解歧义的块省略

{% highlight JavaScript %}

if (condition)
    callFunction1();
    callFunction2();

{% endhighlight %} 


### 命名

#### 常用命名规则

* Camel命名法，如`thisIsAnApple`
* Pascal命名法，如`ThisIsAnApple`
* 下划线命名法，如`this_is_an_apple`
* 中划线命名法，如`this-is-an-apple`

根据不同类型的内容，采用不同的命名法：

- 变量名，使用Camel命名法
- 参数名，使用Camel命名法
- 函数名，使用Camel命名法
- 方法/属性，使用Camel命名法
- 私有（保护）属性、函数以下划线`_`开头
- 常量名，使用全部大写的下划线命名法，如`IS_DEBUG_ENABLED`
- 类名，使用Pascal命名法
- 枚举名，使用Pascal命名法
- 枚举的属性，使用全部大写的下划线命名法
- 命名空间，使用Camel命名法

命名同时还需要关注语义，如：

- 变量名*应当*使用名词
- **Boolean**类型的*应当*使用**is**、**has**等起头，表示其类型
- 函数名*应当*使用动宾短语
- 类名用*应当*使用名词

#### 常量命名

目前主流浏览器中，**JavaScript**中无法使用`const`（[ES6已经支持const](#[1])）声明常量。如果要声明一个在程序运行阶段不会更改的变量（如配置项），命名规则为：每个字母大写，单词间以下划线分割。

###### 常量命名示例

{% highlight JavaScript %}

var IS_DEBUG_ENABLED = true; 

{% endhighlight %}

#### 变量命名

局部变量与全局变量的命名均使用**Camel命名法**。

###### 变量命名示例

{% highlight JavaScript %}

var foo = 'foo';

{% endhighlight %} 

#### 函数命名

**JavaScript**中，函数可以作为构造器用来实例化对象，也可以被调用。为了增加代码可读性，需要在命名上进行区分。

###### 对于被调用的函数，使用**Camel命名法**
  
{% highlight JavaScript %} 

// 函数命名示例：调用函数  
function foo() {
  // ...
}

{% endhighlight %}                            

###### 对于通过`new`来实例化对象的函数构造器，使用**Pascal命名法**
  
{% highlight JavaScript %}

// 函数命名示例：函数构造器
function Dog(name) {
  this.name = name;
}                           

{% endhighlight %}

**JavaScript**已经提供了许多内置的函数构造器，如`Object`, `RegExp`等等。

#### 对象命名

对象的命名规则与对象本身的职责有关。

* 如果对象是用于数据保存的，则使用**Camel命名法**
* 如果对象是作为静态类的，则使用**Pascal命名法**


### 注释

你必须为你的代码写注释，尽管除了你其他人不太可能去看你的代码。对于工程师来说，注释是一种编程意识，同时也是一种责任。
当你写代码时，可能你会觉得这段代码的意思是很明显的，但是当你过一段时间回过头来看这段代码时，你得花可能比写这段代码更多的时间去回想起它是做什么的。

当然也这也不是让你去为每个变量，每一个表达式都加上注释，但是，至少你也得为每个函数添加注释，让看代码的人知道这个函数是做什么的，它需要哪些类型的函数，而它的返回值类型又是什么，它里面又用到了哪些很Trick的技巧或是技术。

>对于注释，最重要的也是最难坚持的习惯是保持注释最新，因为过时的注释会误导你，有时比没有注释还更糟糕。

#### 文件描述

在**JavaScript**文件顶部需要有文件描述的注释，其中包括项目名、作者、修改日期等信息。

###### 注释示例：文件描述

{% highlight JavaScript %}

/*!
 * jQuery JavaScript Library v1.9.0
 * http://jquery.com/
 *
 * Includes Sizzle.js
 * http://sizzlejs.com/
 *
 * Copyright 2005, 2012 jQuery Foundation, Inc. and other contributors
 * Released under the MIT license
 * http://jquery.org/license
 *
 * Date: 2013-1-14
 */

{% endhighlight %}

但是有一点需要注意，文件描述通常不用于生成API文档的。

#### 函数与方法描述

函数和方法必须用注释描述其功能。

###### 注释示例：函数与方法描述
  
{% highlight JavaScript %}

/**
 * 发送一个ajax post请求
 *
 * @param {string} url 发送请求的url
 * @param {Object} jsonData 需要发送的数据
 * @param {Function} successFn 请求成功时触发，function(data, status)
 * @param {Function} failedFn 请求失败时触发，function(msg)
 */
function jsonPost(url, jsonData, successFn, failedFn) {
  $.ajax({
    type: 'POST',
    url: url,
    data: jsonData,
    dataType: 'json',
    success: function (data, textStatus, jqXHR) {
      var ret = data;

      //状态为0或者大于2，表示成功
      if (ret.status === 0 || ret.status > 2) {
          //调用成功函数，传递数据，同时传递状态码
          successFn(ret.data, ret.status, ret.msg);
      }
      //状态为1，表示失败
      else if (ret.status === 1) {
          if (failedFn) {
              failedFn(ret.msg);
          }
      }
      //状态为2，表示重定向至某个地址
      else if (ret.status === 2) {
          window.location = ret.data;
      }
    }
  }).fail(function(jqXHR, textStatus, errorThrown) {
    if (failedFn) {
      failedFn(errorThrown.message);
    }
  });
}

{% endhighlight %}

我们描述一个函数的时候，只描述做什么(What)，不描述具体如何做(How)。

#### 注释标签

常用的注释标签包括`param`、`return`、`private`、`extend`等。下表是**[JSDoc](#[2]})**支持的部分标签.

#### JSDoc常用标签

| 标签 | 说明 |
| :-------------: | :------------- :|
| @augments  |  指明类的基类，同@extends。建议使用@extends |
| @author  |  作者名 |
| @class  |  用来给一个类提供描述（不描述构造函数） |
| @constant  |  指明常量 |
| @constructor  |  描述一个类的构造函数 |
| @constructs  |  指定一个函数作为构造函数 |
| @default  |  描述变量的默认值 |
| @deprecated  |  表示函数/类已不再提供 |
| @description  |  说明的描述，与首行不带tag的描述同义 |
| @event  |  描述事件 |
| @example  |  提供使用示例，比如常用的调用方式 |
| @extends  |  指明类的基类 |
| @field  |  指明变量引用 |
| @function  |  指明变量引用了一个函数（function类型） |
| @ignore  |  让jsdoc忽视改描述，不生成文档 |
| @link  |  类似于@link标签，用于连接许多其它页面 |
| @memberOf  |  指明成员所属类 |
| @name  |  强迫生成文档时不适用默认名，指定一个名称 |
| @namespace  |  描述一个对象为namespace |
| @param  |  描述一个函数的参数 |
| @private  |  表示成员或类为私有，默认不生成文档 |
| @property  |  在构造器上指明类的属性 |
| @public  |  指明一个内部变量或成员为公有 |
| @requires  |  描述依赖的资源 |
| @return     |  描述一个函数的返回值 |
| @returns  |  描述一个函数的返回值  |
| @see  |  描述相关资源 |
| @since  |  指明一个版本，表示该特性从该版本开始被支持  |
| @throws  |  描述函数可能产生的错误  |
| @type  |  指定函数的返回类型  |
| @version  |  描述版本  |
{: .neat }

#### 操作描述

当算法或业务逻辑代码不容易被阅读时，我们可以用单行或多行注释进行内部描述。

###### 注释示例：操作描述

{% highlight JavaScript %}
  
if (condition1) { // 注释写在这里
  callFunction1(); // 注释写在这里
} else {
  callFunction2(); // 注释写在这里
}

/*
 * 注释写在这里
 */
for (var i = 0, len = list.length; i < len; i++) {}

{% endhighlight %}


### 条件

#### 分支排列顺序

按执行频率排列分支的顺序。这样做有两个好处：

* 阅读的人容易找到最常见的情况，增加可读性
* 提高执行效率

#### 条件判断的陷阱

在**JavaScript**中, `false`, `null`, `0`, `''`, `undefined`和`NaN`都被当做`false`值。

而在执行`==`时，则会有一些让人难以理解的陷阱。

###### 关于`==`的Tricks

{% highlight JavaScript %}
  
(function () {
  var undefined;

  undefined == null; // true

  1 == true; // true
  2 == true; // false
  0 == false; // true
  0 == ''; // true
  NaN == NaN; // false
  +0 == -0; // true
  [] == false; // true
  [] == ![]; // true
})();

{% endhighlight %}        

对于不同类型的`==`判断，有这样一些规则，顺序自上而下：

1. 如果`x`和`y`的类型相同，则使用[`===`算法](http://es5.github.io/#x11.9.6)比对
1. 如果`x`为`null`且`y`为`undefined`，则返回`true`，反之亦然
1. 如果其中一个是`Number`类型，另外一个是`String`类型，那么需要先把`String`类型的转换为`Number`，然后再执行`==`
1. 如果其中一个是`Boolean`类型，另外一个是非`Boolean`类型，那么需要先把`Boolean`类型转换为`Number`类型，然后再执行`==`
1. 如果其中一个是`String`或是`Number`类型，另外一个是`Object`类型，那么需要先把`Object`类型转换为**primitive**的类型，然后再执行`==`
1. 其它情况返回`false`

想了解更多，请参考**JavaScript tricks 2: Use === instead of ==**[[3]](#[3])。

#### 使用`case`代替`if`

在同时满足下面的条件时，建议使用`case`代替`if`：

* 分支条件相对单一
* 2个分支以上

下面是一个简单的小例子。

###### 使用`case`代替`if`例子

{% highlight JavaScript %}

// 替换前，使用if：
if (type === 0) {
  call0();
} else if (type == 1) {
  call1();
} else {
  callOther();
}

// 替换后，使用case：
switch (type) {
case 0:
  call0();
  break;
case 1:
  call1();
  break;
default:
  callOther();
  break;
}
        
{% endhighlight %}

#### `switch`模式

在写`switch`语句时，你可以遵循该模式从而改进代码的可读性和鲁棒性。如下面的代码：

{% highlight JavaScript %}

switch (type) {
case 0:
  call0();
  break;
case 1:
  call1();
  break;
default:
  callOther();
  break;
}

{% endhighlight %}

* `case`与`switch`对齐
* 缩进`case`内的代码
* 所有的`case`语句以`break`结尾，避免*fall-through*，如果你确实不需要某个`break`，那么必须添加注释说明，否则别人会认为这是一个错误
* 以`default`结尾，从而保证始终会有一个值返回，即使所有的条件都不满足


### 数据类型

在**JavaScript**中，分为*primitive*和*non-primitive*值。一共有5种primitive类型，分别表示不同的primitive值。

* Null: null
* Undefined: undefined
* Number: 所有的数字，如0, 10, 3.1415，以及`NaN`和`Infinity`
* Boolean: 只有两个布尔值，true 和 false
* String: 所有字符串，如 'foo'和''

所有其它的值都是non-primitive的，也就是`Object`类型，包括：**Object**、**Function**、**Array**、**String**、**Boolean**、**Number**、**Math**、**Date**、**RegExp**、**Error**。

#### `typeof`

我们可以通过`typeof`操作符检测某个值的类型：

{% highlight JavaScript %}

typeof null; // 'object'
typeof undefined; // 'undefined'
typeof 0; // 'number' (`typeof NaN` is also 'number')
typeof true; // 'boolean'
typeof 'foo'; // 'string'
typeof {}; // 'object'
typeof function () {}; // 'function'
typeof []; // 'object'

{% endhighlight %}

 `typeof null`是不应该返回`'object'`的，因为它并不是一个对象。 这是自从**JavaScript**第一版中就已经存在的一个错误，但是已经为时已晚，现在已经不能去修复它了,因为它会造成*breaking change*。更理想情况下是返回`'null'`。

#### `instanceof`

我们可以用instanceof来检测对象的构造器。

{% highlight JavaScript %}

value instanceof Constructor

{% endhighlight %}

如果它返回`true`，那么它就是由这个`Constructor`所创建的一个对象实例。

{% highlight JavaScript %}

function Foo() {}
var f = new Foo();

f instanceof Foo; // true
f instanceof Object; // true
[] instanceof Array; // true

undefined instanceof Object; // false
null instanceof Object; // false

{% endhighlight %}

对于`value instanceof Constructor`这样的表达式，如果`Constructor.prototype`在`value`的原型(*__proto__*)链里，那么就返回`true`。

但是，由于instanceof遍历原型链的特性，可能产生如下问题：

###### instanceof的风险

{% highlight JavaScript %} 

function A() {
  this.testA = new Function();
}

function B() {
  this.testB = new Function();
}

var a = new A();
A.prototype = new B();

a instanceof A; // false

{% endhighlight %}

上面的例子中，`instanceof`认为`a`不是`A`的实例，因为`A`的*prototype*已经被更改。

>实例化对象前，应完成类的prototype成员构建，否则可能导致不期望的判断结果。

#### 类型检测的例子与思考

我们需要判断一个变量的类型的时候，需要考虑很多的可能性。

一个例子：判断变量`a`是否字符串。

最简单的：

{% highlight JavaScript %}

typeof a === 'string'

{% endhighlight %}            

可能a是字符串的包装类型，于是：

{% highlight JavaScript %}

a.constructor === String

{% endhighlight %}            

可是`a`为`null`或`undefined`时会报*runtime error*，于是：

{% highlight JavaScript %}

typeof a === 'string' || a instanceof String

{% endhighlight %}             

如果我们把`String`构造函数覆盖了，如`function String(){}`，会判断出错，于是：

{% highlight JavaScript %}

Object.prototype.toString.call(a) =＝= '[object String]'

{% endhighlight %}            

如果`Object.prototype.toString`再被覆盖了......这个时候，我们就没办法了。

* 对于布尔、数值与字符串来说，我们应针对简单类型进行编程。
* 不要重写原生对象构造器，不要盲目地去扩展原生对象的*prototype*。

#### 简单类型与包装类型

**JavaScript**被许多其它编程语言影响过，其中之一就是**Java**，是**Java**让**JavaScript**也有了简单类型和包装类型。

* 借鉴**C**语言的基本语法
* 借鉴**Java**语言的数据类型和内存管理，直接导致**JavaScript**中数据类型分为*primitive*和*non-primitive*
* 借鉴**Scheme**语言，将函数提升到"第一等公民"（first class）的地位　
* 借鉴**Self**语言，使用基于原型（prototype）的继承机制
* 借鉴**AWK**，使用`function`关键词
* 借鉴**Perl**和**Python**，使用对字符串，数组以及正则表达式的处理
* 借鉴HyperTalk，让HTML支持`onclick`内联事件模型

那么，问题来了：

>如果一个原生String类型没有属性，那么为什么`'foo'.length可以正确执行返回字符串的长度呢？

那是因为**JavaScript**会自动地进行隐式装箱操作，也就是把原生类型转换为与它对应的包装类型。
在上面这个例子中，字符串值`foo`会转换为**String**对象，从而让你可以访问`length`属性。

{% highlight JavaScript %}

String.prototype.returnMe = function () {
  return this;
}
var a = 'foo';
var b = a.returnMe();

a; //'foo' 
typeof a; //'string' (still a primitive)
b; //'foo'
typeof b; //'object'

{% endhighlight %}

###### 简单类型与包装类型的差异

{% highlight JavaScript %}

var str = '1',
var str2 = new String('1');

typeof str;  // string
typeof str2; // object
                
{% endhighlight %}

>对于布尔、数值与字符串来说，应针对简单类型进行编程，包装类型的方法会自动通过隐式装箱调用。

###### 使用简单类型

{% highlight JavaScript %}

var str = 'Hello world!';
str.length;
                  
{% endhighlight %}

#### 简单类型转换

##### Converting to String

将其它值转换为字符串，最常见的方法就是使用*+*运算符，因为*+*运算符中如果其中一个操作数是字符串，那么它就会执行字符串拼接。
所以，想要得到类型转换后的字符串值，最简单的方法就是将该值与一个空字符串拼接。如：

{% highlight JavaScript %}

foo + ''

{% endhighlight %}

另外一个可选的方法就是[将*String*构造函数作为函数调用](http://es5.github.io/#x15.5.1)，即将待转换值作为参数传入它。

{% highlight JavaScript %}

var string = String(foo);

{% endhighlight %}

* 使用 `'' + colValue` 做类型转换：数值

| # | -3.14 | -0 | +0 | 1 | 3.14 | 314e-2 | -Infinity | +Infinity | NaN |
|---
| '' + colValue   | '-3.14'   | '0' | '0' | '1' | '3.14' | '3.14' | '-Infinity' | '+Infinity' | 'NaN' |
{: .table }

* 使用 `'' + colValue` 做类型转换：其它值

| # | undefined | null | true | false | {} | [] | function foo() {} |
|---
| '' + colValue | 'undefined' | 'null' | 'true' | 'false' | '[object Object]' | '' | 'function foo(){}' |
{: .table }


对象或是函数需要转换为字符串时，会默认调用它们的*toString*方法。注意，`Object.prototype.toString`和`Function.prototype.toString`方法我们是可以任意覆盖的。

将函数转换为字符串的过程我们称之为*函数反编译*，一般我们可以通过`fn.toString()`或`String(fn)`或`fn + ''`实现，但是最后都是通过`Function.prototype.toString`[[4]](#[4])实现的。但是它并不一定会反射函数的源代码，因为它的实现与平台相关，结果并不总是一样的[[5]](#[5])，特别是ES6中新出现的*arrow function*。

##### Converting to Number 

字符串到数值的转换，大家首先会想到的最好的方式一般是*parseInt*和*parseFloat*，但是他们真的是最好的方式吗？

{% highlight JavaScript %}

// 将字符串转移为数值（包括整数和浮点数）的不同方式
parseFloat('100')
+'100'
// 只用于整数
parseInt('100', 10)
'100'|0
'100' >> 0
'100' << 0
//Only works for positive numbers
'100' >>> 0

// ToInteger实现：http://es5.github.io/#x9.4
function ToInteger(x) {
  x = Number(x);
  return x < 0 ? Math.ceil(x) : Math.floor(x);
}

{% endhighlight %}

* [JSPerf: parseInt test](http://jsperf.com/type-cast/3)
* [JSPerf: parseFloat test](http://jsperf.com/type-cast/2)

我们得到的结论：

* 在*Chrome*和*Firefox*中，*parseFloat*比*+*运算符快
* 在*Chrome*上，*parseInt*是最快的，位运算符最慢（慢72%）
* 在*Firefox*中，位运算符最快，而*+*最慢（比*parseInt*快99%，比*+*快100%）
* 在大多数情况下，`parseInt`是你最正确的选择

##### `parseInt`并不是常常能正确的工作: #todo

{% highlight JavaScript %}

[1, 2, 3, 4].map(parseInt); // [1, NaN, NaN, NaN]
parseInt('08'); // 0
parseInt('08', 10); //8
parseInt(1000000000000000000000.5, 10); // 1

{% endhighlight %}

##### Coverting Float to Integer

我们可以使用`Math.floor`，`Math.round`，`Math.ceil`，又或是`ToInteger`方法。


### 对象（Object）

#### 初始化

使用对象直接量`{}`初始化对象，不使用`new Object`。

###### 对象初始化

{% highlight JavaScript %}

var obj = {
  num: 1,
  str: 'hello'
};

{% endhighlight %}

>对象的最后一个成员后不要添加*,*号，否则在IE下会报语法错误。

类似的，初始化`Array`或是`RegExp`时，也应使用直接量。

###### `Array`与`RegExp`初始化

{% highlight JavaScript %}

var arr = [1, 2, 3, 4, 5, 6];
var regex = /^[a-z]+$/i;
                  
{% endhighlight %}

我们都知道在**JavaScript**中提供了二种方式定义正则表达式，一个是通过`RegExp`构造函数，另一个通过直接量。
`RegExp`构造函数产生的对象是在*runtime*时编译的，但是直接量却是在脚本加载时就已经编译了，所以它的性能相对会更好[[6]](#[6])。从另一方面来看，字面量更适合已经知道表达式构成时，而通过构造函数更适合动态构建正则表达式，如通过用户的输入。

#### 成员访问

对象成员访问有两种方式：

* obj.identifier
* obj[expression]

如果成员名称为**JavaScript**保留字，则可以通过`obj[expression]`方式访问。其他时候，通过`obj.identifier`访问,该方式增加可读性，并且减少字符数。

#### 删除成员

`delete`[[7]](#[7])关键字可以删除对象的成员。

###### 使用delete删除对象成员

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


#### 原生对象污染

>禁止扩展原生对象，如`Object`、`String`、`Function`等。

污染原生对象可能会带来冲突，比如两个开发者或者使用的两个库都扩展了`String`的`format`函数，则代码字面上后面的会覆盖前面的，导致不可预料的错误。
污染原生对象可能导致不期望的后果，如扩展了`Object`，则对任何一个对象的`for...in`都会遍历出这个扩展。

{% highlight JavaScript %}

function Person(name) {
  this.name = name;
}
Person.prototype = {
  describe: function () {
    return 'Name: ' + this.name;
  }
};
var person = new Person('Jane');
for (var key in person) {
  console.log(key); // name, describe
}

Object.prototype.foo = 1;
for (var i in obj) {
  console.log(i); // foo
}

Object.getOwnPropertyDescriptor(Object.prototype, 'foo'); // Object {value: 1, writable: true, enumerable: true, configurable: true, foo: 1}

{% endhighlight %}

我们通过`Object.getOwnPropertyDescriptor`可以看到，`foo`的`enumerable`属性是为`true`的，所以在`for in`遍历时我们会看到它。


### 函数（Function）

#### 函数也是对象

**JavaScript**是支持*First-class functions*的，也就是说函数是*一等公民*。

>所谓*第一等公民*（first class），指的是函数与其他数据类型一样，处于平等地位，可以赋值给其他变量，也可以作为参数，传入另一个函数，或者作为别的函数的返回值。

{% highlight JavaScript %}

var print = function (i) {
  console.log(i);
};

[1, 2, 3].forEach(print);
print instanceof Object; //true

{% endhighlight %}

#### 调用方式

函数可以以以下4种方式调用：

##### 直接调用
  
{% highlight JavaScript %}

foo('Hello world!');

{% endhighlight %}                         

##### 方法调用

{% highlight JavaScript %}
    
obj.foo('Hello world!');

{% endhighlight %}              

##### 构造器调用
  
{% highlight JavaScript %} 

new Foo('hello');

{% endhighlight %}                
  
##### apply或call
  
{% highlight JavaScript %}

Object.prototype.toString.call(arr); // '[object Array]'
                        
{% endhighlight %}

#### this指针

函数在被调用时，this指针是由调用者所决定的。

* 直接调用时，`this`指针为`Globel`，浏览器端为`window`，但是如果是在*strict mode*下访问`this`会抛出错误
* 方法调用时，`this`指针为方法所属对象。如`obj.foo()`，`this`指针为`obj`。
* 构造器调用时，解析器会创建一个`Object`，`this`指针为这个`Object`，默认这个`Object`会被返回
* 使用`apply`或`call`时，可以通过第一个参数指定调用时的`this`指针

#### 作用域（scope）

与熟知的绝大多数高级语言（c、c++、java...）不同，**JavaScript**变量的作用域是*function*域，不是*block*域。

###### 变量的函数作用域

{% highlight JavaScript %}

function myFunc() {
  for (var i = 0; i < 10; i++) {
    var num = i; // 生命周期是function的生命周期
  }

  num; // 9
}
myFunc();

for (var k in {a: 1, b: 2 }) {
  alert(k);
}
alert(k); // 尽管循环已经结束，但是变量`k`仍然在作用域中

{% endhighlight %}

函数在被调用的时候，会创建一个作用域。作用域可以看做是一个**JavaScript**对象，其中包括`arguments`属性，以及变量，函数声明和与形参同名的属性。

#### 闭包

函数对象会引用父函数作用域对象。所以，我们能够通过闭包，让随后执行的函数使用当前父函数运行时的环境（除了`this`与`arguments`）

###### 简单的闭包

{% highlight JavaScript %}

var myFunc = (function (num) {
  var me = this;

  return function () {
    num; // 2
    me == window; // true;
  };
})(2);
myFunc();

{% endhighlight %}

#### 作用域泛滥

父函数作用域对象被子函数引用后，如果子函数包含引用，则父函数的作用域不会被回收。下面是一个作用域泛滥的例子，这个例子额外创建了lis.length个scope，并且他们都不会被释放：

###### 作用域泛滥

{% highlight JavaScript %}

function init() {
  var lis = ul.getElementsByTagName('li');
  _.forEach(lis, function (item) {
    item.onclick = function (e) {
      e = e || window.event;
      // ......
    };
  });
}
init();

{% endhighlight %}

#### 闭包原则

* 能不使用闭包的时候，不使用闭包
* 保持闭包环境的简单可管理，通常闭包环境中不包含超过10个父域变量
* 不包含对非原生对象的引用，如`DOM`对象。如果包含循环引用，并引用了非原生对象，可能导致无法释放产生内存泄露


>如果闭包中需要使用dom，闭包环境应引用dom元素id，在需要使用的时候再获取。

###### 闭包函数中使用dom元素

{% highlight JavaScript %}

function getHandler() {
  var domId = 'dom';
  return function () {
    var dom = document.getElementById(domId); // 获取DOM元素
  };
}

{% endhighlight %}             

>在mouseover事件或其他频繁执行的状态，如果多次获取同一个dom对象会导致效率问题，可以在第一次持有dom对象的引用，在使用完后释放。


### 字符串

字符串处理是前端最常见的操作，所以它很有可能成为性能瓶颈。

#### 查找

字符串可以使用正则也可以使用`String.prototype.indexOf`、`String.prototype.search`，从这个[测试](http://jsperf.com/string-searching/2)可以看出，这里的性能问题有很多种情况，对于我们关注的IE6而言，*indexOf*字符串和*search*正则性能是相同的，不够*search*字符串要比正则快，有可能是构建正则的开销导致，除了IE6、IE7，其它JS引擎的正则都不会慢于字符串，甚至快很多。

在*v8*中的*indexOf*会根据待匹配的字符串特点来选择不同的算法，对于小于7个字符会使用*线性查找*，而大于等于7个字符则使用*Boyer-Moore-Horspool*。

#### 拼接

字符串拼接是一个相当消耗资源的操作，但是最好的做法是什么呢？过去常见的性能优化方法中都会提到使用数组*push+join*的方式来取代字符串的拼接，但是实际上是这样的么？
如果我们对拼接后的字符串不做任何操作，那么最新版的Chrome(40.0.2214.111 (64-bit))和Firefox(35.0.1)结果是一样的。

可以通过JSPerf上的一个[测试](http://jsperf.com/string-concat-fast/9)可以看出对于，结果是不一样的。

* Firefox上，*+＝*操作符比*+*, *String.prototype.concat*和*Array.prototype.join*都快. 并且*Array.prototype.join*是最慢的
* Chrome上，*Array.prototype.join*是最快的，而*String.prototype.concat*是最慢的

v8中的实现类似**Java**的*StringBuilder*，内部也是用数组来实现的，但做了特殊优化，所以性能比JS数组快。
Firefox的方法是对于短字符串使用*memcpy*，长字符串使用*Rope数据结构*。

总的来说，字符串拼接，应使用数组作为*StringBuffer*保存字符串片段，使用时调用*join*方法。避免使用*+*或*+=*的方式拼接较长的字符串，每个字符串都会使用一个小的内存片段，过多的内存片段会影响性能。

>绝大部分现代浏览器都对+=的字符串拼接进行了优化，但在*IE6*和*IE7*中字符串拼接远远慢于*push+join*，同时IE6的使用率依然很高。

###### 不提倡的拼接方式：+=

{% highlight JavaScript %}

var str = '';
for (var i = 0, len = list.length; i < len; i++) {
  str += '<div>' + list[i] + '</div>';
}
dom.innerHTML = str;

{% endhighlight %}


###### 提倡的拼接方式：push+join

{% highlight JavaScript %}

var str = [];
for (var i = 0, len = list.length; i < len; i++) {
  str.push('<div>' + list[i] + '</div>');
}
dom.innerHTML = str.join('');

{% endhighlight %}

###### 更好的拼接方式：push+join, 使用临时变量存储数组长度

{% highlight JavaScript %}

var str = [];
var strLen = 0;

for (var i = 0, len = list.length; i < len; i++) {
  str[strLen++] = '<div>' + list[i] + '</div>';
}
dom.innerHTML = str.join('');

{% endhighlight %}


#### 字符串格式化

复杂的字符串拼接，应使用格式化的方式，提高代码可读性。请参考baidu.string.format。


###### 字符串格式化

{% highlight JavaScript %}
  // 让人难以理解的拼接：    
  var str = '<div id="' + id + '" class="' + className + '">' + html + '</div>';

  // 使用格式化：
  var tpl = '<div id="${0}" class="${1}">${2}</div>';
  var str = baidu.format( tpl,
                          id, className, html
    );
 {% endhighlight %}                 




#### 字符串常用实例方法


###### charAt(position)

charAt方法返回字符串处于position位置的字符。如果position为负值或超出字符串长度，则返回空字符串。


####### string.charAt

{% highlight JavaScript %}
  'hello'.charAt(0); // e
 {% endhighlight %}                     




###### charCodeAt(position)

charCodeAt方法返回字符串处于position位置字符的unicode码。如果position为负值或超出字符串长度，则返回NaN。


####### string.charCodeAt

{% highlight JavaScript %}
  'hello'.charCodeAt(0); // 101，e的unicode码，也就是ascii码。
                      
{% endhighlight %}



###### indexOf(searchString, position)

indexOf方法在字符串内查找子字符串searchString的第一个匹配的位置。无匹配时返回-1。可选参数position可设置从指定位置开始查找。


####### string.indexOf

{% highlight JavaScript %}
  var index = 'Hello World'.indexOf('l'); // 2
{% endhighlight %}                      




###### lastIndexOf(searchString, position)

lastIndexOf方法与indexOf方法类似，但是lastIndexOf方法是从尾部向前查找。


####### string.lastIndexOf

{% highlight JavaScript %}
  var index = 'Hello World'.lastIndexOf('l'); // 9
 {% endhighlight %}                     




###### localeCompare(otherString)

localeCompare用于比较两个字符串，该方法与本机环境相关。如果字符串比otherString小，则返回-1，相等则返回0。


[note]:
该方法一般与数组的sort方法结合使用。


~~~~

####### string.localeCompare

{% highlight JavaScript %}
  ['张学友', '刘德华', '郭富城', '黎明'].sort(function (a, b) {return a.localeCompare(b);}); // ["郭富城", "黎明", "刘德华", "张学友"]
                      
{% endhighlight %}



###### match(regexp)

match方法对字符串匹配一个正则表达式，返回一个匹配数组。

如果regexp不包含g标识，该方法结果与regexp.exec(string)结果相同，包含第一个匹配和其中的分组。如果regexp包含g标识，该方法结果包含所有的匹配。


####### string.match

{% highlight JavaScript %}
  var html = '<html><head><title>title</title></head><body><p>p</p></body></html>';
  html.match(/<[a-z0-9]+(\s+[a-z0-9='"]+)?>/g); // ['<html>', '<head>', '<title>', '<body>', '<p>']
                      

{% endhighlight %}


###### replace(search, replaceValue)

replace方法对字符串进行替换，并返回一个新字符串。


[note]:
search参数可以是一个字符串或一个正则表达式。只有当search是一个带有g标识的正则表达式时，才会替换字符串中所有匹配项，否则只替换第一个匹配项。


~~~~

####### string.replace：替换第一项与全部替换

{% highlight JavaScript %}
  'border_left_style'.replace('_', '-'); // 'border-left_style'，只替换第一项
  'border_left_style'.replace(/_/g, '-'); // 'border-left-style'，替换所有项
  {% endhighlight %}                    

replaceValue参数可以是一个字符串或一个函数。当replaceValue是一个函数时，将对每一个匹配项进行调用，将返回的文本替换匹配文本。


####### string.replace：使用替换函数

{% highlight JavaScript %}
  // 'border-left-style'
  'borderLeftStyle'.replace(/[A-Z]/g, function ($0) {
      return '-' + $0.toLowerCase();
  });
  {% endhighlight %}                    




###### search(regexp)

search方法在字符串内查找regexp的第一个匹配的位置。和indexOf不同的是：


1. 它只接受正则表达式。
2. 它不接受position参数。

####### string.search

{% highlight JavaScript %}
  'type="text"'.search(/"/); // 5
                      
{% endhighlight %}



###### slice(start, end)

slice方法截取字符串的一部分构成新的字符串，从start到end（不包括end）。

start参数为负值时，将与字符串的length相加，如果还是负数则为0。

end参数可选，默认为字符串长度。如果指定一个end，处理规则和start参数一样。


####### string.slice

{% highlight JavaScript %}
  'Hello world!'.slice(0, 5); // Hello
  'Hello world!'.slice(-6, -1); // world
 {% endhighlight %}                     


[info]:
不要使用substring来截取字符串，它的功能和slice完全一样，只是不支持负数作为参数。


~~~~



###### split(separator, limit)

split方法将字符串分割成数组，separator可以是一个字符串，或是一个正则表达式。

limit参数可以限定分割的数量，这个参数不常用。


####### string.split

{% highlight JavaScript %}
  '127.0.0.1'.split('.'); // ['127', '0', '0', '1']
                      
{% endhighlight %}

[info]:
IE下，如果split结果的第一项或最后一项是空字符串，会被直接省略掉。


~~~~



###### substr(start, length)

substr与slice一样都可以用来截取字符串。不同的是substr第二个参数指定的是要截取的字符串长度。


####### string.substr

{% highlight JavaScript %}
  'Hello world!'.substr(6, 5); // world
 {% endhighlight %}                     


[note]:
在某些环境下，substr方法的start参数不支持负数，负数时会默认成0。


~~~~



###### toLowerCase()

toLowerCase方法将字符串所有字母转换为小写格式。


####### string.toLowerCase

{% highlight JavaScript %}
  var str = 'Hello World!'.toLowerCase(); // 'hello world!'
 {% endhighlight %}                     




###### toUpperCase()

toUpperCase方法将字符串所有字母转换为大写格式。


####### string.toUpperCase

{% highlight JavaScript %}
  var str = 'Hello World!'.toUpperCase(); // 'HELLO WORLD!'
    {% endhighlight %}                  




#### 字符串常用静态方法


###### String.fromCharCode(code...)

fromCharCode方法可以从一串参数中返回一个字符串，每个参数对应的是相应位置的unicode码。


####### String.formCharCode

{% highlight JavaScript %}
  String.fromCharCode(20013, 22269); // 中国
                      
{% endhighlight %}


### 数组


#### 复制

数组的复制可以使用slice方法。

使用slice方法是浅复制。如果数组中包含对象，则相应索引的对象引用是同一个对象。如果需要深复制请参考：baidu.object.clone


###### 数组的浅复制

{% highlight JavaScript %}
  [1, 2, 3].slice(0); // [1, 2, 3]
  
  var arr = [{}]
  arr.slice(0)[0] == a[0]; // true
   {% endhighlight %}               


[info]:
避免使用Array.apply的方法复制数组。当只有一个参数时，会被当作初始化的数组长度，导致不期望的结果。


~~~~



#### 删除


###### 删除数组项

删除数组项使用splice方法。


####### 使用splice删除数组项

{% highlight JavaScript %}
  var arr = [1, 2, 3];
  arr.splice(0, 1); 
  alert(arr); // [2, 3]
      {% endhighlight %}                


###### 清空数组

我们可以设置数组的length为0来清除数组的所有项。


####### 使用length清空数组

  
{% highlight JavaScript %}
    var arr = [1, 2, 3];
  arr.length = 0; // []
     {% endhighlight %}                 


[note]:
当length被设置为小于当前长度时，下标大于或等于设置长度的项会被删除。


~~~~

####### 使用length删除数组尾部的项

{% highlight JavaScript %}
  var arr = [1, 2, 3];
  arr.length = 2; // [1, 2]
     {% endhighlight %}                 

#### 遍历


###### 数组长度的变量保存

数组遍历时，应先将数组的length保存到临时变量中，避免在循环的每一步都读取数组的length。


####### 数组遍历：提前存储数组的lenght

{% highlight JavaScript %}
  for (var i = 0, len = list.length; i < len; i++) {
      // ...
  }
{% endhighlight %}


[note]:
如果能保证数组所有项的ToBoolean不为false，可以使用for (var i = 0, obj; obj = list[i]; i++)进行遍历。这种遍历方式对性能有提高。


~~~~



###### 倒序遍历

当遍历的顺序无关时，建议使用倒序遍历。


####### 数组遍历：使用倒序便利

{% highlight JavaScript %}
  var len = list.length;
  
  while (len--) {
      var item = list[len];
  }
   {% endhighlight %}                   


[info]:
查找匹配项并删除时，应使用倒序遍历。


~~~~



#### 排序

在**JavaScript**中，可以使用sort方法对数组进行排序。

在基于比较的排序情况下，不建议自己写排序。因为使用sort排序性能并不比自己写快速排序差，甚至更好。


###### 数值排序

sort方法默认会将数组每一项转换为字符串，如果要按数值排序，需要传递一个比较函数。


####### 使用sort方法进行排序

{% highlight JavaScript %}
  [3, 5, 2, 1].sort(function (a, b) {return a - b;}); // [1, 2, 3, 5]
                      
{% endhighlight %}
###### 中文排序

使用字符串的localeCompare方法，可以对中文进行排序。

localeCompare方法的返回值类似于sort方法比较函数的约定。

localeCompare方法依赖于本地环境。


####### 使用string.localeCompare进行中文排序

  
{% highlight JavaScript %}
    ['张学友', '刘德华', '郭富城', '黎明'].sort(function (a, b) {return a.localeCompare(b);}); // ["郭富城", "黎明", "刘德华", "张学友"]
   {% endhighlight %}                   



#### 常用数组实例方法


###### concat(item...)

concat方法返回一个新数组，并将参数附加在数组后面。如果参数是一个数组，则数组中的每一项会被分别添加。


####### array.concat

{% highlight JavaScript %}
  var arr = [1, 2, 3];
  var newArr = arr.concat([4, 5, 6], 7); // [1, 2, 3, 4, 5, 6, 7]
   {% endhighlight %}                   


###### join(separator)

join方法把数组构造成一个字符串，并以分隔符分隔。默认的分隔符为“,”。


####### array.join

{% highlight JavaScript %}
  var arr = ['Hello', 'world];
  arr.join(' '); // "Hello world"
  {% endhighlight %}                    


###### pop()

pop方法会移除数组最后一项，并返回该项。


####### array.pop

{% highlight JavaScript %}
  var arr = [1, 2, 3];
  var last = arr.pop(); // arr: [1, 2]; last: 3
    {% endhighlight %}                  


###### push(item...)

push方法向数组尾部添加一个或多个项，并返回操作后的数组长度。


[note]:
push与pop配合可使数组像stack一样工作。


~~~~
push与concat有一些不同：


1. push会修改当前数组。
2. 如果参数是一个数组，push会把参数作为一项添加到当前数组中。

####### array.push

  
{% highlight JavaScript %}
    var arr = [1, 2, 3];
  arr.push([4, 5, 6], 7); // [1, 2, 3, [4, 5, 6], 7]
     {% endhighlight %}                 

###### shift()

shift方法会移除数组第一项，并返回该项。


####### array.shift

{% highlight JavaScript %}
  var arr = [1, 2, 3];
  var first = arr.shift(); // arr: [2, 3]; first: 1
   {% endhighlight %}                   


###### slice(start, end)

slice返回数组的浅复制。索引从start开始到end（不包括end）。

如果start或end是负数，解析器会试图把他们与数组的length相加，使其成为非负数。


####### array.slice

{% highlight JavaScript %}
  var arr = [1, 2, 3, 4];
  arr.slice(1, 3); // [2, 3]
 {% endhighlight %}                     


###### sort(compareFunction)

sort方法可以对数组进行排序。默认会按照字符串的方式进行排序。


####### array.sort：默认按字符串排序

{% highlight JavaScript %}
  [1, 4, 8, 10].sort(); // [1, 10, 4, 8]
 {% endhighlight %}                     

我们可以传递一个比较函数。该函数对数组中的两项进行比较，相等时返回0，如果想要第二个参数排在前面，则返回一个正数。


####### array.sort：通过比较函数实现数值排序

{% highlight JavaScript %}
  [7, 4, 6, 2].sort(function (a, b) {return b - a;}); // [7, 6, 4, 2]
                      
{% endhighlight %}

###### splice(start, deleteCount, item...)

splice方法最大的作用是移除数组的项。其还能在删除项的位置插入新的item。

splice方法会以一个新的数组返回删除的项。


####### array.splice

{% highlight JavaScript %}
  var arr = [1, 2, 3, 4, 5, 6];
  arr.splice(1, 2, 7); // [1, 7, 4, 5, 6]
   {% endhighlight %}                   


###### unshift(item...)

unshift方法向数组开始部分添加一个或多个项。


[note]:
unshift与pop配合可使数组像队列一样工作。


~~~~

####### array.unshift

{% highlight JavaScript %}
  var arr = [3, 4, 5];
  arr.unshift(1, 2); // [1, 2, 3, 4, 5]
                      
{% endhighlight %}
### 面向对象


#### 静态类

通常我们使用静态类封装同一类型或同一业务相关的功能。静态类在**JavaScript**里使用频率远高于可实例化的类。静态类在**JavaScript**里表现为Object。


###### 声明静态类

{% highlight JavaScript %}
  var Util = {
      formatDate: function (date) {
      },
          
      encodeHTML: function (source) {
      }
  };
                  
{% endhighlight %}
使用function执行并返回，可以达到变量私有的效果。


[note]:
该方法可以提高Javascript代码压缩的效果。因为局部变量是可以被压缩的，而全局变量以及对象成员是无法被压缩的。


~~~~

###### 声明静态类：通过闭包

{% highlight JavaScript %}
  var Util = function () {
      var format = "yyyy-MM-dd"; // 私有变量，不对外暴露
      
      return {
          formatDate: function (date) {
          },
          
          encodeHTML: function (source) {
          }
      };
  }();
                  
{% endhighlight %}

[info]:
如果返回对象是function或包含对function的引用，则当前执行的scope不会被释放。使用该方法请遵循闭包原则。


~~~~



#### 继承


###### 使用原型继承

在继承的实现上，通常我们利用**JavaScript**的语言特性，使用原型继承。

原型继承的优点是：


1. 做类型判断的时候，dog is a Animal。这点如果使用属性复制法，则无法实现。
2. 多实例使用一个原型对象，减少内存开销。
原型继承的缺点是：


1. 由于子类原型是父类的实例，所以实例化的时候需要关注构造函数的参数。
2. 子类构造函数和方法需要指定父类的名称来进行super的调用，在子类编写的过程中都需要关心父类的名称。如SuperClass.call或SuperClass.prototype.method.call。

####### 原型继承

{% highlight JavaScript %}
  function Animal(name) {
      this.name = name;
  }
      
  Animal.prototype = {
      jump: function () {
          alert('animal ' + this.name + ' jump');
      }
  };
      
  function Dog(name) {
      Animal.call(this, name);
  }
      
  Dog.prototype = new Animal();
      
  Dog.prototype.jump = function () {
      alert('dog ' + this.name + ' jump');
  };
                      
{% endhighlight %}


###### 原型继承的风险

由于原型对象的成员被所有实例共享，所以编码是我们应该遵守这样的原则：原型对象只包含程序不会修改的成员，如方法函数或配置项。

下面是一个简单的风险例子：


####### 原型继承的风险

{% highlight JavaScript %}
  function ListBase() {
      this.container = [];
  }
  
  ListBase.prototype = {
      push: function (item) {
          this.container.push(item);
      },
          
      alert: function () {
          alert(this.container);
      }
  };
  
  function List() {
  }
  
  List.prototype = new ListBase();
  
  var list1 = new List();
  var list2 = new List();
  
  list1.push(1);
  list2.push(2);
  list2.alert();  // 1,2
  {% endhighlight %}                    



### DOM

由于dom对象不是**JavaScript**原生对象，所以我们在使用dom对象的时候，可能会面临这样的麻烦：性能、浏览器兼容性、内存泄露等。


#### 获取元素


###### 获取单个元素

通常，我们使用document.getElementById来获取dom元素，避免使用document.all。document.getElementById是标准方法，兼容所有浏览器。


[info]:
ie浏览器会混淆元素的id和name属性，document.getElementById可能获得不期望的元素。在对元素的id与name属性的命名需要非常小心，应使用不同的命名法。下面是一个name与id冲突的例子：


~~~~

####### id与name的冲突

  :::html
  <input type="text" name="test">
  <div id="test"></div>
  
  <!-- ie6下为INPUT -->
  <button onclick="alert(document.getElementById('test').tagName)"></button>
                      


###### 获取元素集合


1. getElementsByTagName
  
  getElementsByTagName(tagName)方法可以根据标签名获得元素下的子元素，包含非直接附属的子元素。我们可以指定tagName参数为'*'来获得元素的所有子元素。
  
  
  
  
    :::html
        <!-- dom操作实时反映到结果集合中 -->
  
    <body>
    <div></div>
    <span></span>
    
    <script>
    var elements = document.body.getElementsByTagName('*');
    alert(elements[0].tagName); // div
    
    var div = elements[0];
    var u = document.createElement('u');
    
    document.body.insertBefore(u, div);
    alert(elements[0].tagName); // u，实时反应到结果中
    </script>
    </body>
                                

  
2. childNodes
  
  childNodes属性可以获取控件的直接子节点集合。集合中包括文本、注释、属性等类型的节点。
  
  
3. children
  
  children属性可以获取控件的直接子元素集合。集合中只包含html元素。

[info]:
  以上方法或属性的结果并不直接引用dom元素，而是对索引进行读取，所以dom结构的改变会实时反映到结果中。使用了getElementsByTagName方法需要小心dom操作的时序性。
  
  
~~~~




#### 样式读取

样式读取的陷阱很多。比如通过class具有的样式无法被直接读取，还有对颜色的读取在不同浏览器下的表示方式存在差异，等等。所以我们应该遵循如下建议：


1. 不以样式判断状态，也不建议以className判断状态。状态应被单独管理。这样可以减少读取样式的可能性，避免差异发生。
2. 如果非要读取元素实时样式，使用tangram的baidu.dom.getStyle方法。



#### 样式设置


###### className

通过设置元素的className属性可以为元素设置class。该方式兼容所有浏览器。


####### 设置元素的class

  
{% highlight JavaScript %}
    dom.className = 'my-class';
    {% endhighlight %}                  


[info]:
避免使用setAttribute方法设置元素的class。


~~~~



###### style

通过style可以给元素设置样式。通常css样式对应的设置名称为：去掉单词连接的中划线，并使用驼峰命名法。如：margin-left -> marginLeft。


####### 设置元素的style

{% highlight JavaScript %}
  dom.style.top = '10px';
                      
{% endhighlight %}

###### 一个例子：display的管理

通常我们认为style.display = ''可以设置一个元素为显示。但是当元素具有的class中包含display:none的定义时，必须设置为block才能显示该元素。这对我们带来了困扰。

我们可以通过下面的方法解决：


1. 不将display:none定义在class中，而定义在元素的style属性中。display作为可变的样式定义，不应该被定义在元素的显示样式规则中。
  
  
    :::html
        <!-- 可变样式定义在标签内内联中 -->
    
    <div class="myclass" style="display:none"></div>
                                
  
  
2. 使用一个额外的class（如myclass-hide）来控制显示隐藏，在这个class中定义display:none，需要隐藏时让元素多具有一个class。
  
  
  {% highlight JavaScript %}
        // 使用额外的class管理可变样式
      
    baidu.addClass(dom, 'myclass-hide');
                                
{% endhighlight %}
[info]:
避免通过获取元素的当前display样式来进行显示隐藏。获取当前样式会导致性能问题。我们应该规划管理我们的样式。


~~~~



#### 性能

提高dom操作的性能有一个主要原则：减少dom操作，减少浏览器的reflow。


###### 减少dom操作

举一个简单的例子：构建一个列表。我们可以用两种方式：


1. 在循环体中createElement并append到父元素中。
2. 在循环体中拼接html字符串，循环结束后为父元素赋innerHTML。
第一种方法看起来比较标准，但是每次循环都会对dom进行操作，性能极低。在这里推荐使用第二种方法。




###### 减少浏览器的reflow

下面一些场景会触发浏览器的reflow：


1. DOM元素的添加、修改（内容）、删除。
2. 应用新的样式或者修改任何影响元素布局的属性。
3. Resize浏览器窗口、滚动页面。
4. 读取元素的某些属性（offsetLeft、offsetTop、offsetHeight、offsetWidth、scrollTop/Left/Width/Height、clientTop/Left/Width/Height、getComputedStyle()、currentStyle(in IE)) 。
下面的例子是一个频繁触发reflow的例子：


####### 一个频繁触发reflow的例子

{% highlight JavaScript %}
  var parent   = document.getElementById('parent');
  var elements = parent.getElementsByTagName('li');
  var len      = elements.length;
  var i;
      
  for (i = 0, ; i < len; i++) {
      elements[i].style.width = parent.offsetWidth + 'px';
  }
                      
{% endhighlight %}
在这个例子中，循环的每一步都读取了parent元素的offsetWidth，所以触发reflow进行重新渲染了len次。这个例子中，更好的做法是读取一次，较少reflow。


####### 一个频繁触发reflow的例子的改进

  
  var parent   = document.getElementById('parent');
  var elements = parent.getElementsByTagName('li');
  var width    = parent.offsetWidth;
  var len      = elements.length;
  var i;
      
  for (i = 0, ; i < len; i++) {
      elements[i].style.width = width + 'px';
  }
                      

[更多关于reflow的资料请见这里](http://blog.csdn.net/baiduforum/archive/2010/03/25/5415527.aspx)




### DOM事件


#### 浏览器差异性

IE与标准浏览器在事件处理上有很明显的差异，这些差异为浏览器端开发带来了很大的麻烦：


1. EventArgument
  
  IE通过window.event获取EventArgument；标准浏览器通过监听器的第一个参数获取EventArgument。
  
  EventArgument对象本身也存在差异性，最典型的是IE通过srcElement属性获得触发事件的元素，标准浏览器则是target属性。
  
  
2. 事件触发
  
  IE的事件触发为冒泡模型；标准浏览器的触发为捕获-冒泡模型。
  
  
3. 监听方法
  
  ie使用attachEvent方法添加监听器；标准浏览器使用addEventListener方法添加监听器。



#### 添加事件处理的方法

通常，为一个dom元素添加事件处理，可以使用3种方法：


1. 标签内联
  
  这种方法最简单，但是在html中书写行为，违背了结构-表现-行为分离的原则。
  
  
    :::html
        <!-- 添加事件处理：标签内联 -->
  
    <div onclick="callFunction()"></div>
                            
  
  
2. expando属性
  
  这种方法比较通用，但是如果为一个元素挂载多个event handler的话，后面的会覆盖前面。
  
  
  {% highlight JavaScript %}
        // 添加事件处理：expando属性
  
    div.onclick = function (e) {
    };
                            
  {% endhighlight %}
  
3. 添加监听器
  
  这种方法比较流行，但是浏览器存在差异，并可能带来性能问题。


[info]:
  标准的addEventListener提供了两种时间触发模型：冒泡和捕获。可以通过第三个参数指定。IE的attachEvent支持冒泡的事件触发。所以为了保持一致性，通常addEventListener的第三个参数都为false。
  
  事件类型中，标准浏览器不需要添加“on”，如“click”；IE需要添加为“onclick”。
  
  IE下使用attachEvent挂载监听器，当事件被触发时，函数的指针指向window，而不是触发事件的元素本身。
  
  
~~~~
  
  {% highlight JavaScript %}
        // 添加事件处理：添加监听器
  
      
    function listener(e) {}
        
    if (baidu.ie) {
        dom.attachEvent('onclick', listener);
    } else {
        dom.addEventListener('click', listener, false);
    }
                            
{% endhighlight %}

[note]:
  为了屏蔽浏览器兼容性，我们可以使用baidu.event.on
  
  
~~~~




#### 添加事件原则


1. 拼接html时，可以使用标签内联的方法添加事件。
  
  有时候我们需要用**JavaScript**拼接html字符串。这个时候我们面对的不是dom对象，可以使用标签内联的方式添加事件。标签内联的方法可以不用关心事件的释放。

  标签内联的方式使得我们丧失了对事件控制的权利。我们没有办法获得触发事件的target，没有办法停止事件冒泡，没有办法停止默认行为。我们唯一能做的就是使用this把当前dom元素传递给处理函数。
  
  
  
2. 运行环境可控时，使用expando属性的方法添加事件。
  
  这个时候，**JavaScript**运行的页面环境全部是由我们控制的，我们不担心会有其他的**JavaScript**脚本覆盖我们挂载的event handler。
  
  对于大多数项目，使用这种方法代替流行的addEventListener，因为该方式会获得更高的性能（如document.body.onmousemove的拖拽处理）
  
  
3. 对于页面级别的事件管理，使用添加监听器的方法。
  
  有时我们需要在点击页面时关闭所有的浮动层。类似的情况向body添加监听器更容易实现，相互之间无影响。
  
  
  对于元素级别事件挂载：有的时候一个项目由多人开发，大家为了不覆盖相互对同一个元素的事件处理而添加监听器。但这种情况大多是因为对交互的分析、事件的管理没有很好的规划。
  
  
4. 针对第三方环境时，使用添加监听器的方法。
  
  添加监听器的方法能够不覆盖之前对响应元素添加的监听器。不过监听器之间的处理顺序无法保证。
  
  开发人员需要持有监听器函数的引用，在适当时候（元素释放、页面卸载等）卸载添加的监听器。



### 页面加载与代码构建


#### 页面加载

通常我们在body标签结束之前加载外部js。在这个位置加载js的好处是：


1. 让用户预先下载可见的html视图，避免下载js的阻断导致用户面对空白页面等待。
2. 保证页面的dom结构已经加载完成，操作dom可以不用顾忌或判断是否加载完毕。

###### 加载外部js

  
  :::html
    <body>
      <div class="wrap">...</div>
      <script type="text/**JavaScript**" src="my.js"></script>
  </body>
                  


[info]:
script标签中必选的属性为type与src。


~~~~
下面的一些属性虽然常用，但已不推荐：


1. defer属性为ie only的属性，能延后脚本的执行时间，不推荐。
2. language属性已被html5标准删除，不推荐。
3. charset属性可以通过脚本文件本身编码来解决，不推荐。



#### 分模块与分文件开发

通常，一个项目中我们可能设计公共模块与业务模块划分，开发时我们会划分模块，并分成多个**JavaScript**文件。

关于模块划分，有如下建议：


1. 公共模块以一个namespace暴露。可参考tangram。
2. 业务模块是独立的业务单元，以一个namespace来暴露。模块中包括模块公共部分（语言、公共函数等），以及各个业务功能。
3. charset属性可以通过脚本文件本身编码来解决，不推荐。
关于文件划分：开发时，我们可以使用一个js文件来装载要用到的**JavaScript**文件（如build.js）。这样做的好处是，在提测前构建的时候，无需修改相应的html或模板。


###### 开发时外部js装载

  
{% highlight JavaScript %}
    document.write('<script type="text/**JavaScript**" src="/src/UIBase.js"></script>');
  document.write('<script type="text/**JavaScript**" src="/src/UIManager.js"></script>');
  document.write('<script type="text/**JavaScript**" src="/src/ui/MonthView.js"></script>');
  document.write('<script type="text/**JavaScript**" src="/src/ui/Calendar.js"></script>');
  document.write('<script type="text/**JavaScript**" src="/src/ui/Link.js"></script>');
  document.write('<script type="text/**JavaScript**" src="/src/ui/Button.js"></script>');
  document.write('<script type="text/**JavaScript**" src="/src/ui/TextInput.js"></script>');
  document.write('<script type="text/**JavaScript**" src="/src/ui/BaseBox.js"></script>');
  document.write('<script type="text/**JavaScript**" src="/src/ui/CheckBox.js"></script>');
  document.write('<script type="text/**JavaScript**" src="/src/ui/RadioBox.js"></script>');
  // ......
                  
{% endhighlight %}

[info]:
切勿以document.write输出静态js引用的方式上线！在IE下，document.write无法保证脚本加载的时序性。经过测试，我们认为对于同域的静态资源，时序型可以保证，所以开发时可以采用这种方式。


~~~~



#### 自动构建

提测时，我们需要一些脚本来自动将这些文件的具体内容打包到这个文件中，并且进行压缩。这个过程避免手工。想想，如果手工合并文件，是一件多麻烦的事情，在每个提测前可能都要干一遍......

我们可以使用shell、ant、wscript、python等脚本进行合并。下面是一个简单的shell构建脚本的例子，这个例子中build.js会被打包压缩成build-c.js，为了避免文件被覆盖，需要手工备份build.js，mv build-c.js。当然我们也可以不备份而从svn中恢复，但那样我们需要解决svn冲突。

下面是一个简单的自动构建的shell脚本示例：


###### 自动构建的shell脚本

  
  #! /bin/sh
  
  cd ..
  
  # merge
  cat assets/build.js | 
      awk -F'"' '/src="[^"]+.js"/{print $4}' |
          cut -c 2- |
              xargs cat 1>assets/build-all.js
      
  # compress
  java -jar ~/local/ecui.jar assets/build-all.js --mode 2 --charset utf-8 -o assets/build-c.js
                  






### 第三方**JavaScript**

有时候，我们开发的**JavaScript**是为了给第三方页面提供服务的（广告、开放api...），我们的js会被运行在各种各样的页面环境中。相比环境完全可控的js开发，我们需要关注一些额外的东西：


#### 全局变量冲突

由于环境的未知性，所有暴露的全局变量都可能产生危险。我们应该使用function隔离作用域。


###### 隔离作用域，防止变量冲突

  
{% highlight JavaScript %}
    (function () {
      var i = 0;
  })();
      {% endhighlight %}            

对于提供api而必须暴露的全局变量，首先减少暴露的个数，以1个为宜。通过挂载到window的property的方式暴露。


###### 全局变量暴露

  
{% highlight JavaScript %}
    (function () {
      function BaiduClass() {
      }
          
      window.BaiduClass = BaiduClass;
  })();
                  
{% endhighlight %}


#### 字符编码

第三方页面使用的字符集编码是无法确定的，可能是gbk、utf-8等等。外联的js如果编码与页面编码不一致，可能导致问题。

解决办法：将ascii大于127的字符（如中文字符），使用unicode进行转码，保证代码中不包含ascii大于127的字符。


###### 字符编码转换

  
{% highlight JavaScript %}
    // 原始代码
  var str = '中国';
      
  // unicode转码后：
  var str = '\u4e2d\u56fd';
                  
{% endhighlight %}

[info]:
unicode转码能带来字符编码的安全性。但是对于脚本执行环境编码可控的页面，不建议进行unicode转码。因为会给js文件增加额外的字节大小。


~~~~



#### dom的创建

在第三方页面中创建dom元素，有两种可能：与用户页面本身dom结构有关或无关。


1. 与用户页面dom结构有关的情况，比如需要在引入脚本的位置创建元素时，使用document.write：
  
  
    // 与用户页面dom结构有关时，使用document.write创建元素
    
    document.write('<div>myHTML</div>');
                          


2. 与用户页面dom结构无关的情况，比如绘制一个提供服务的浮动层时，应在用户页面load完成后，使用createElement创建：
  
  
    // 与用户页面dom结构无关时，在用户页面load完成后，使用createElement创建
    
    baidu.dom.ready(function () {
      var div = document.createElement('div');
      document.body.appendChild(div);
    });
                            


[info]:
由于window.onload事件的触发时机是页面完全加载(包括图片等资源)，所以建议使用DOMContentLoaded事件，在dom树在页面中构建完成后即可创建元素。该事件非所有浏览器都支持，详细方案见baidu.dom.ready。


~~~~

[info]:
绝对定位的元素应创建于body标签下。切忌创建在未知的当前区域，有可能因为处于表格内部或其他绝对定位元素内，导致定位错误。

~~~~

### 参考

* {: id='[1]'}[ES6: const](https://people.mozilla.org/~jorendorff/es6-draft.html#sec-let-and-const-declarations)
* {: id='[2]'}[JSDoc](http://usejsdoc.org/)
* {: id='[3]'}[JavaScript tricks 2: Use `===` instead of `==`](/frontend/javascript/tricks/2015/02/14/javascript-tricks-2-use-===-instead-of-==/)
* {: id='[4]'}[Function.prototype.toString()](http://es5.github.io/#x15.3.4.2)
* {: id='[5]'}[Function Decompilation](http://kangax.github.io/jstests/function-decompilation/)
* {: id='[6]'}[JSPerf: RegExp Literal vs RegExp Object](http://jsperf.com/regex-literal-vs-object)
* {: id='[7]'}[Understanding Delete](http://perfectionkills.com/understanding-delete/)

### 参考书籍

* [Professional JavaScript for Web Developers](http://www.amazon.com/Professional-JavaScript-Developers-Nicholas-Zakas/dp/1118026691/ref=sr_1_13?tag=nczonline-20)
* [Speaking JavaScript](http://speakingjs.com/es5/index.html)
* [JavaScript: The Good Parts](http://www.amazon.com/gp/product/0596517742?ie=UTF8&camp=213733&creative=393185&creativeASIN=0596517742&linkCode=shr&tag=ericleads-20&linkId=IJKESYSOTWGC27DR)
* [Effective JavaScript](http://www.amazon.com/gp/product/0321812182?ie=UTF8&camp=213733&creative=393185&creativeASIN=0321812182&linkCode=shr&tag=ericleads-20&linkId=JIC63I267I6UDQQZ)
* [Expert JavaScript](http://www.amazon.com/Expert-JavaScript-Experts-Voice-Development/dp/1430260971/ref=sr_1_1?s=books&ie=UTF8&qid=1422710124&sr=1-1&keywords=Expert+JavaScript)
