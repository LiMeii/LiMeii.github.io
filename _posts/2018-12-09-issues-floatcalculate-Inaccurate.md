---
title: JavaScript浮点数运算精度问题
tags: 问题
layout: post
---


### 问题描述

最近项目里做一个新功能，可以在系统里取钱，就涉及到浮点数的加减乘除，发现JavaScript里对浮点数进行操作的时候并不准确。

如下图一样，2-1.6正常应该是0.4，但在JS计算得出的结果是0.3999999999999999

![js-float-calculate-issue01]( https://limeii.github.io/assets/images/posts/issues/js-float-calculate-issue01.png){:height="100%" width="100%"}


在JavaScript中所有数字包括整数和小数都只有一种类型：Number，它的实现遵循IEEE 754标准，使用64位固定长度表示。IEEE 754规定，有效数字第一位默认总是1，不保存在64位浮点数之中。

也就是说，有效数字总是1.xx…xx的形式，其中xx..xx的部分保存在64位浮点数之中，最长可能为52位。因此，JavaScript提供的有效数字最长为53个二进制位（64位浮点的后52位+有效数字第一位的1）。


发生这种浮点数预算不准确是由本身JavaScript编码和设计导致的。


### 解决方案

在网上现在有很多JS library可以用来解决这个问题，比方：[number-precision](https://github.com/nefe/number-precision)，[decimal.js](https://github.com/MikeMcl/decimal.js)。

我自己用了 decimal.js, 下面图中是decimal.js API，基本概括了所有的计算方法：

![js-float-calculate-issue02]( https://limeii.github.io/assets/images/posts/issues/js-float-calculate-issue02.png){:height="100%" width="100%"}


也可以参考它的在线API文档，有非常详细的用法和例子：[decimal.js online API](https://mikemcl.github.io/decimal.js/)