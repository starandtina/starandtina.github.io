---
layout: post
title: "Important Notes on Structuring Your App State"
description: "Important Notes on Structuring Your App State"
category: ['Front-End', 'State', 'dev']
tags: ['Front-End', 'Redux', 'Flux', 'State']
---
{% include JB/setup %}

Redux的出现，让我们更加去关注我们应用的状态并且花更多的时间在上面，因为它便不是一个简单容易的工作。一点点状态的改动可能就会引起蝴蝶效应。对于如何去管理我们的应用状态以便让你的业务逻辑更健全更简单，下面提供了一些实用的Tips。
{: .countheads }

* ToC
{:toc}

## 避免按照服务器端的API设计状态

服务器端的API设计会有各种不同的考虑，这些与你的应用的状态并不一定吻合，一般情况下，服务端返回的数据与我们的应用状态都是无关的，而且我们也不应该依赖于服务端返回的数据结构来定义我们的应用状态。

## 尽量使用`Map`，少用`Array`

一般来说，数组是不利于管理状态的。考虑其中一种情况，如果你需要获取或是更新一个具体产品数据项时，会发生什么事情？比如说，如果当前的应用是用于更新该产品的价格，又或是我们需要重新从服务器端获取数据。为了找到某一个特定的产品，比起直接从一个对象中根据它的`ID`直接获取该产品，而去迭代一个大数组是相当不方便的。

所以，我们的推荐方案是什么呢？**使用对象并且用它的主键作为索引来查找吧**。

{% highlight JavaScript %}

{
  "products": {
    "88cd7621-d3e1-42b7-b2b8-8ca82cdac2f0": {
      "title": "Blue Shirt",
      "price": 9.99
    },
    "aec17a8e-4793-4687-9be4-02a6cf305590": {
      "title": "Red Hat",
      "price": 7.99
    }
  },
  "productIds": [
    "88cd7621-d3e1-42b7-b2b8-8ca82cdac2f0",
    "aec17a8e-4793-4687-9be4-02a6cf305590"
  ]
}

{% endhighlight %}    

## 避免按照*View*来设计状态

应用状态的最终目的都是为了*View*服务，最终都是通过*View*体现出来的。一般来说，很难忍住不去按照*View*来设计你的应用状态的。
而且，不同的*View*会以不同的方式来呈现你的状态，基本上不太可能满足所有情况而又不会有重复数据。

{% highlight JavaScript %}

{
  "products": {
    "88cd7621-d3e1-42b7-b2b8-8ca82cdac2f0": {
      "title": "Blue Shirt",
      "price": 9.99
    },
    "aec17a8e-4793-4687-9be4-02a6cf305590": {
      "title": "Red Hat",
      "price": 7.99
    }
  },
  "outOfStockProductMap": {
    "aec17a8e-4793-4687-9be4-02a6cf305590": true
  }
}

{% endhighlight %} 

## 千万不要在应用状态中有冗余数据

如何知道你的状态是否有冗余数据呢？一个快速测试的方法是，检测是否你为了保持数据的一致性而需要同时更新二个或多个地方。
处理冗余数据也很简单，你所需要做的就是删除其中一个就行了。如果我们只在一个地方存储数据，那么就不太可能出现状态的不一致了。

以上面的代码为例，如果某个产品没有库存了，我们需要更新`outOfStockProductMap`就行了。而不需要去更新每个具体的产品。

## 千万不要在应用状态中存储根据当前过滤条件而衍生的数据

任何存储在应用状态中的导出数据都违反了一个基本原则，因为为了维持数据的一致性，那么必然的结果是所有的更新会发生在各处不同的地方。

还是以上面的代码为例子。如果我们新增一个需求，需要添加一个打折信息在每件产品上。那么当前的应用就需要要么给用户显示所有产品，或是没有打折信息的所有产品，又或是只有打折信息的所有产品。

一个常见的错误是，我们会分别维护三个数组在状态中，每一个包含相应过滤条件的产品*ID*。既然这三个数组可以从当前的过滤条件和产品对象中衍生出来，一个更好的做法是动态生成它们：

{% highlight JavaScript %}

{
  "products": {
    "88cd7621-d3e1-42b7-b2b8-8ca82cdac2f0": {
      "title": "Blue Shirt",
      "price": 9.99,
      "discount": 1.99
    },
    "aec17a8e-4793-4687-9be4-02a6cf305590": {
      "title": "Red Hat",
      "price": 7.99,
      "discount": 0
    }
  }
}

// selector
function filteredProductIds(state, filter) {
  return _.keys(_.pickBy(state.productsById, (product) => {
    if (filter == "ALL_PRODUCTS") return true;
    if (filter == "NO_DISCOUNTS" && product.discount == 0) return true;
    if (filter == "ONLY_DISCOUNTS" && product.discount > 0) return true;
    return false;
  }));  
}

{% endhighlight %} 

`Selector`函数在每次状态改变时都会执行，从而计算出衍生数据以供*View*渲染。

## 范式化嵌套对象

一切的努力都是尽量简化对象层级和结构。我们需要一直维护我们的应用状态，我们需要让这一过程尽量保持简单。如果你的数据对象之间是毫不相关的，那么简易性是很容易维持的。但是，如果应用状态内部是纵横交错的，那么会发生什么情况呢？

还是以上面的代码为例。考虑更复杂一点的情况，我们想要添加一个订单管理系统，在这个系统里，用户可以在一个订单中一次性买多个产品。我们假设服务器端**API**返回的**JSON**数据为：

{% highlight JavaScript %}

{
  "total": 1,
  "offset": 0,
  "orders": [
    {
      "id": "14e743f8-8fa5-4520-be62-4339551383b5",
      "customer": "John Smith",
      "products": [
        {
          "id": "88cd7621-d3e1-42b7-b2b8-8ca82cdac2f0",
          "title": "Blue Shirt",
          "price": 9.99,
          "giftWrap": true,
          "notes": "It's a gift, please remove price tag"
        }
      ],
      "totalPrice": 9.99
    }
  ]
}

{% endhighlight %} 

为了做分页，有时也会加上`total`和`offset`信息。一个订单可能包含多个产品，所以它们之间肯定有某种内部联系。从第一条Tip里我们知道了，我们就不应该按照服务器端返回的数据结构来设计我们的应用状态，因为它有可能会引起数据冗余。

一个更好的做法是范式化你的数据并且维护二个对象（一个用于存储产品，另外一个用于存储订单）。既然这些对象都是基于唯一的*ID*的，那么我们可以用这个*ID*来表明它们之间内部的关系。

{% highlight JavaScript %}

{
  "products": {
    "88cd7621-d3e1-42b7-b2b8-8ca82cdac2f0": {
      "title": "Blue Shirt",
      "price": 9.99
    },
    "aec17a8e-4793-4687-9be4-02a6cf305590": {
      "title": "Red Hat",
      "price": 7.99
    }
  },
  "orders": {
    "14e743f8-8fa5-4520-be62-4339551383b5": {
      "customer": "John Smith",
      "products": {
        "88cd7621-d3e1-42b7-b2b8-8ca82cdac2f0": {
          "giftWrap": true,
          "notes": "It's a gift, please remove price tag"
        }
      },
      "totalPrice": 9.99
    }
  }
}

{% endhighlight %} 

上面就是范式化后的结果，如果我们想要找到属于某个订单的所有产品，那么我们只需要迭代产品对象就可以了。每一个`key`就是产品的唯一*ID*。用这个唯一产品*ID*访问`products`对象时，就可以提供一个产品的所有细节了。另外，只与某个产品相关的一些特殊信息，如`giftWrap`，我们可以存储在订单对象下面的具体某个产品上的。

目前，我们可以借助[normalizr](https://github.com/paularmstrong/normalizr)来帮我们自动化完成这项工作，它可以接受一个**Schema**定义，然后帮你完成格式化对象任务。

## 应用状态可以被当成一个内存数据库

前面列举出的一些Tips都是很相像的。我们其实一直在做的就是DBA的工作，设计一个传统的数据库。

当设计一个传统的数据库时，我们需要避免冗余和衍生数据，使用对象结构的表加上主键（*ID*）来索引数据并且格式化多张表之间的关系。这就是到目前为止我们所讨论的所有东西。

把你的应用状态当成一个内存数据库一样对待，可以帮助你在构建更好的应用状态时提供更好的正确的决策。

## 像对待一等公民一样对待应用状态

如果你只想从这篇文章中记住一件事，那就是它了。

应用状态或是我们的数据结构，应该跟我们的代码一样重要，特别是像*React*这样的声明式编程范式，整个系统是根据当前的状态而变化的。状态就是一等公民，是跟我们写的代码同等重要。它是我们的*View*层的唯一来源。

在写代码前，我们需要花更多的时间在设计我们的应用状态上。我们应该时刻关注它的复杂度，以及我们需要花多少时间在维护状态上。并且，如果有任何信号告诉我们应该重构当前的应用状态时，我们就应该像重构我们的代码一样，Just do it!
