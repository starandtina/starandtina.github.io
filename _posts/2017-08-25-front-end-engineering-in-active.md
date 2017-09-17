---
layout: post
title: "Front-End Engineering in ACTIVE"
description: ""
category: 
tags: []
---
{% include JB/setup %}

在活跃网络（成都）呆了三年多，搬过砖，踩过雷，架过构，管过人，想聊聊这些年在活跃网络的前端工程化实践之路。
{: .countheads }

* ToC
{:toc}

### 前端架构历史

在2014年之前，活跃网络（ACTIVE Network）的前端技术架构基本可以分为二种类型：

- 以**jQuery**为基础的单体**Web App**，后端技术包括但不限于：Java, .NET, PHP, RoR, etc.
- 以**Backbone.js + RequireJS**为基础构建的基于MVC的**SPA**
- ...

很明显，在第一种方案中，是以后端为主的MVC，前端开发重度依赖后端的开发环境，并且前后端的职责纠缠不清（路由，模板等等），很多时候前后端的代码会严重耦合。而对于业务场景比较复杂的系统，前后端的代码经常会混杂在一起，长此以往，将无从谈起可维护性和可扩展性，后面功能的开发也只能是在以前代码的基础上修修补补，直到无法忍受的一天就会产生重构的想法。

第二种方案中，是随着AJAX的兴起而带来的前端SPA时代。前端负责**视图+路由+模板+AJAX**，后端只是负责提供数据接口（RESTFul API，WebSocket, etc.），从此前后端职责更加清晰，前端可以自由选择自己的框架，库，及至模块引擎，也可以进行合理的垂直分层，让各层各司其职，互相配合。

同时，遇到的挑战也是很明显的，主要有以下几个方面：

- 前端没有统一的*标准规范*，包括底层技术选型（库，框架，工具，测试等等），开发工作流程，风格指南，代码规范等等
- 前端底层技术栈不统一，无法达到最大程度的复用（组件，流程复用）
- 前端没有统一的自动化流程，包括CI/CD
- 前端没有测试（单元测试，集成测试或是自动化测试）来保证代码质量，基本是靠工程师自测，QA的手动测试外加部分自动化

所有这些都是我们目前所遇到的挑战和必须要解决的难题，最终前端的核心诉求都是一致的：

> 减少技术成本，提升产品迭代效率以及产品体验

## 前端标准规范

前端标准化我们要做的就是做好底层前端架构的选型，规范开发技术范围，达到最大程度的利用，从而减少我们的开发技术成本，进而提升产品的迭代效率。在这一方面我们做了很多工作。

### ACTIVE CSS

*Active.css*是活跃网络内部的CSS框架，它确保了整个活跃网络产品线可以保持一致的外观和行为，并且让我们可以很容易的去对整体做更新。它所提供的数十几种灵活实用且可重用的组件，可以帮助我们在之后的开发过程中进一步提高生产力，并且，它也让所有的开发者，设计师，产品经理，以及市场人员可以参考同一套标准，做到有据可依，有理可循。

在过去，每个产品团队会自己实现的一套CSS样式库，所以我们常常会遇到一些问题：

- 很难去维护和重用CSS样式
- 保持活跃网络所有产品线体验一致

但是，有了*Active.css*之后，我们获得如下的好处：

- 标准化

整个公司可以共用一套CSS样式库，以让我们可以维持一个更高标准和最佳实践

- 一致性

针对开发者，设计师，产品经理，以及市场人员等等，我们建立了一套公共语言，并且让它应用到不同的产品的同时，还保持一致性

- 高效

每个产品团队不用自己去开发和维护公共样式库，只需要重用这一套标准规范而已。开发人员可以花更多的时间在构建业务上，而不是花在考虑应该如何写CSS上

- 可维护性

是以组件为最小单元来构建每个CSS组件，通过组合进而构建更为复杂的组件或者业务单元

另外，为了维持整体的CSS风格统一，也推出了*CSS风格指南*，它会提供一些关于CSS的最佳实践，如文件目录结构组织，命名，样式规则等等。

![ACTIVE CSS Style Guidelines](/assets/images/active-css-guidelines.png)
{: class='image-wrapper'}

### ACTIVE UI

从2016年初开始，我们开始采用[React](https://facebook.github.io/immutable-js/)作为构建前端视图的基础库，在这过程中，逐渐把公用组件剥离出来，使用得这些组件可以在各个项目组中得到最大程度的复用。

组件主要包括以下几类：

- 独立于业务的单一组件：`Button`，`Dropdown`，`Table`，`Modal`，`DatePicer`，`Pagination`，`Input`等等
- 复合组件：`Form`，`CheckboxGroup`，`RadioGroup`，`SearchInput`，`Tabs`等等
- 业务组件（需要在多个项目中重用）：`AddressEditor`，`L10nProvider`
- 工具组件：`ShowAt`，`HideAt`等等

另外，每个单一组件相应的CSS定义都是放在*Active.css*中，也就是JS和CSS是分开单独维护开发的。为什么这样做呢？

活跃网络有很多存在了十几年的老项目，他们的实现方式多种多样，但是他们都有一个统一需求，那就是『改头换面』，重新换上新的样式风格，但是为了尽量减少工作量，我们一般只做样式上的修改，而不是做行为上的改动，我们称之为『Reskin』。

![ACTIVE UI](/assets/images/active-ui.png)
{: class='image-wrapper'}

### Git 工作流

#### 功能分支工作流

为了提高开发效率，形成规范化的代码管理，团队开始了从SVN到Git的迁移，内部采用的是[功能分支工作流](https://www.atlassian.com/git/tutorials/comparing-workflows#feature-branch-workflow)，也就是说所有新功能的开发都是基于一个独立的功能分支，而不是传统的master分支。这样每个工程师可以工作在自己的功能分支上做开发，而不会影响到主分支，主分支可以随时发布上线，当然这得借助于自动化的**CI/CD**来保证。

### Git 提交消息格式

当我们提交代码时，都会涉及到一个永恒不变的话题---*如何给`Commit`加一段简短且有意义的描述*

那么我们应该如何写一个**Git 提交消息**呢？请参考[How to Write a Git Commit Message](https://chris.beams.io/posts/git-commit/)

TLDR:

- 用一个空行来分隔开主题和正文
- 主题限制在50个字符内
- 主题行以一个大写字母开始
- 主题行不要以点结尾
- 主题行使用命令式语气
- 正文的每行限制在72个字符内
- 使用正文来解释what & why，而不是How

目前主要参考的是：

- [Udacity Git Commit Message Style Guide](https://udacity.github.io/git-styleguide/)
- [AngluarJS Commit Message Format](https://github.com/angular/angular.js/blob/master/CONTRIBUTING.md#commit-message-format)

二者大同小异，满足上面这七大原则，都可以帮助我们统一**Git 提交消息格式**，从而与他人协作更加方便。

> 推荐使用[commitizen](https://github.com/commitizen/cz-cli)来帮助团队规范化整个流程，与npm scripts配合自动化整个流程

### 前端技术选型

在做前端技术选型架构时，我们会充分考虑各个框架库的优缺点，选择更加贴合项目需求的成熟技术方案。目的很简单，希望能够最大限度的帮助我们减少技术成本，提高迭代效率，运行效率，以及用户体验。

#### Single Page App(SPA)

我们经常提到SPA，那么到底什么是SPA呢？

- 它是一个单一的**HTML**页面
- 它在页面之间跳转时，不需要重新刷新
- 它使用**JavaScript**来编写业务逻辑，并运行在浏览器端
- 在用户与页面交互时，它可以部分更新页面
- 路由：[On Demand Loading](https://webpack.js.org/guides/code-splitting/#dynamic-imports)
- 基于[Service Worker](https://github.com/GoogleChrome/sw-precache)提供离线支持

> 目前并没有使用**Node.js**来作为中间的**Web Server**层，前后端之间唯一的通信桥梁就是**Web API**。如果之后需要支持**SSR**（server-side rendering）或是**State Persistence**等等时，会考虑在前后端之间添加**Node.js**层。

#### React

使用[React <sub>视图层</sub>](https://github.com/facebook/react)来构建基于组件的富用户界面层。React是基于组件化的设计，与前端的技术趋势（模块化，组件化）非常接近，在前端社区中也非常流行，支持者众多，并且它是由Facebook官方支持和亲身实践的。下面是我们很喜欢的一些**React**的特性：
  
- 组合
- 单向数据流
- 声明式
- 显示变化（Explicit Mutations）
- React API简单，小而美
- 没有多余的概念，只是**JavaScript**
- 虚拟DOM
- 函数式编程
- SSR（服务端渲染）
- ...

#### Router

使用[react-router@v4 <sub>Declarative routing for React</sub>](https://github.com/ReactTraining/react-router)来作为客户端的路由解决方案，从而让用户可以在页面之间跳转而不需要重新刷新整个页面。从react-router@v4开始，它的基础代码全部重写，如今的Route更偏向于组件，它只是基于当前的URL是否与所设定的路径一致，从而决定显示或是隐藏组件，这也更加**React**，贴合组件化的思想。更多的请参考[All About React Router 4](https://css-tricks.com/react-router-4/)。

#### Redux

使用[Redux <sub>Predictable State Container</sub>](http://redux.js.org/) 来构造和管理复杂的**Web App**状态，它能帮助我们解决：

- 管理各种各样的状态：Domain数据，UI状态，App状态等等
- 在Web应用的整个生成周期中，在哪里保存我所有的数据呢？
- 我如何处理这些数据的修改呢？
- 我如何让我的整个Web应用感知到这些数据的变化呢？

#### Redux Promise Middleware

在整个Redux **Web App**中，统一使用团队内部构建的Redux Promise Middleware来处理所有的副作用（异步逻辑），比如RESTFul API。与redux-thunk，redux-saga或是redux-obserable相比，它更适合我们团队的需求，简单易上手，基本满足我们的需求：

- 它相当于一个轻量级的库，专门用于Resolve Promises和Reject Promises，并且它是完全基于乐观更新的
- 它让我们可以更加灵活的去处理Redux中的异步actions，它是基于乐观更新的，依据一定的命名规范会自动派发`${REQUEST}`, `${REQUEST}_SUCCESS`和`${REQUEST}_FAILURE`的actionts

#### Isomorphic Fetch

基于[isomorphic-fetch](https://github.com/matthew-andrews/isomorphic-fetch)封装了一套**api.js**，作为**Web API**工具来处理AJAX的请求和返回。

#### Immutable.js

使用[Immutable.js](https://facebook.github.io/immutable-js/)来作为状态存储库。它提供了很多常用的Persistent Immutable data structures来帮助我们管理数据存储，并且保证它们是Immutability的。这就要求我们操作所有数据时，都要当作不可变数据（immutable data）来对待，保证每次都是全新对象，没有引用关系，这样才能保证State的独立性，便于测试和追踪变化。同样的，它也是来自于Facebook，与React是天生一对。

#### React AUI

使用基于React构建的react-aui来作为基础的UI组件库，它包含了一系列满足我们的Product Style Guide的灵活，实用且可重用的组件。

#### Yarn

使用 [Yarn](https://yarnpkg.com/)来做为我们的包管理工具。

#### Jest

使用[Jest](https://facebook.github.io/jest/)和[Enzyme](https://github.com/airbnb/enzyme)来作为测试的解决方案。

#### Webpack

使用[Webpack](https://webpack.js.org/)来作为Module Bundler，它提供了丰富的Loaders和Plugins来帮助我们处理各种各样的资源，同时它还支持Tree Shaking和Code Splting，可以更好的帮助我们做组件化开发与资源的管理。

> 根据增量的原则，我们应该精心规划每个页面的资源加载策略，使得用户无论访问哪个页面都能按需加载页面所需资源，没访问过的无需加载，访问过的可以缓存复用，最终带来流畅的应用体验。

### 开发流程

### 项目目录结构

请参考[理想中的React+Redux项目结构](http://starandtina.github.io/2017-06-04/ideal-react-redux-project-stucture)一文。

### 风格指标和代码规范（ESLint + Prettier）

风格指南是代码规范的其中一种形式，它针对的是单个文件的布局，而代码风格更加强调的是一些编程最佳实践，文件和目录的设计等等。

**JavaScript**是一门动态并且类型松散的语言，再加上团队成员水平参差不齐，急需要一种工具可以帮助我们发现代码中常见的bugs或是潜在的问题，提高代码质量，保持团队代码风格一致。因此我们引入了[ESLint](https://eslint.org/)作为代码的静态分析工具，并自动集成到我们的CI/CD流程中，也就是一旦发现ESLint错误，那么整个CI/CD会失败，直到修复这些错误为止。

另外，ESLint支持创建[ESLint Shareable Configs](https://eslint.org/docs/developer-guide/shareable-configs)，团队内部基于[eslint-airbnb](https://github.com/airbnb/javascript/tree/master/packages/eslint-config-airbnb)创建了适应自己的ESLint Shareable Config。

最近，我们也集成了[Prettier](https://github.com/prettier/prettier)来作为代码格式化工具，之后，**Prettier**会主要负责代码风格一致，而**ESLint**会确保我们的代码质量，是满足最佳实践的。

![.eslintrc](/assets/images/eslintrc.png)
{: class='image-wrapper'}

### 测试

前端的测试解决方案是基于**Jest + Enzyme**的，并且会添加*prepush hook*，以确保在提交代码前，至少在本地机器上跑测试通过的。

基本上，我们会要求单元测试的代码覆盖率至少是要达到**90%**的，它表示的是*Statements*，*Branches*，*Functions*，*Lines*这四个维度的平均值。有了这些测试后，在之后的开发，重构或是修复Bug中，我们就可以依据需求任意的做出相应的修改，同时，我们也有足够的信心不会破坏已有的代码。

![jest.config.js](/assets/images/jest.config.js.png)
{: class='image-wrapper'}

### 自动化 - CI/CD

**CI**也就是我们经常说的持续集成，指的是频繁地（或者是一天多次）将代码集成到主干分支，它现在是我们开发流程中必不可少的一环。它主要可以帮助我们：

- 快速发现错误。每完成一点更新，就集成到主干，可以快速发现错误，定位错误也比较容易。
- 防止分支大幅偏离主干。如果不是经常集成，主干又在不断更新，会导致以后集成的难度变大，甚至难以集成。

持续集成的目的，就是让产品可以快速迭代，同时还能保持高质量。它的核心措施是，代码集成到主干之前，必须通过自动化测试。只要有一个测试用例失败，就不能集成。

**CD**也就是持续交付（Continuous delivery），它指的是频繁地将软件的新版本，交付给质量团队或者用户，以供评审。如果评审通过，代码就进入生产阶段。
持续交付可以看作持续集成的下一步。它强调的是，不管怎么更新，软件是随时随地可以交付的。

在ACTIVE Network，工程师在自己的分支上完成代码开发后，会先提交一个Gitlab Merge Request(MR，类似GitHub的PR)，以供团队其它成员做Code Review。代码仓库都会预先设置好[Webhook](https://docs.gitlab.com/ce/user/project/integrations/webhooks.html)，所以只要有创建，更新或是合并MR时，都会自动运行仓库的[Jenkins Pipeline](https://jenkins.io/doc/book/pipeline/getting-started/)，下面是一个典型的**Jenkinsfile**：

```groovy
#!groovy

node {
  currentBuild.displayName = sh(returnStdout: true, script: 'cat package.json | grep version | sed \'s/"//g\' | awk -F \': \' \'{print $2}\' | sed s/,//g').trim() // the name of the current Jenkins job

  gitlabCommitStatus {
    currentBuild.result = "SUCCESS"

    stage('Checkout') {
      checkout scm
    }

    stage('Install') {
      sh 'yarn'
    }

    stage('Lint') {
      sh 'npm run lint'
    }

    stage('Test') {
      try {
        sh 'npm test'
      } catch (err) {
        currentBuild.result = "FAILURE"
        error 'Testing failed'
        throw err
      } finally {
        publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: false, reportDir: 'coverage/lcov-report/', reportFiles: 'index.html', reportName: 'LCOV Report'])
      }
    }
  }
}
```

一般来说，它会执行如下的步骤：

1. 拉出最新的需要合并的分支代码
1. 安装依赖
1. 运行lint脚本
1. 运行测试（包括单元测试和集成测试）
1. 发布测试代码覆盖率报告

### 发布

通过CI流程，代码已经可以合并到主干分支了，我们就可以基于它做发布。一般发布流程包括：

- 运行ESLint
- 运行测试
- 构建（Webpack）
- 在pacage.json中升级版本号（如果没有版本，则只升级PATCH版本）
- 创建一个新的提交
- 创建一个叫`v*`的标签（如v1.15.35），并指向上一步创建的提交
- 推送至远程分支(master)
- 以**zip**格式将构建成功的前端资源文件（HTML, JS, CSS, FONTS等等）打包，并发布到远端的Nexus服务器
- 推送`v*`标签至远程仓库
- 创建并推送`latest`标签至远程仓库

这样一来，前端代码库拥有了自己的版本管理，并且每个打包的版本只是一些构建后的HTML/JS/CSS/Fonts的集合，任意的后端服务都可以使用它。

后端的生产服务器在部署前，会先备份当前代码，然后从远程的Nexus服务器上下载指定的前端代码库版本，之后将打包文件解包成本地的一个目录，再将运行路径的符号链接指向这个目录，然后重新启动应用，这方面的部署工具有**Ansible**，**Chef**，**Puppet**等，我们内部用的是**Ansible**来做部署。

### 团队建设

如何保持团队增长是一个难题！一个Team Leader的成功取决于他的存在是否推进了公司的成功，尤其是技术相关方面的成功，另外他的存在是否帮助了团队里每个工程师的成功，帮助他/她们作为一个个体的成长和成熟。

#### 技术分享

前端技术瞬息万变，库，框架，工具集，开发流程，测试，CI/CD等等，基本上每隔三个月就会更新换代，这就要求我们时刻保持更新，自我成长，多去接触更广阔的世界。

我们内部每周四定期会有一个Tech Talk，针对某一项技术（无论前后端，新或是旧，只要有用），或是一个大家平常遇到的项目问题，或是技术调研的结果。分享的目的在于将自己所了解的东西以最简单的方式讲给大家听，然后自己和其他人在这个过程中都能有所成长。

关于技术分享更多的主题，可以参考[Trello Board](https://trello.com/b/7GoAEqNs/active-front-end-tech-talk)。

#### Code Reivew

写代码可以帮助我们快速成长，同样的，作为一个代码审核人，也是一个自我学习和成长的过程。代码审核最重要的好处就是帮助我们最大限度的确保**代码质量**，统一代码风格，提早发现Bugs。同时，这也能帮助我们建立一个健康的工程师文化，促进团队成员之间的沟通，保持代码的简洁和可维护性。

代码审核绝对不是指责的工具，同事之间相互的CR过程中，我们也能从彼此那里学到很多编程技巧，并且培养起很好的编程习惯，**学习和成长**才是核心目的。

为了规范化CR流程，我们会把CR加入到我们开发流程中来，尽量自动化整个流程，另外，针对CR，我们也有一些行为准则：

- 使用**Gitlab Merge Request**来管理CR
- 每个CR必须满足[SRP](https://en.wikipedia.org/wiki/Single_responsibility_principle)原则，每次只关注某某一个功能
- 提交CR后，必须确保CI是成功的
- 使用`precommit hook`来做代码规范或是自动化测试检查，而不是留到CR中
- 每个人都要参与CR中，而不只是Senior工程师来对Junior工程师做的
- 在代码提交到主干分支前做CR（`pre-commit` CR），而不是合并到主干分支之后
- 每次提交审核的代码必须少于400行，避免提交数几个文件或是数千行代码
