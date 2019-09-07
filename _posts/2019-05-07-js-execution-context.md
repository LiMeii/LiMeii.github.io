---
title: JS：深入理解JavaScript-执行上下文
tags: JS
layout: post
---

在上一篇文章【[JS：深入理解JavaScript-词法环境](https://limeii.github.io/2019/05/js-lexical-environment/)】详细介绍了词法环境，它是在V8引擎词法分析阶段用来登记变量的，这样在引擎真正执行代码的时候，就知道去哪里拿变量的值。也提到过，每次执行回调函数的时候，会把方法以```执行上下文（Execution Context）```的方式遵循后进先出的原则压入```执行栈（Call Stack）```执行代码，执行完以后会被弹出执行栈。


那么只有function代码可以放到执行栈中运行吗？具体执行栈里有什么，它是怎么工作的呢？


**未完待续**