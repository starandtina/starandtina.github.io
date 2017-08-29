---
layout: post
title: "front end engineering 4 active"
description: ""
category: 
tags: []
---
{% include JB/setup %}

在活跃网络（成都）呆了三年，搬过砖，架过构，管过人，想聊聊这些年的前端工程化实践之路。
{: .countheads }

* ToC
{:toc}

在2014年之前，活跃网络（ACTIVE Network）的前端部分基本可以分为二种类型的：

- 以jQuery为基础的单体Web App，后端技术包括但不限于：Java, .NET, PHP, RoR, etc.
- 以Backbone.js + RequireJS为基础构建基于前端MVC的SPA

很明显，第一种方案中，是以后端为主的MVC，前端开发重度依赖开发环境并且前后端职责始终会纠缠不清，很多时候前后端的代码会严重耦合。对于业务场景很复杂的系统，前后端的代码经常会混杂在一起，长此以往，将会完全失去维护性。

第二种方案中，就是随着AJAX的兴起而带来的SPA时代。前端负责View+Route+Template+AJAX，后端只负责提供数据接口，从此前后端职责更加清晰，前端可以自由选择自己的框架和库，也可以合理的进行分层，让各层各司其职。

但是遇到的挑战也是很明显的，主要有以下几个方面：

- 前端没有统一的开发工作流
- 前端没有统一的Style Guide和Code Conventions
- 前端没有测试（单元测试或是集成测试）来保证质量，基本靠自测和QA
- 前端技术栈不统一，从而就无法达到最大程度的利用，也就无从减少技术成本，提升产品的迭代效率以及产品体验

### Git Workflow

#### Feature Branch Workflow

为了提高开发效率，形成规范化的代码管理，团队开始了从SVN到Git的迁移，我们采用的是[Feature Branch Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows#feature-branch-workflow)，也就是说所有新功能的开发都是基于一个独立的分支，而不是传统的master分支。这样每个工程师可以工作在自己的feature分支上，而不会影响到主分支，主分支可以随时发布上线，当然这得借助于自动化的CI/CD。

### Git Commit Message Format

提供代码到分支上时，都会涉及到这个永恒不变的话题，给自己的commit加一段简短的描述，那么我们如何写一个Git Commit Message呢？请参考[How to Write a Git Commit Message](https://chris.beams.io/posts/git-commit/)

TLDR:

- Separate subject from body with a blank line
- Limit the subject line to 50 characters
- Capitalize the subject line
- Do not end the subject line with a period
- Use the imperative mood in the subject line
- Wrap the body at 72 characters
- Use the body to explain what and why vs. how

团队里面主要参考的是[Udacity Git Commit Message Style Guide](https://udacity.github.io/git-styleguide/)和[AngluarJS Commit Message Format](https://github.com/angular/angular.js/blob/master/CONTRIBUTING.md#commit-message-format)，基于满足上面这七大原则，同时，二者大同小异，但目标都是一致的，都是为了工作在Git环境并且与他人协作更方便一些。

> 推荐使用[commitizen](https://github.com/commitizen/cz-cli)来帮助团队规范化整个流程

### Architecture

在做前端技术选型架构时，我们充分考虑各个框架库的优缺点，选择适合我们项目需求，目的很简单，能够最大限度的帮助我们减少技术成本，提高迭代效率和用户体验。

- Build the SPA Web App
  - A single HTML page
  - No reload between page navigation
  - Business logic is run on the browser side using JavaScript
  - Update the page as user interact with it
- Routing with [On Demand Loading](https://webpack.js.org/guides/code-splitting/#dynamic-imports)
- Offline support with [Service Worker](https://github.com/GoogleChrome/sw-precache) & fethcing data from network

> There's no Web Server, the only communication bridge between front-end and back-end is through Web API. Will consider adding one more layer between front-end and back-end if we need to support SSR(server-side rendering, state persistence, etc.)

- Use [React <sub>View Layer</sub>](https://github.com/facebook/react) to build the component based rich user interfaces. React is a componentized design for web pages. Very popular among front-end developers, supported and proved by Facebook
  - Composition
  - Unidirectional Dataflow
  - Declarative
  - Explicit Mutations
  - Just JavaScript

- Use [react-router@v4 <sub>Declarative routing for React</sub>](https://github.com/ReactTraining/react-router) as the client-side routing solution for navigation between pages without refresh the whole pages

- Use [Redux <sub>Predictable State Container</sub>](http://redux.js.org/) to structure and manage more complex Web App state, it helps us solve:
  - Manage states: domain data, UI state, App state, etc.
  - Where do I keep all the data regarding my application along its lifetime?
  - How do I handle modification of such data?
  - How do I propagate modifications to all parts of my application?

- Use Redux Promise Middleware to handle side effects or asynchronous actions in the Redux Web App
  - It's a lightweight library for resolving and rejecting promises with conditional optimistic updates.
  - It enables robust handling of async code in Redux. The middleware enables optimistic updates and dispatches pending, fulfilled and rejected actions. It can be combined with redux-thunk to chain async actions.

- Use [isomorphic-fetch](https://github.com/matthew-andrews/isomorphic-fetch) as the Web API util to handle AJAX requests & responses

- Use [Immutable](https://facebook.github.io/immutable-js/) as the state storage library. It provides an easy to reason about data storage mechanism. Also from Facebook, it's a perfect companion to React.

- Use [react-aaui](http://fee) as the base UI components library which is built on top of React. With this, we can easily reuse existing components described in the Active Product Style Guide.

- Use [NPM](https://www.npmjs.com/) as the package management tool

- Use [Jest](https://facebook.github.io/jest/) [Enzyme](https://github.com/airbnb/enzyme) as the unit testing solution

### 开发流程（Development Workflow）

### Project Structure && Style Guides && Code Convention

#### Project Structure

请参考[理想中的React+Redux项目结构](http://starandtina.github.io/2017-06-04/ideal-react-redux-project-stucture)

### Style Guides && Code Convention

Style Guides是Code Convention的其中一种形式，它针对的是单个文件的layout，而Code Convention更加强调的是一些编程最佳实践，文件和目录的设计等等。

JavaScript是一门动态并且类型松散的语言，再加上团队成员水平参差不齐，急需要一种工具可以帮助我们发现代码中常见的bugs或是潜在的问题，提高代码质量，保持团队代码风格一致。因此我们引入了[ESLint](https://eslint.org/)作为代码的静态分析工具，并自动集成到我们的CI/CD流程中，也就是一旦发现ESLint错误，那么整个CI/CD会失败，直到修复这些错误为止。

另外，ESLint支持创建[ESLint Shareable Configs](https://eslint.org/docs/developer-guide/shareable-configs)，团队内部基于[eslint-airbnb](https://github.com/airbnb/javascript/tree/master/packages/eslint-config-airbnb)创建了适应自己的ESLint Shareable Config。

### Testing

### CI/CD 
