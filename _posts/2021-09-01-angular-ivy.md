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

## 什么是 Ivy

我们在写 Angular 项目的时候，比如创建一个 component，一般会有两个文件，一个是 ts 文件，一个是 html 文件，在 html 文件里，我们会用到一些 Angular 的 directive 和 pipe（比如 *ngIf / *ngFor）这些 Angular 语法是没办法直接在浏览器里解析渲染，需要先编译成浏览器可识别的代码，才能正常运行和渲染。在 Angular9 之前的版本，负责这部分的编译叫 View Engine，Angular9 以后的版本就是用 Ivy 模板引擎把带有 Angular 语法的 html 编译成浏览器可识别的 js 代码 和 html 代码。

## 为什么要用 Ivy

Ivy 是完全重写了 Angular 的编译器 和 runtime，Ivy 有以下优势：
- 大大减少了编译时间
- 大大减少了编译以后 bundle 文件的大小
- html 文件也可以 做 tree shaking


## 什么是 ngc
在 Anuglar9 之前的模板引擎是 View Engine，Angular9 默认使用 Ivy，Ivy 重写了整个编译器，但是我们实际在写代码的时候，并没有太大的影响，比如：代码语法没有改变；从 Angular8 升级到 Angular12，业务逻辑代码语法不需要改动，可以直接运行。对于开发人员来说，从 View Engine 升级到 Ivy，并没有什么痛苦，可以说是没什么感觉，这是因为有 ngc 在中间帮我们这部分工作都做好了。


ngc 是一个 ‘compatibility compiler’，这个 ngc 编译器就负责一个任务：在编译的过程的中，会去检查```node_modules```，如果有 Angular 的 lib，会去读取这个 lib 的```metadata.json```文件和 JS 代码，会把这些代码编译成 Ivy 可以识别的代码。这个编译过程直接写在了 Angular CLI 里面，并不需要我们手动触发，在第一次跑```ng serve``` ```ng build```，会发现编译的时间要长一些。因为在第一次编译的时候，ngc 会把基于 View Engine 写的代码编译成兼容 Ivy 的代码，这个编译只是在第一次跑 ```ng serve``` ```ng build```的时候会做，但是如果你加了新的 Angular lib，会触发 ngc 编译。