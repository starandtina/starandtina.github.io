---
layout: post
title: 'JSON Data Specification'
description: 'JSON Data Specification'
category: ['Front-End', 'JavaScript']
tags: ['JavaScript', 'JSON', 'js-dev']
---
{% include JB/setup %}

本文主要介绍**JSON**数据传输标准。
{: .countheads }

* ToC
{:toc}

### 简介

JSON是前后端事实上的数据传输标准。JSON依托于http协议（rfc2616）与JSON数据交换格式（[rfc4627](http://www.ietf.org/rfc/rfc4627.txt)）。

#### 设计目标

使业务系统向浏览器端传递的JSON数据保持一致，容易被理解和处理，并兼顾传输的数据量。

#### 要求

在本文档中，使用的关键字会以中文+括号包含的关键字英文表示：必须（MUST）。关键字"MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL"被定义在rfc2119中。

### JSON数据类型

JSON（JavaScript Object Notation）是一种轻量级，基于文本，语言无关的数据交换格式。其包括了基本数据类型4种和复合数据类型2种，共6种数据类型。在下面章节中，JSON数据类型的表示法为JSON+空格+数据类型，如：JSON Array。

传输的数据，包括对象属性以及数组成员，必须（MUST）是6种JSON数据类型之一。杜绝（MUST NOT）使用function、Date等js对象类型。

#### 基本数据类型

* Number可以表示整数和浮点数。
* Boolean可以表示真假，值为true或false。
* String表示一个字符串。


"true"和true，这两个数据代表的是不同的数据类型。非字符串类型数据输出时一定不要（MUST NOT）为两端加上双引号，否则可能产生不希望的后果（如if中判断"false"的结果是true）。其他容易产生错误的例子如：0和"0"等。

Null通常用于表示空对象。

#### 复合数据类型

Object是无序的集合，以键值对的方式保持数据。一个Object中包含零到多个name/value的数据，数据间以逗号(,)分隔。name为String类型，value可以是任意类型的数据。

Object的最后一个元素之后一定不要（MUST NOT）加上分隔符的逗号，否则可能导致解析出错。
Array(数组)为多个值的有序集合，数组元素间以逗号(,)分隔。

### http响应头

#### status

http响应的status必须（MUST）为200。通常JSON数据被用于通过XMLHttpRequest对象访问，通过javascript进行处理。返回错误的状态码可能导致错误不被响应，数据不被处理。

#### Content-Type

Content-Type字段定义了响应体的类型。一般情况下，浏览器会根据该类型对内容进行正确的处理。对于传输JSON数据的响应，Content-Type推荐（RECOMMENDED）设置为**"text/javascript"**或**"application/json"**。避免（MUST NOT）将Context-Type设置为text/html，否则可能导致安全问题。

Content-Type中可以指定字符集。通常需要（SHOULD）明确指定一个字符集。如果是通过XMLHTTPRequest请求的数据，并且字符编码为UTF-8时，可以不指定字符集。

###### Context-Type示例

{% highlight JavaScript %}
Content-Type: text/javascript;charset=UTF-8
{% endhighlight %}

### 数据字段

返回的数据包含在http响应体中。数据必须（MUST）是一个JSON Object。该Object可能包含3个字段：status，statusInfo，data。

#### status

status字段必须（MUST）是一个不小于0的JSON Number整数，表示请求的状态。这个字段可以（SHOULD）被省略，省略时和为0时表示同一含义。

* 0：表示server端理解了请求，成功处理并返回。
* 非0：表示发生错误。可以（SHOULD）根据错误类型扩展错误码。

###### 一个成功请求的status字段

{% highlight JSON %}
{
  "status": 0,
  "data": "hello world!"
}
{% endhighlight %}

#### statusInfo

statusInfo字段通常（SHOULD）是一个JSON String或JSON Object，表示除了请求状态外server端想要对status做出的说明，使client端能够获取更多信息进行后续处理。这个字段是可选的（OPTIONAL）。下面的两个例子中，statusInfo字段的信息都可以用于client端程序的后续处理，但是粒度和处理方式会有不同。

###### client端参数错误的statusInfo

简单说明的statusInfo：

{% highlight JSON %}
{
  "status": 1,
  "statusInfo": "参数错误"
}
{% endhighlight %}

具有更多信息的statusInfo：

{% highlight JSON %}
{
  "status": 1,
  "statusInfo": {
    "text": "参数错误",
    "parameters": {
        "email": "电子邮件格式不正确"
    }
  }
}
{% endhighlight %}

#### data

data字段可以是除JSON Null之外的任意JSON类型，表示请求返回的数据主体。这个字段是可选的（OPTIONAL）。数据主体data包含了在请求成功时有意义的数据。

###### 一个查询姓名请求的返回数据

{% highlight JSON %}
{
  "status": 0,
  "data": "Lily"
}
{% endhighlight %}

### 数据场景

本章为常见数据场景定义了通用的标准数据格式，用于数据传输和使用。额外地，本章为部分可能大数据量传输的数据场景定义了变通数据格式。变通数据格式可在数据解析阶段转换成标准数据格式。

变通数据格式必须（MUST）是一个JSON Object，其中必须（MUST）包含*type*属性和data属性。*type*属性标识数据类型，便于对数据进行解析；data属性包含变通后的数据。变通数据可以（MAY）包含其他的属性，标识数据的其他扩展信息。

变通数据格式的*type*属性定义了table值。*type*属性可以使用者扩展其他属性值，扩展的属性值必须（MUST）以“项目缩写-名称”命名，如“fc-list”，自主解析。

#### 日期类型

日期类型不属于JSON数据类型。对于日期类型，我们需要（MUST）使用JSON String来表示。为了让日期能够更容易的被显示和被解析，对于日期我们应当（SHOULD）使用更适合internet的格式，遵循[rfc3339](#rfc3339)。

###### 数据场景：日期

{% highlight JSON %}
{
  "status": 0,
  "data": "2010-10-10"
}
{% endhighlight %}

#### 记录

记录代表二维表中的一行，通常用于表示某个具体事务抽象的属性。标准记录数据必须（MUST）为一个JSON Object，记录的主键命名必须（MUST）为“id”。单条记录数据不包含变通数据格式。


###### 数据场景：记录

{% highlight JSON %}
{
  "id": 250,
  "name": "Lucy",
  "sex": 1,
  "age": 18
}
{% endhighlight %}

#### 二维表

二维表类型表识为**Table**，是关系模型的主要数据结构。二维表结构具有变通数据格式。标准二维表数据必须（MUST）以一维JSON Array形式表示，JSON Array中每一项是一个JSON Object，代表一条记录。JSON Object的每个成员代表一个字段。每条记录的主键命名必须（MUST）为**"id"**。

在标准二维表中，字段名在每条记录中都被传输，会造成额外的数据量传输。这个问题会随着记录数的增大会更加突出。为了减少传输数据量，变通格式使用二维JSON Array传输数据，扩展fields属性用于字段说明。fields字段为JSON Array。


###### 数据场景：标准二维表

{% highlight JSON %}
[
  {
    "id": 250,
    "name": "Lucy",
    "sex": 1,
    "age": 18
  },
  {
    "id": 251,
    "name": "Lily",
    "sex": 1,
    "age": 28
  }
]
{% endhighlight %}

###### 数据场景：变通二维表

{% highlight JSON %}
{
  "*type*": "table",
  "fields": ["id", "name", "sex", "age"],
  "data": [
    [250, "lucy", 1, 18],
    [251, "Lily", 1, 28]
  ]
}
{% endhighlight %}

#### 数据页

数据页是列表数据常用的数据方式，可能通过查询或翻页获得数据。数据页是二维表数据的包装，包含列表数据本身更多的信息。

数据页必须（MUST）是一个JSON Object，其中必须（MUST）包含的属性为*data*。*data*是一个二维表。数据页可以包括一些可选（OPTIONAL）的属性，表示当前数据页的信息。下表列举了数据页的可选属性。

#### 数据页可选属性

|    属性名     |      描述     |
| :-------------: | :------------- |
| page | 当前页码，类型为JSON Number，计数必须（MUST）为不小于0的整数，从0开始。 |
| pageSize | 每页显示条数，类型为JSON Number，必须（MUST）大于0。 |
| total | 列表总记录数，类型为JSON Number，必须（MUST）为不小于0的整数。表示当前条件下所有记录的数目，非本页的记录数。 |
| orderBy | 列表排序规则，类型为JSON String。多个排序规则之间以逗号分割（,）；正序或倒序以asc或desc表示，与字段名之间以一个空格间隔。例："id desc,name asc" |
| keyword | 列表所属的搜索关键字，类型为JSON String。 |
| condition | 列表所属的搜索条件集合，类型为JSON Object。属性中可以包含或不包含keyword字段，如果不包含，建议（RECOMMMANDED）在解析的时候附加搜索关键字keyword条件。 |
{: .neat }

###### 数据场景：数据页

{% highlight JSON %}
{
  "page": 1,
  "pageSize": 30,
  "keyword": "",
  "total": 100,
  "orderBy": "name",
  "data": [
    {
      "id": 250,
      "name": "Lucy",
      "sex": 1,
      "age": 18
    },
    {
      "id": 251,
      "name": "Lily",
      "sex": 1,
      "age": 28
    }
  ]
}
{% endhighlight %}

#### 键/值对象

对于在一个JSON Object中表示键/值：

+ 键的属性名必须（MUST）为name，杜绝（MUST NOT）使用key或k
+ 值的属性名必须（MUST）为value，杜绝（MUST NOT）使用v。


###### 数据场景：键/值对象

{% highlight JSON %}
{
  "name": "BMW",
  "value": 1
}
{% endhighlight %}

#### 键/值有序集合

键/值有序集合表示对事务或逻辑类型的抽象与分类。常见的应用场景有单选复选框集合，下拉菜单等。

标准的键/值有序集合是一个JSON Array，集合中的每一项是一个JSON Object。项必须（MUST）包含name和value属性。可以（MAY）通过其他的属性修饰每一项的特殊信息，如selected。


###### 数据场景：键/值有序集合

{% highlight JSON %}
[
  {
    "name": "BMW",
    "value": 1
  },
  {
    "name": "Benz",
    "value": 2,
    "selected": true
  }
]
{% endhighlight %}

#### 树

树形数据用于表示层叠的数据结构。树型数据必须（MUST）是一个JSON Object，代表树型数据的根节点。下面是标准定义的可选节点列表，不在列表中的属性可以（SHOULD）自行扩展。


#### 树型数据结构的可选节点属性

| 属性名 |  类型   |    描述      |
|:-------:|:------- | :------------|
| id | JSON Number 或 JSON String | 节点的唯一标识。|
| text | JSON String | 名称或用于显示的字符串。|
| children | JSON Array | 子节点列表。|
{: .neat }

###### 数据场景：树型数据

{% highlight JSON %}
{
  "id": 1,
  "text": "中国",
  "children": [
    {
      "id": 10,
      "text": "成都",
      "children": [
        {
          "id": 100,
          "text": "成华区"
        },
        {
          "id": 101,
          "text": "金牛区"
        },
        {
          "id": 102,
          "text": "高新区"
        }
        ......
      ]
    },
    {
        "id": 31,
        "text": "海南",
        "children": [
          {
            "id": 600,
            "text": "海口"
          },
          {
            "id": 601,
            "text": "三亚"
          },
          {
            "id": 602,
            "text": "五指山"
          }
          ......
        ]
    }
    ......
  ]
}
{% endhighlight %}

### 引用

+ [RFC 2119: Key words for use in RFCs to Indicate Requirement Levels](http://www.rfcreader.com/#rfc2119)
+ [RFC 4627: The application/json Media Type for JavaScript Object Notation (JSON)](http://www.rfcreader.com/#rfc4627)
+ [RFC 2616: Hypertext Transfer Protocol](http://www.rfcreader.com/#rfc2616)
+ {: id='rfc3339' }[RFC 3339: Date and Time on the Internet: Timestamps](http://www.rfcreader.com/#rfc3339)
