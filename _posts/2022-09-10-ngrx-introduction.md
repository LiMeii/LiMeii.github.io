---
title: Angular：深入了解 NgRx 的优势
tags: Angular
layout: post
---

最近在项目组里做了一个 session 分享怎么用 NgRx，以及 NgRx 的优势是什么。顺带写篇文章记录下这次的分享内容。


在这篇文件里会介绍以下内容：
- 什么是 NgRx
- 结合 Demo 代码，介绍 NgRX 的基本用法
- 从函数编程的角度来看，NgRx 的优势是什么

# 什么是 NgRx
NgRx 是 state management library，是一个状态管理包，你也可以理解它是 RxJS 和 Redux 的结合体。


在真正开始解释什么是 NgRx 以及它的用法，先来看一个很简单的 NgRx 例子：有个搜索页面，在搜索框里输入关键字，可以查询到 Github 里相关的用户，并把这些用户显示在搜索结果页面，整体的效果如下：
![demo](/assets/images/posts/ngrx/demo.gif)

这个 Demo 的代码在这里：【[LiMeii/angular-ngrx](https://github.com/LiMeii/angular-ngrx)】


如果 NgRx 是一个 state 管理包，那什么是 state 呢？state 是一个 JS 对象。以上面这个 Demo 为例两说，在输入关键字后，用户按下 Enter 键，会触发 Github User 的 API， 把 Github 用户名字里有这个关键字的所有用户的信息返回给前端，显示在页面上，那么这里的用户信息其实就是状态，它就是一个纯 JS 对象。


NgRx 的核心是 state（即 JS 对象），那怎么管理 state 对象呢？NgRx 是通过：```actions``` ```effects``` ```reducers``` ```selector``` 管理 state，下面这张图里显示了这几个之间的关系：

![state management](/assets/images/posts/ngrx/state-management.png)

从上图中我们可以看到：如果需要读取 state 里的数据，并把这些数据显示到页面上（component view），必须要通过 ```selector``` 这个函数；如果需要更新 state 这个对象的值，那就必须通过 ```reducer``` 这个函数；那怎么触发```reducer```这个函数执行呢？必须要通过```Action```触发```reducer```函数执行；如果有异步事件要更新 state 的值呢？比如我们这个 demo 例子中，是通过 call GitHub 的 user API 拿到相关用户信息，那这些异步事件（API call）就可以写在 effects 里，在 API response 回来以后，再触发 ```Action```，从而触发```reducer```函数去更新 state 的值；```Action```是谁来触发的呢？有两种方式：一种是用户的交互操作，比如上面这个例子里，用户输入关键字后，按下 Enter 键，触发一个 search action；还有一种方式是在 effects 里，在异步事件 callback 回来以后，触发一个 search success action；


再详细介绍什么是 ```actions``` ```effects``` ```reducers``` ```selector```之前， 我们先来总结下整个事件执行的流程：用户在 search input 里输入关键字，按下 Enter 键，会触发一个 search action，在 effects 里去 call GitHub user API，这个 API response 回来以后，触发一个 search success action，这个 action 会触发 reducer 函数执行更新 state 的值，state 更新以后，会把更新后的值，通过 selector 这个函数，推送给页面（component view），component 拿到最新的 state 值后，把这些值更新显示在页面上。


大家有没有发现，整个流程，是从图中左边的用户交互（user interaction）开始，到图中右边的页面（component view）结束；如果页面又有新的用户交互操作，会重复从左到右的这个流程。这个流程叫做单向数据流（unidirectional data flow），也就是在更新操作 state 这个对象值的时候，必须要严格按照这个单向数据流的方式去做。为什么要这么做呢？其实最开始提出这种管理更新 state 对象的方式是由 Facebook 提出来的，叫 【[Flux pattern](https://facebook.github.io/flux/docs/in-depth-overview/)】，因为这个话题也非常的大，不在我们这次讨论的范围里，大家有兴趣可以去看看它的官方文档的介绍。简单来说就是，通过这种单向数据流的方式管理对象的值，可以保证对象的唯一性和正确性，从而可以避免一些数据不一致的 bug。Flux pattern 特别适合用在中大型复杂的应用里。


简单来说，NgRx 是一个通过 ```actions``` ```effects``` ```reducers``` ```selector``` 这些事件函数来管理（更新、读取）state 对象的 library。


# 如何使用 NgRx 来管理 state
了解什么是 NgRx 以后，怎么用它呢？具体代码是如何写的呢？我们还是以上面的 demo 为例，来看看怎么用 NgRx 实现一个简单的 GitHub 用户搜索功能。

## state
我们之前讲过，state 实际就是一个 JS 对象，那么我们来看看具体代码是怎么定义和初始化 state：
![state management](/assets/images/posts/ngrx/ngrx-state.png)

很简单，先定义一个 state 的结构（interface）然后初始化一下就可以。

## selector
定义好 state 后，假设这个 state 里有值，怎么把这个 state 里的 searchResult（用户信息）显示到页面上呢？我们必须通过 selector 这个纯函数，读取 state 对象的值，拿到 searchResult 后，把这个数据推送给页面，我们来看下如果定义这个 selector：

![state management](/assets/images/posts/ngrx/ngrx-selector.png)

isLoading searchedKeywords 这两个我们先不看，主要看下 searchResult 相关的，其实也很简单，通过```createFeatureSelector<searchStates.State>(featurekey)``` 拿到整个对象的值，因为真正的用户信息是在 ```state.searchResult.items``` 这个对象结构里，所以通过```createSelector(featureStateSelector, (fs) => fs.searchResult);``` 拿到 searchResult 对象，然后通过```createSelector(searchResult, (data) => data?.items)``` 拿到 用户信息 list。

## 如何把用户信息推送给页面
定义好 selector 函数读取用户信息 list，怎么把这些数据推送给页面呢？其实也很简单，在页面里 订阅用户信息就可以了，之前有讲过 NgRx 是基于 RxJS 的，所有的数据操作都是 Observable，页面代码如下：

component 代码如下：
![state management](/assets/images/posts/ngrx/ngrx-component.png)

html 页面代码如下：
![state management](/assets/images/posts/ngrx/ngrx-component-view.png)

在 compoent 里，```this.store.select(searchedGitUserLists);```是订阅用户信息 Observable，在 html 页面里 通过 ```*ngFor="let user of userLists$ | async"``` async pipe 读取 ```userLists$``` 并把用户信息循环显示在页面上。

## action
我们之前介绍过，如果想要更新 state 对象的值，必须由用户交互操作触发 action 或者 effects 的异步操作 callback 之后触发 action，再由 action 去触发 reducer 函数更新 state 的值。

那什么是 action 呢？ action 是所有可以改变 state 的操作。比如以我们这个 demo 为例，用户输入关键字以后，按下 Enter 键，这个操作会改变 state 的值，我们可以把这个操作定义为 ```search action```; API callback 回来以后，需要把 API response 回来的值更新到 state 里，我们可以把这个操作定义为```search success action```；如果 API call 失败，我们要把错误信息更新到 state 里，可以把这个操作定义为 ```search failed action```。具体代码如下：

![state management](/assets/images/posts/ngrx/ngrx-action.png)


## effects
所有的异步操作，比如常用的 API call，需要放在 effects 里，我们来看下具体代码写法：

![state management](/assets/images/posts/ngrx/ngrx-effects.png)

代码```this.actions$.pipe( ofType(searchUserActions.search) ```表示监听所有 action 事件，如果是 search action 就触发 API call，如果 API call success，触发 search success action：```searchUserActions.searchSuccess```；如果 API call failed，触发 search failed action：```searchUserActions.searchFailed```.

## reducer
有了 action 和 effects，那么如何执行 reducer 去更新 state 呢？代码如下：
![state management](/assets/images/posts/ngrx/ngrx-reducer.png)

从代码里可以看到，通过监听所有的 action，一旦 action 发生，通过 ES6 的两个语法（展开和解构）返回一个新的 state 对象。

## 用户交互如何触发 action
我们知道用户操作可以触发 action，那再用户按下 Enter 后，怎么来触发 search action 呢？
![state management](/assets/images/posts/ngrx/ngrx-action-trigger.png)
其实也很简单，一行代码就可以``` this.store.dispatch(searchUserActions.search({ userName: this.searchVal?.trim() }))```

# NgRx 的优势
看完上面如何用 NgRx 实现一个简单的 search GitHub 用户信息功能的代码，大家有没有觉得，用这种方式写代码，很繁琐，要定义很多文件，而且大部分都是把业务代码套用在 NgRx 的模板代码里，