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


# 如何用 NgRx 来管理 state