---
layout: post
title: "Ideal React Redux Project Stucture"
description: "Ideal React Redux Project Stucture"
category: ['React', 'Redux', 'Project Structure']
tags: ['React', 'Redux', 'Project Structure']
---
{% include JB/setup %}

最近一年，在公司的很多项目上都采用了React/Redux技术栈，对于React或是Redux本身没有多大的问题，但是有一个问题一直困扰着我，那就是**如何组织项目的代码结构**。

最近是工作在一个全新的前端项目上，前端仍然是React + Redux，与后端通过API Service交互，也就是说Routes, View, States以及业务逻辑等等全部放在前端处理的。

对于项目目录结构，有一些基本要求是必须达到的：

- 对于任何大小的项目必须保持可扩展性和灵活性
- 很容易知道哪个文件应该放在哪个目录里面
- 代码重用很容易

下面是我所采用的一个项目结构：

```JavaScript
- api
- build
- scripts
- i18n
  - en_US.json
  _ zh_CN.json
  ...
- src
  - i18n
  - routes
    - Home
      - components
      - containers
      - modules
        index.js
    - Signin
      ...
    - Signup
      ...
  - shared
    - components
    - constans
    - containers
    - middlewares
    - modules
    - util
  - styels
  - tempest.js
- test
```

## api

**api**目录是一个基于`express.js`搭建的一个本地API Mock server，可以通过`npm run api`跑起来，默认本地开发调试时，所有的API请求都会导向它。另外，我们也加入了对[faker.js](https://github.com/marak/Faker.js/)的支持，帮助我们生成大量 的模拟数据，尽量接近用户真实环境。

## build

**build**目录包含构建脚本和开发环境的基础配置。我们是使用[Webpack](http://webpack.js.org/)来作为**module bundler**的。

## scripts

**scripts**目录主要包含部署和一些工具脚本，如编译打包**i18n**文件。

## i18n

**i18n**目录包含编译后的**i18n**文件，是以`JSON`格式存在的，每个`JSON`文件中的国际化文本消息是以键值对的形式存在的。

## src

**src**目录就是包含所有项目源代码的目录。

### i18n

**i18n**目录包含编译前的**i18n**源文件，是以`.properties`格式存在的。

### routes

**routes**包含项目中每个**route**的定义。

一个**route**目录包含：

  * 必须包含一个入口文件`index.js`，由它返回与`route`相对应的Component
  * **components**, **containers**, **modules(redux)**, **styles**等等目录
  * 嵌套的**child routes**，如 `events -> events/:id`

从上面看出，每个`route`是由`container`和`components`来构建视图，而数据和业务逻辑则交给**reducers**, **actions**和**selectors**，另外，我们采用了[Ducks: Redux Reducer Bundles](https://github.com/erikras/ducks-modular-redux)模式，我们会把这三块放在**modules**目录中。

#### index.js

每个`route`必须包含一个入口文件`index.js`，它主要用于加载与`route`相对应的Component。我们希望可以把整个大的bundle拆分成可以在之后异步下载的 chunk，每个chunk对应一个`route`。例如，这允许首先提供最低限度的引导 bundle，并在稍后再异步地加载其他功能。Webpack提供了相对应的支持。

Webpack 把 [`import()`](https://github.com/tc39/proposal-dynamic-import) 作为一个代码分离点(code split-point)，并把引入的模块作为一个单独的 chunk。`import()`将模块名字作为参数并返回一个 Promoise 对象，即 `import(name) -> Promise`。

```JavaScript
import asyncComponent from 'tempest.js/components/asyncComponent'

import './index.less'

const Signin = asyncComponent(
  () => import('./containers/SigninContainer').then(module => module.default),
  { name: 'Signin' },
)

export default Signin
```

> 从Webpack@2.4.0开始，它引入一个叫[**magic comments**](https://webpack.js.org/api/module-methods/#import-)的新Feature，它让你可以为每个异步Chunk命名，大大简化了资源的管理和SSR。

### shared

**shared**目录主要包含公共的**components**, **containers**, **modules(redux)**，**middlewares**，以及一些工具函数。

可能大家会很奇怪这里为什么会有这些公共的目录呢？不是应该放在对应的**route**目录下面吗？

这是因为我们发现，一个`route`常常会跟许多**domain data**打交道，比如说，一个events列表页面，不仅需要`events`，还需要`user`和`filter`数据。从另外一个方面来讲，一个**domain data**也可以同时被多个`routes`共享，而它本身可能并不是一个`route`。所以我们会这些公共部分提取出来，达到代码重用的目的。

### styles

**styles**包含整个项目的公用样式结构，如`Layout`, `variables`, `mixins`。每个组件对应的样式都是放在相对就的组件目录的。

### tempest.js

`tempest.js`是我们内部基于我们的最佳实践创建一个基于React + Redux的框架。它默认提供了一套**API**，从而帮助我们更高效便捷的创建Redux App，并最大程度的达到代码复用的目的。

### index.js

这是我们整个项目的入口文件`index.js`，它主要用于初始化Redux App，并启动整个Web App。

```
import React from 'react'
import { AppContainer } from 'react-hot-loader'

import 'active.css/less/index.less'

import ARCL10nProvider from 'shared/containers/ARCL10nProvider'
import Layout from 'shared/components/Layout'
import createApp from 'tempest.js'
import handlersMiddleware from 'shared/middlewares/handlersMiddleware'

import createRouter from './routes'
import userReducer from './shared/modules/user'

import './styles/index.less'

function render(Component) {
  createApp()
    .reducer({ user: userReducer })
    .middleware([handlersMiddleware])
    .view(store => (
      <AppContainer>
        <ARCL10nProvider>
          <Layout>
            <Component store={store} />
          </Layout>
        </ARCL10nProvider>
      </AppContainer>
    ))
    .run('#root')
}

render(createRouter)

if (module.hot) {
  module.hot.accept('./routes', () => {
    render(createRouter)
  })
}

```

## 总结

目前来说这种结构对于我们来说是一种最好的解决方案，但是这种结构并不能满足所有的情况（比如SSR），但是它已经足够灵活，同时支持水平扩展，新增功能并不会侵入已有的逻辑。
