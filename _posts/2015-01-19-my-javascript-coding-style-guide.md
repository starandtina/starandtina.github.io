---
layout: post
title: "My JavaScript Coding Styleguide"
description: "My JavaScript Coding Styleguide"
category: ['JavaScript', 'Styleguide']
tags: ['JavaScript', 'Styleguide']
---
{% include JB/setup %}

本文定义了个人的JavaScript编码规范。
{: .countheads }

* ToC
{:toc}

- *必须*指代*MUST*，拥有强制性
- *不得*指代*MUST NOT*，拥有强制性
- *应当*指代*SHOULD*，不拥有强制性但建议实施，建议各项目组进一步明确定义
- *可以*指代*MAY*，不拥有强制性但建议实施，建议各项目组进一步明确定义

----

### 行长度

每一行代码严格以**80**字符为最大长度，即一行包括前后的空格，不得超过**80**个字符。

由于必定存在一些特殊的原因，导致一条语句的长度超过80，*应当*按以下原则进行切分：

#### 字符串过长

按一定长度截断字符串，并使用`+`运算符进行连接。分隔字符串尽量按语音进行，如不要在一个完整的名词中间断开。

特别的，对于HTML片段的拼接，通过缩进，保持和HTML相同的结构：

{% highlight JavaScript %}

var html = '' // 此处用一个空字符串，以便整个HTML片段都在新行严格对齐
    + '<article>'
    +     '<h1>Title here</h1>'
    +     '<p>This is a paragraph</p>'
    +     '<footer>Complete</footer>'
    + '</article>';

{% endhighlight %}

也可使用数组来进行拼接，相对`+`运算更容易调整缩进：

{% highlight JavaScript %}

var html = [
    '<article>',
        '<h1>Title here</h1>',
        '<p>This is a paragraph</p>',
        '<footer>Complete</footer>',
    '</article>';
];
html = html.join(''); // 注意需要join

{% endhighlight %}

#### 函数调用时参数过多

当参数过多，在一行书写函数调用语句会超过80个字符的限制时，*应当*将每个参数独立写在一行上，并将结束的右括号`)`独立一行，所有参数**增加一个缩进**，以控制每行的长度，如：

{% highlight JavaScript %}

// 注意每一个参数的缩进
foo(
    aVeryVeryLongArgument,
    anotherVeryLongArgument,
    callback
);

{% endhighlight %}

当参数较多，一行一个书写会导致过长时，*应当*按逻辑对参数进行组合，最经典的是`jQuery.format`函数，如：

{% highlight JavaScript %}

// 将参数分为“模板”和“数据”两块
jQuery.format(
    dateFormatTemplate,
    year, month, date, hour, minute, second
);

// 将参数分为“模板”、“日期”和“时间”
jQuery.format(
    dateFormatTemplate,
    year, month, date, 
    hour, minute, second
);

{% endhighlight %}

#### 三元运算符过长

三元运算符由3部分组成，因此其换行*应当*根据每个部分的长度不同，形成3种不同的情况：

{% highlight JavaScript %}
    // 无需换行
    var result = condition ? resultA : resultB;

    // 条件超长的情况
    var result = thisIsAVeryVeryLongCondition ?
        resultA : resultB;

    // 结果分支超长的情况
    var result = condition ?
        thisIsAVeryVeryLongResult :
        resultB;
    var result = condition ?
        resultA :
        thisIsAVeryVeryLongResult;
{% endhighlight %}

*不得*出现以下情况：

{% highlight JavaScript %}
    // 最后一个结果很长，但不建议合并条件和第一分支
    // 不要这么干
    var result = condition ? resultA :
        thisIsAVeryVeryLongResult;
{% endhighlight %}

这种方法会导致语义上的分裂，即“条件和分支”在一行，“另一分支”在一行，没有按逻辑进行组织。

#### 过长的逻辑条件组合

当因为较复杂的逻辑条件组合导致80个字符无法满足需求时，考虑将每个条件独立一行，逻辑运算符**放置在行尾或行首**进行分隔，或将部分逻辑按逻辑组合进行分隔，最终将右括号`)`与左大括号`{`放在独立一行，如：

{% highlight JavaScript %}

// 注意逻辑运算符前的缩进
if (user.isAuthenticated()
    && user.isInRole('admin')
    && user.hasAuthority('add-admin')
    || user.hasAuthority('delete-admin')
) {
    // Code
}

{% endhighlight %}

### 空格

在特定的位置加上空格有助于代码的可读性，以下位置*必须*加上空格：

- 除括号外，所有运算符的前后，如`+`, `-`等等
- 用作代码块起始的左大括号`{`前，包括`if`、`else`、`try`、`finally`这些关键字之后，以及**函数定义的参数列表**之后
- 以下关键字之后：`for`、`switch`、`while`
- 对象初始化（`{ ... }`）的每个属性名的冒号`:`后
- 所有逗号`,`后
- 单行的对象初始化（`{ ... }`）左大括号`{`后和右大括号`}`前

注意，在**函数参数列表**之前*不得*加上空格，以下是一个函数的正确声明方式：

{% highlight JavaScript %}

function foo(x, y, z) {
    // FunctionBody
}

var foo = function(x, y, z) {
    // FunctionBody
}

{% endhighlight %}

### 对齐和缩进

严格采用**4个空格**为一次缩进，*不得*采用TAB作为缩进。在以下情况下*必须*缩进：

- 一个代码块与其父代码块相比*必须*多一次缩进
- 未结束的语句在换行后*必须*多一次缩进
- `switch`下的`case`和`default`*必须*缩进

缩进会带来自然的对齐，在缩进之外，不需要额外的对齐：

- 多个变量声明时，*不得*对齐`=`符号
- 初始化对象时，*不得*对齐`:`符号

### 换行

在以下位置*必须*换行：

- 每个独立语句结束后
- `if`、`else`、`catch`、`finally`、`while`等关键字前
- 运算符处换行时，运算符*必须*在新行的行首

对于因为单行长度超过限制时产生的换行，参考**行长度**中的策略进行分隔。

除此之外，存在额外的策略：

#### 过长的JSON和数组

如果对象属性较多导致每个属性一行占用空间过大，*可以*按语义或逻辑进行分组的组织，如：

{% highlight JavaScript %}

// 英文-数字的映射
var mapping = {
    one: 1, two: 2, three: 3, four: 4, five: 5,
    six: 6, seven: 7, eight: 8, nine: 9, ten: 10,
    eleven: 11, twelve: 12, thirteen: 13, fourteen: 14, fifteen: 15,
    sixteen: 16, seventeen: 17, eighteen: 18, nineteen: 19, twenty: 20
};

{% endhighlight %}

通过5个一组的分组，将每一行控制在合理的范围内，并且按逻辑进行了切分。

对于项目较多的数组，也*可以*采用相同的方法，如：

{% highlight JavaScript %}

// 数字-英文的映射
var mapping = [
    'one', 'two', 'three', 'four', 'five',
    'six', 'seven', 'eight', 'nine', 'ten',
    'eleven', 'twelve', 'thirteen', 'fourteen', 'fifteen',
    'sixteen', 'seventeen', 'eighteen', 'nineteen', 'twenty'
];

{% endhighlight %}

#### 单行和跨行参数混用

当函数调用时，传递的参数**大于或等于2个**，且**有一个或以上参数跨越多行**时，要求每一个参数独立一行，这通常出现在匿名函数或者对象初始化等作为参数时，如`setTimeout`函数等：

{% highlight JavaScript %}

setTimeout(
    function() {
        alert('hello');
    },
    200
);

order.data.read(
    'id=' + me.model.id, 
    function(data) {
        me.attchToModel(data.result);
        callback();
    }, 
    300
);

{% endhighlight %}

如果认为每个参数单独一行占用过多的空间，则*应当*将跨域多行的参数独立出来为一个变量：

{% highlight JavaScript %}

function attachDataToModel() {
    me.attchToModel(data.result);
    callback();
}
order.data.read('id=' + me.model.id, attachDataToModel, 300);

{% endhighlight %}

#### 数组和对象初始化的混用

严格按照每个对象的起始`{`和结束`}`在独立一行的风格书写，如：

{% highlight JavaScript %}

var array = [
    {
        // ...
    },
    {
        // ...
    }
];

{% endhighlight %}

### 命名

命名的方法通常有以下几类：

- camel命名法，形如`thisIsAnApple`
- pascal命名法，形如`ThisIsAnApple`
- 下划线命名法，形如`this_is_an_apple`
- 中划线命名法，形如`this-is-an-apple`

根据不同类型的内容，采用不同的命名法：

- 变量名，使用camel命名法
- 参数名，使用camel命名法
- 函数名，使用camel命名法
- 方法/属性，使用camel命名法
- 私有（保护）属性、函数以下划线`_`开头
- 常量名，使用全部大写的下划线命名法，如`IS_DEBUG_ENABLED`
- 类名，使用pascal命名法
- 枚举名，使用pascal命名法
- 枚举的属性，使用全部大写的下划线命名法
- 命名空间，使用camel命名法

命名同时还需要关注语义，如：

- 变量名*应当*使用名词
- **boolean**类型的*应当*使用**is**、**has**等起头，表示其类型
- 函数名*应当*用动宾短语
- 类名用*应当*名词

### 语法

#### 变量声明

一个`var`关键字*必须*只**声明一个变量**。

变量*必须*即用即声明，*不得*在函数（或其它形式的代码块）起始位置统一声明所有变量。

##### 字符串

字符串*必须*使用单引号`'`。

##### 对象和数组

对象和数组优先使用其初始化器（`{ ... }`和`[ ... ]`）声明，不要使用`new Object`和`new Array`。

如果一个对象的所有属性名均不是保留字，则该对象声明时，所有属性名*不得*添加引号，这包括属性名是数字的情况。

如果一个对象的属性名有一个或多个是保留字，则该对象声明时，**所有**属性名*必须*添加引号。

#### 异常

允许使用异常，但在创建`Error`对象时，*必须*明确地传递`message`参数。

#### 继承

采用`jQuery.inherit`或相似机制实现继承，不得随意覆盖`prototype`属性。

声明类的方法时，采用`MyClass.prototype.foo = function() {}`的形式，不得直接覆盖`prototype`属性。

### 其它

#### 修改内置原型

需要修改内置对象的原型时，该代码必须由一个高级别工程师进行Review。

#### `eval`和`with`

当代码中使用`eval`或`with`时，该代码必须由一个同级别工程师和一个高级别工程师进行Review。
