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
NgRx 其实就是 state management library，是一个状态管理包，你也可以理解它是 RxJS 和 Redux 的结合体。


在真正开始解释什么是 NgRx 以及它的用法，先来看一个很简单的例子：就是有个搜索页面，在搜索框里输入关键字，可以查询到 Github 里相关的用户，并把这些用户显示在搜索结果页面，整体的效果如下：
![demo](/assets/images/posts/ngrx/demo.gif)

这个 Demo 的代码在这里：【[LiMeii/angular-ngrx](https://github.com/LiMeii/angular-ngrx)】


如果 NgRx 是一个 state 管理包，那什么是 state 呢，其实 state 就是一个 JS 对象。以上面这个 Demo 为例两说，在输入关键字后，用户按下 Enter 键，会触发 Github User 的 API， 把 Github 用户名字里有这个关键字的所有用户的信息返回给前端，显示在页面上，那么这里的用户信息其实就是状态，它就是一个纯 JS 对象。


NgRx 是通过：```actions``` ```effects``` ```reducers``` ```selector``` 管理 state，下面这张图里显示了这几个之间的关系：

![state management](/assets/images/posts/ngrx/state-management.png)