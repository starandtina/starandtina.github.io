---
layout: post
title: "Redux的简单性"
description: "Redux的简单性"
category: ['Redux']
tags: ['Redux']
---
{% include JB/setup %}

最近关于Redux是什么以及如何使用Redux的讨论很多，过去一年半我们也在许多真实的项目中亲身实践了利用React + Redux来构建复杂的SPA Web App，想谈谈自己的观点。
{: .countheads }

* ToC
{:toc}

### Redux是什么

Redux最初的源代码只有99行，但是它的核心却实现了一个相当简单的**发布订阅模式**。Redux会保存当前的状态(Store)，然后在需要的时候（Dispatching）调用一个特定的**Pure**函数(Reducer)来更新该状态，并且在状态改变通知订阅者。That's it!

### Redux ~= MVC

在Redux中，我们也可以找到与[经典的MVC](http://starandtina.github.io/2015-09-06/backbone-mvc-style-guide#real-mvc)中相似的一些概念。

> 简单来说，经典的MVC架构是这样工作的。首先我们会有一个Model，它往往是整个MVC架构的中心。如果这个Model有更新了，它会通知它的观察者们某个改变发生了。然后是View，它就是我们常常能看到的UI，并且View一般会监听着Model。当View收到Model发来的更新通知时，View会更新它的外观来反映Model的更新。作为用户的我们通常是与View交互（如点击，输入事件等等），但是呢，View并不知道如何处理用户的交互操作。所以呢，View会告诉Controller用户刚刚做了什么并且假设Controller知道如果处理用户的操作。Controller这时候会更新Model，之后不会一次一次又一次一次的重复重复再重复。另外，所有通信都是单向的。

- Action = Controller

Action主要描述了**发生了什么**（What happened）。实际开发中，我们可以把Actions当成Controller来对待的。无论何时，只要你想触发一个修改（如响应DOM事件，异步加载数据，设置Dropdown组件打开状态等等），那么你就可以派发一个Action。这就像在MVC中，你也会交给Controller来处理接下来的事情一样。（Controller可能会去更新相应的Model，从而触发重新渲染）

- Reducer ~= Model

Reducer主要描述了**如何处理这些改变** (How things changed)。也就是说，Reducers会负责管理Web App当前的状态（比如，用户登录信息，Domain数据，UI状态，App状态等等）。当我们派发一个Action时，也就是交给Reducer来决定如何更新当前的状态。与经典的MVC相比，我们是交给Model来更新相应的状态的，比如常常会看到Model中会有很多叫`setXXX`的方法，而在Redux中，也会有一个相应的Reduer来响应这些`setXXX`，从而更新相应的状态值。

- Store

Store主要存储了当前Web App的状态。Store是Redux独有的一个概念，在经典的MVC中，没有一个与Store相对应的。Redux中的Store更像是一个聚集了所有Slice Reducers的状态容器。Store提供了一个叫`getState`的API用于获取当前的状态，同时，也暴露了订阅状态更新的接口（**react-redux**中`connect()`正是利用了这个接口来监听状态改变从而更新组件的`props`）。Store也会接受所有派发的Actions，同时它会将这些Actions传递给所有的Reducers（也许是通过`combineReducers`），从而更新状态。Store也支持Middlewares（中间件）函数，从而让在调用真正的Reducers之前实现自定义逻辑（派发Thunks或是Promises），中间件函数一般是用于处理异步数据获取，日志等等功能。

### Redux没有提供的

Redux本身是一套用于管理状态的简单机制，核心是想让状态的改变是可预测的。可能也正是由于Redux看起来是如此的简单，从而才出现了围绕*如何使用Redux*的各种各样方法，观点，以及最佳实践。

正是由于Redux的*简单*，当我们需要构建一个大型的Web App时，而它却没有提供：

* Immutability
* Pure functions
* Middlewares
* Normalization
* Selectors
* Thunks
* Sagas
* 异步逻辑应该放在哪里？
* 数据是否应该范式化（normalized）或是继续保持嵌套？
* Store中可以保存不可序列化的对象吗？如Promises或是对象实例？
* 是否应该使用Action Creatores来创建我们需要的Actions呢？
* 是否Action Type应该是`String`或是`Symbol`类型呢？是否应该使用常量来定义Action Type呢？或是直接写就好了？
- 为什么添加一个简单的获取列表数据功能，我们需要添加这么多文件（Constants, Actions, Reducers, Components, Containers等等）呢？修改时也需要在这么多文件之间跳转呢
- Redux为什么没有内置解决异步请求？这么多的异步实现方案（如Redux Thunk, Redux Promise, Redux Saga, Redux Observable等等），应该选择哪一个』
- 业务逻辑到底应该放在哪里呢？是在Redux Action Creators吗？还是在Reducers里？或是在Saga中？或是直接放在组件中
- 我应该如何组织我的项目目录结构？每个文件应该放在哪个目录下
-  ...

### Redux设计哲学

[Redux Philosophy & Design Goals](https://github.com/reactjs/redux/blob/07cf1424cb5e7bb7284acb282c996690cfc6d8c5/README.md)这是最初2015年6月Redux刚问世时，它所宣扬的设计哲学，到现在看起来时，仍然是那么的激动人心！！!

* You shouldn't need a book on functional programming to use Redux.
* Everything (Stores, Action Creators, configuration) is hot reloadable.
* Preserves the benefits of Flux, but adds other nice properties thanks to its functional nature.
* Prevents some of the anti-patterns common in Flux code.
* Works great in isomoprhic apps because it doesn't use singletons and the data can be rehydrated.
* Doesn't care how you store your data: you may use JS objects, arrays, ImmutableJS, etc.
* Under the hood, it keeps all yout data in a tree, but you don't need to think about it.
* Lets you efficiently subscribe to finer-grained updates than individual Stores.
* Provides hooks for powerful devtools (e.g. time travel, record/replay) to be implementable without user buy-in.
* Provides extension points so it's easy to [support promises](https://github.com/gaearon/redux/issues/99#issuecomment-112212639) or [generate constants](https://gist.github.com/skevy/8a4ffc3cfdaf5fd68739) outside the core.
* No wrapper calls in your stores and actions. Your stuff is your stuff.
* It's super easy to test things in isolation without mocks.
* You can use “flat” Stores, or [compose and reuse Stores](https://gist.github.com/gaearon/d77ca812015c0356654f) just like you compose Components.
* The API surface area is minimal.
* Have I mentioned hot reloading yet?

### 总结

Redux内部的工作原理简单，但是根据自己项目的需求，如何正确的使用Redux才是真正的难题！
