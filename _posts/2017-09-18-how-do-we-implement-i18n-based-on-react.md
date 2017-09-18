---
layout: post
title: "我们如何基于React实现i18n"
description: "我们如何基于React实现i18n"
category: ['React']
tags: ['React', 'Context', 'Observer']
---
{% include JB/setup %}

简单来说，我们需要提供一套API，根据当前**locale**而显示不同的文本消息，日期时间，货币等等。
{: .countheads }

* ToC
{:toc}

## 目标

- 根据当前locale**异步加载**i18n资源，包括文本，日期配置，货币配置等等
- locale一旦改变，相应的文本，日期，货币会立即相应变化，**无论React组件树层级多深**
- 可以独立于React组件使用
- 一套简单的声明式的API优于命令式的API

由于我们的需求，以及下面的种种原因，最终导致我们没有选择[react-intl](https://github.com/yahoo/react-intl)作为我们的i18n解决方案。

- [Safely use context react-intl#901](https://github.com/yahoo/react-intl/pull/901)
- [Children don't update when context changes react-intl#660](https://github.com/yahoo/react-intl/issues/660)
- [How to implement shouldComponentUpdate with this.context? react#2517](https://github.com/facebook/react/issues/2517)
- [My views aren’t updating when something changes outside of Redux](https://github.com/reactjs/react-redux/blob/93cdfaeaf9d3e5400ffc05fe9d177118286109ca/docs/troubleshooting.md#my-views-arent-updating-when-something-changes-outside-of-redux)

## 实现

### 异步加载i18n资源

我们是用[Webpack](https://webpack.js.org)来作为打包工具的，Webpack天生就支持代码分割，按需异步加载代码块，所以无论是Webpack@1中的[require-ensure](https://webpack.js.org/api/module-methods/#require-ensure)，还是其后续者Webpack@2中的[import()语法](https://webpack.js.org/guides/code-splitting/#dynamic-imports)，都可以帮助我们做到*根据当前locale**异步加载**i18n资源*。

默认选择的是**import()**，首先[它是符合ECMAScript规范的](https://github.com/tc39/proposal-dynamic-import)，另外，从Webpack@2.4起，Webpack也支持通过`webpackChunkName`来自定义每个区块名，已经可以完全取代`require.ensure`。

### React@`Context`

首先我们是基于React组件模型的，它提供了很好的一套父子组件的通信方式。父组件可以通过`props`传递数据给子组件，而子组件可以通过调用*callback prop*来回传数据，这样一来，数据流就很清晰，代码也很容易维护。出现问题时，也很容易找出问题所在，因为你知道是谁传递的这个callback，是谁调用了这个callback，是哪二个父子组件在通信。所以，第一个想法是，我们可以通过这样单向传递的方式来逐级传递i18n资源的，但是，随着组件层级加深，问题也随之而来，每个组件不得不显示的将`props`传递给它的子组件（即使有可能没有用到它），更槽糕的是，有些中间层组件，根本不在我们的控制之内，有可能是第三方的组件库。

而[React Context](https://facebook.github.io/react/docs/context.html)完美的解决了这个问题。

> React Context: 当一个组件在它的**context**上定义了一些数据后，任何它的后代子组件都可以访问这些数据。

这也就意味着，任何在组件树中的子组件都可以访问从当前的Context中访问数据，而并不需要通过`prop`来传递。具体使用方法，请参考[官方文档](https://facebook.github.io/react/docs/context.html#how-to-use-context)。

### React@`shouldComponentUpdate` -> Observer Pattern

`Context`是什么时候会更新呢？[官方文档是这样写的](https://facebook.github.io/react/docs/context.html#updating-context)：

> The getChildContext function will be called when the state or props changes. In order to update data in the context, trigger a local state update with this.setState. This will trigger a new context and changes will be received by the children.

但是`Context`更新了之后，在组件树中的子组件能够实时的获取最新的值吗？这个就取决于中间层组件的`shouldComponentUpdate`实现。任何一个React组件都可以定义自己的`shouldComponentUpdate`实现（或者继承自`PureComponent`），如果它返回`false`，那就表明它不需要重新渲染。但是，如果某一个中间层组件也返回`false`，那么它的子组件也就不会更新，即使是当前组件树中的`context`已经变了。

```
高层组件（包含context.color定义）

    中间层组件（`shouldComponentUpdate`返回`false`）
        ...
            ....
                子组件（从当前context中获取数据）

```
以上图为例，如果`context.color`更新了，子组件是不会重新渲染的，因为它的父级组件的`shouldComponentUpdate`返回了`false`。这样直接导致的结果就是 - 全乱套了，UI和状态之间无法同步，从而导致各种莫名其妙的Bugs，这也是大家常常不愿意用它的一个重要原因。

那么，怎么解决呢？答案就是`Context` + **Observer Pattern**。首先，我们会有二个假设，

- `Context`是immutable的，由它自己维护自己的内部状态
- 组件只需要接受一次`Context`，之后`Context`的更新都能[通过订阅获取]((https://addyosmani.com/resources/essentialjsdesignpatterns/book/#observerpatternjavascript))

### API设计

#### `L10n`

它主要用于：

- 实现观察者模式，维护观察者列表，
- 当locale改变时，负责通知所有观察者（如`L10nMessage`，`L10nDateTime`，`L10nCurrency`等等）
- 提供格式化方法，如`formatMessage`，`formatDateTime`，`formatCurrency`等等

#### `L10nProvider`

它主要用于：

- 根据当前locale**异步加载**i18n资源
- 封装`L10n`对象，通过`Context`暴露给子级组件
- 通常来说，它会作为整个应用最顶层组件使用，其它`L10nXXX`组件不能脱离它独立使用

#### `injectL10n`

它主要用于：

- 根据当前`Context`中的`l10n`对象，生成一个新的l10n对象，然后以*prop*的形式注入到绑定组件中

这样就保证在locale或是其它资源（如`messages`）变化后，被注入的组件会有`props`变化，从而会触发重新渲染，重新获取`l10n`中的值

```
getBoundL10n = () => {
  const l10n = this.context...
  const boundFuncs = ...
  const boundConfig = ...
  return {
    [l10nName]: {
      ...l10n,
      ...boundConfig,
      ...boundFuncs,
    },
  }
}
```

#### `L10nMessage`

`L10nMessage`是`l10n.formatMessage`方法的组件封装，它根据当前传入的`id`，找到对应的文本信息，同时也支持字符串插值。

![L10nMessage.js](/assets/images/L10nMessage.js.png)
{: class='image-wrapper'}

#### `L10nDateTime`

`L10nDateTime`是`l10n.formatDateTime`的组件封装，它根据当前传入的`date`对象和日期时间的`format`，转换回对应`format`的字符串形式。

比如，`en_US`的长日期时间格式为`MMMM d, yyyy h:mm a`，然后传入当前的date后，就会转换为`2017/09/31 11:30 a.m.`

#### `L10nCurrency`

`L10nCurrency`是`l10n.formatCurrency`的组件封装，它根据传入的数值`amount`和货币代码（如`USD`）,转换回对应格式的字符串表示形式。同时，用户可以指定以下选项：
componentDidMount
```
{
  integerOnly: false,
  separationCount: 3,
  separator: ',',
  negativeMark: '-',
}
```

### 数据流

有了这些组件API的设计，那么它们之间是怎么协同工作的呢？

```
<L10nProvider locale='zh_CN'>
  ...
    <UpdateBlocker>
        <L10nMessage id='save' />
        ...
    </UpdateBlocker>
  <Form>
    <Field
      placeholder={l10n.formatMessage('save')}
    />
  </Form>      
</L10nProvider>
```

- `L10nProvider`中会初始化一个`L10n`对象
- `DateTimeSymbols`，`currenciesConfig`和`en_US`的i18n文本消息，会默认绑定到上一步生成的`L10n`对象上
- 在`L10nProvider`的`componentDidMount`生命周期方法中，会去动态加载指定的`locale`的文本消息（此处是`zh_CN`）
  - 加载成功后，会与当前`this.l10n.messages`合并
  - 通知所有订阅了改变的子组件，如`L10nMessage`
- 子组件收到更新通知后，会从当前`Context`中，拿到immutable的`L10n`对象后，重新生成一个新的`L10n`对象字面量，然后以*prop*的形式注入到绑定组件中
- 最后，子组件根据最新的`locale`，`messages`格式化



