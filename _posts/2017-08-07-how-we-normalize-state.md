---
layout: post
title: "How We Normalize State"
description: "How We Normalize State"
category: ['React', 'Redux', 'Normalize']
tags: ['React', 'Redux', 'Normalize', 'Normalizr']
---
{% include JB/setup %}

上一篇讲到开发**React + Redux** Web App时的最佳目录结构，今天想接着聊聊另外一个话题 - Normalizing State（中文一般翻译为**状态范式化**），以及如何利用 selectors 去访问这些数据。
{: .countheads }

* ToC
{:toc}

## 尽量使用索引和*selectors*分别来存储和访问*state*

设计范式化的 State 范式化的数据主要包含下面几个概念：

- 任何类型的数据在*state*中都有自己的『表』结构
- 任何『数据表』应将各个项目存储在对象中，其中每个项目的*ID*作为*key*，项目本身作为*value*
- 任何对单个项目的引用都应该根据存储项目的*ID*来完成
- *ID*数组应该用于排序

与原始的嵌套形式相比，有下面几个地方的改进：

- 每个数据项只在一个地方定义，不会造成数据的不一致或是冗余，如果数据项需要更新的话不用在多处改变
- *reducer*逻辑不用处理深层次的嵌套
- 检索或者更新给定数据项的逻辑变得简单与一致
- 提高性能，每个组件只负责更新自己的数据源，不需要更新整个数据源。比如，*user*属于一个 company，那么更新 company时，仅仅只需要更新`companys.byId[companyId]`这部分，这也就意味着在*UI*中只有数据发生变化的一部分才会发生更新。如果是之前的话，由于是`user -> company`结构，更新 company 时，同时也会更新*user*，还有整个*user*列表，从而会让 connect 到*user*列表的所有组件全部再次重新渲染。

如果你想了解更多关于状态范式化的基本概念和它所带来的好处，请参考[Redux官方文档](http://redux.js.org/docs/recipes/reducers/NormalizingStateShape.html)。

先说一下结果，我们最终所期望的 entities*state*是这样的：

```
{
  "readers": {
    "byId": {},
    "ids": []
  },
  "users": {
    "byId": {},
    "ids": []
  },
  "subReaders": {
    "byId": {},
    "ids": []
  },
  "shareUsers": {
    "byId": {},
    "ids": []
  },
  "readerTags": {
    "byId": {},
    "ids": []
  },
  "companys": {
    "byId": {},
    "ids": []
  }
}
```

其中，*byId*就是一个大的*hashmap*来存储数据，而*ids*存储的则是有序的所有的*id*集合。

在状态范式化以前，如果我们想要筛选出一个特定的*user*对象，那么我们就不得不去迭代所有的*users*，如果*users*很大的话，这将是一个非常耗时的操作。
如果我们还想对*users*根据搜索条件筛选出它的一个子集呢？我们可能会另外在状态树中再单独维护一个数组用于保持筛选结果。

而现在，如果我们需要根据*useId*查找该*user*，那么我们可以这样简单地做：

```
export const selectUser = createDeepEqualSelector(
  [getUsers, getUserId],
  (users, userId) => users.get(String(userId), Map()),
)
```

使用范式化数据后，它并不需要我们再去迭代整个列表，这样做不仅节约了时间而且还简化了代码逻辑。

> 为了区分*selector*函数，所有*selectors*默认都是以**select**为前缀。

另外，我们经常会做的一件事情就是根据范式化的数据结构来渲染整个列表数据（如*users*列表），现在我们可以用另外一个*selector*来实现，它接受*users*对象列表和*users ids* 数组，从而返回所有的*users*列表数据，同时我们可以重用该*selector*。

```
const getUsers =*state*=> state.users.byId
const getUserIds =*state*=> state.users.ids
const selectUsers = createDeepEqualSelector(
  [getUsers, getUserIds],
  (users, userIds) => userIds.map(id => users.get(String(id))),
)
```

同样的，我们还可以用另外一个*selector*来获取筛选后的*users*：

```
export const selectShareUsers = createDeepEqualSelector(
  [getShareUsers, getShareUserIds],
  (shareUsers, shareUserIds) =>
    shareUserIds
      .map(id => shareUsers.get(String(id)))
      .sortBy(item => item.get('companyName')),
)

```

> 我们使用了**reselect**来作为我们的*selector*库，它会为我们带来如下的优势：
  * *Selectors*用于计算派生数据，从而允许 Redux 只存储尽可能少的状态
  * *Selectors*效率相当高，一个*selector*只有在它的参数发生变化时才会重新计算
  * *Selectors*可以自由组合，一个*selector*可以成为其它*selector*的输入

这样一来，我们就实现了数据的存储和获取，同时，*selector*模式也增强了代码的可维护性。*Selectors*中封装的是*Web App*所需要的当前 state的所有信息，所以我们只需要将*Web App*的当前*state*传递给它，它自己会知道如何计算出我们需要的数据，**这样做的好处是可以把组件和*state*彻底解耦**。

设想一下，如果哪天我们想要重构我们的*state*（比如`users`-> `xxooUsers`，:(）。如果没有*selectors*，那么我们就必须根据新的*state*（`xxooUsers`）去更新我们所有的视图组件代码（例如 `mapStateToProps`中获取*users*的代码逻辑），更糟糕的是，随着有类似这种更新需求的视图组件越来越多，这种更新将会越来越困难，如果此时你没有**UT**给你足够的信心加持，那简直 是一场噩梦。为了避免这种牵一发而动全身的问题，最简单的方法是使用*selectors*在视图组件中获取我们需要的*state*，如果*state*结构有任何修改，那么我们只需要更新*selector*，就可以让它能正确的访问到*state*，好处是所有的视图组件根本不需要做任何的修改，也可以继续正常工作。

## 那么我们是如何范式化 state的呢？

与服务端的数据交互通常是通过严格的 RESTFul API的形式，我们需要在将该数据需要引入状态树之前转化为规范化形态。[Normalizr](https://github.com/paularmstrong/normalizr) 库可以帮助实现这个。你可以定义 schema 的类型和关系，将 schema 和响应数据提供给 Normalizr，他会输出响应数据的范式化变换。

下面是 schema 的定义示例：

```
import { schema } from 'normalizr'

const reader = new schema.Entity('readers')
const subReader = new schema.Entity('subReaders', {}, { idAttribute: 'rid' })
const readerInfo = new schema.Entity('readerInfos')
const readerTag = new schema.Entity('readerTags')
const company = new schema.Entity('companys')
const user = new schema.Entity('users', { company })
const shareUser = new schema.Entity('shareUsers')
const account = new schema.Entity('accounts')

export default {
  READERS: [reader],

  SUB_READERS: [subReader],

  READER_INFOS: [readerInfo],

  READER_TAGS: [readerTag],

  USER: user,

  SHARE_USERS: [shareUser],

  ACCOUNTS: [account],

  COMPANY: company,
  COMPANYS: [company],
}

```

由于内部采用了 PromiseMiddlware 来处理异步数据请求，每个异步请求都会由一个 action 发出，如果需要对服务端的返回数据做 Normalizing，那么只需要在`meta`上添加上相应的 schema，那么 PromiseMiddleware 会自动根据该 schema 和响应数据生成范式化数据。

```
export const fetchReaders = ({ suppressLoading } = {}) => ({
  type: FETCH_READERS,
  promise: (dispatch, getState, api) => api.get(URL.fetchReaders),
  meta: { suppressLoading, schema: Schemas.READERS },
})
```

```
...
if (schema) {
  finalPayload = normalize(payload, schema)
}

if (meta && meta.normalize) {
  finalPayload = meta.normalize(payload, { dispatch, getState })
}
...
```

最后，范式化的数据会由相应的`entities`中的*reducer*做处理，然后存储到`entites.readers`中。

```
const readersById = handleActions(
  {
    [DELETE_READER_SUCCESS]: (state, { meta }) =>
      state.filter((r, id) => id !== String(meta.reader)),

    [ADD_READER_SUCCESS]: (state, { payload }) =>
      state.merge({
        [payload.id]: payload,
      }),

    [FETCH_READER_INFO_SUCCESS]: (state, { payload }) =>
      state.mergeDeepIn([String(payload.id)], payload),

    [SAVE_READER_INFO_SUCCESS]: (state, { payload }) =>
      state.mergeDeepIn([String(payload.id)], payload),
  },
  Map(),
  (state, action) => entityById('readers')(state, action),
)

const readers = handleActions(
  {
    [DELETE_READER_SUCCESS]: (state, { meta }) =>
      state.filter(id => id !== meta.reader),

    [ADD_READER_SUCCESS]: (state, { payload }) => state.concat(payload.id),

    [FETCH_READER_INFO_SUCCESS]: (state, { payload }) =>
      state.concat(payload.id),
  },
  OrderedSet(),
  (state, action) => entity('readers')(state, action),
)

export default combineReducers({
  byId: readersById,
  ids: readers,
})
```
