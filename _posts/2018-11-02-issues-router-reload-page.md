---
title: Angular路由到新的模块时候，整个页面都被刷新
tags: 问题
layout: post
---


### 问题描述

#### 开发环境如下：

```
node version： 8.9.3

npm version： 5.6.0

anguar version: 5.0.0

webpack version: 3.10.0

webpack-dev-server: 2.9.3

chrome： 69.0.3497.100 
```

最近在用angular开发项目，在搭建项目路由的时候，遇到一个非常奇怪的问题：路由跳转的时候有时候会整个页面刷新，同时在url的#前面会自动带一个问号。

比如url为： http://localhost:3000/#/login，在login成功以后希望跳转到http://localhost:3000/#/dashboard，开发中用了HMR,所以页面是不会刷新的。


实际情况是login成功以后 url变成 http://localhost:3000/?#/login, 整个页面刷新以后回到 http://localhost:3000/#/login。


### 问题分析

**1.先打个断点调试看下**


不管什么情况，先打个断点调试看下是不是代码有问题，断点调试下来每一步都跟预期一样，没什么问题，跑完最后 this.router.navigate(['dashboard']), 页面开始刷新，出现上面的问题。


**2.我路由用错了?**


第一反应是我路由没用对，把官方文档Router那一章从头看了一遍，好像也没什么问题。google一下好像都没人遇到过这个问题。


反复看了代码，看不出什么问题，因为是通过this.router.navigate(['dashboard'])跳转的，我尝试直接在浏览器中输入http://localhost:3000/#/dashboard，这次是正常跳转到dashboard页面。好了，很大概率是我是我login那块的代码出问题了。


**3.那把login逻辑代码去掉试试看**


把login那块的代码逻辑去掉，在点击submit buttion以后只留一行路由跳转代码：this.router.navigate(['dashboard'])，代码跑起来以后，居然也是正常的。


现在回头看下login的逻辑代码，在点击login submit的以后，先拿到用户输入的用户名和密码，然后调了一个http.get方法拿到mock的登入成功数据，最后是路由跳转。


也就是说跟后台交互的时候，会带在url里加上?，导致页面刷新，这也太奇怪了吧。


**4.换个浏览器试试看？**


开发用的是chrome，那换个浏览器试试看，换成FireFox，FireFox里没有这个问题！！！！


那就是Chrome的问题，google了下，如果有Form表单，并且没有用action="/formsubmit"，那么Chrome会发一个GET request：在当前的url里带上问号,导致整个页面刷新。


### 解决方案

在submit方法中加上以下代码：

```ts
submit(e) {
    e.preventDefault()
}
```