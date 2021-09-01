---
title: Angular：Ivy模板引擎
tags: Angular
layout: post
---

最近做了个 Angular 升级项目，从 Angular8 升级到 Angular12，做这个升级的时候，发现 Angular9 默认使用了 Ivy 模板引擎 和 AOT 编译方式。升级项目结束以后，花时间看了下 Ivy 的相关介绍，写篇文章记录下学习成果吧。


在这篇文章中会介绍以下内容：
- 什么是 Ivy
- 为什么要用 Ivy
- 什么是 ngc
- 为什么要有 ngc
- Angular 引入 Ivy 以后，编译机制有哪些改变
- Ivy 对性能的影响