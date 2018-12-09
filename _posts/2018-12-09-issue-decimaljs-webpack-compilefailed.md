---
title: decimal.js compile failed in webapck
layout: post
---

# decimal.js 在webapck打包的时候报错
<div class="title-meta">
    <span><img class="title-category-img" src="../../../assets/images/categories/bug.svg" alt="issues"></span>
    <span><a class="github-link" href="/2018/09/19/issues-tools.html">问题</a></span>
    <span class="title-bullet">•</span>
    <span>Oct 08, 2018</span>
    <span class="title-bullet">•</span>
    <span>by <a class="github-link" href="http://github.com/limeii" title="http://github.com/limeii">Mei</a></span>
</div>


### 问题描述

```html

decimal.js: 7.1.2
webpack: 3.10.0
angular: 4.2.4

```

在 [JavaScript 浮点数运算精度问题](https://limeii.github.io/2018/12/09/issues-floatcalculate-Inaccurate.html) 这篇文章中讲到了浮点数精度问题，我是用[decimal.js](https://github.com/MikeMcl/decimal.js)来解决这个问题的。


具体的操作步骤是：


1. 1 npm install decimal.js@7.1.2 --save-dev


2. 2 在component里引入decimal.js: import * as Decimal from 'decimal.js';


3. 3 代码里的具体用法，比如： new Decimal(2).minus(1.8)


在webpack打包的时候报错：**Cannot use 'new' with an expression whose type lacks a call or construct signature.**

### 问题分析

这个错误其实不是webpack本身报出来的，而是Typescript认为Decimal不能像正常的class实例化，因为decimal.js的默认export并不是Decimal。


### 解决方案

```ts
import * as Decimal from 'decimal.js';

const _decimal: any = Decimal;
let result = new _decimal(2).minus(1.8);
```
