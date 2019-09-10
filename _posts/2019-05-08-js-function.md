---
title: JS：深入理解JavaScript-函数
tags: JS
layout: post
---

JavaScript中定义函数有这几种方式：
- 函数声明
- 函数表达式
- 立即执行函数
- new Funcion(arg1,arg2...,argn,body)创建函数。

定义函数的方式不同，它们的词法环境会不一样，它们的作用域链也不一样。

## 函数声明

我们来看一个函数声明的例子：

```js
var a = 2;
function foo() {
    console.log(a); // 2
}

function bar() {
    var a = 3;
    foo();
}
bar();
```
在文章【[JS：深入理解JavaScript-词法环境](https://limeii.github.io/2019/05/js-lexical-environment/)】提到过JavaScript是静态作用域，词法环境是由代码结构决定的，开发把代码写成什么样，词法环境就是怎么样，跟方法在哪里调用没有关系。在文章【[JS：深入理解JavaScript-执行上下文](https://limeii.github.io/2019/05/js-execution-context/)】介绍了执行上下文会给每个方法创建词法环境。我们来看下上面代码在创建执行上下文但是还没有被执行之前的词法环境：

![js-function](/assets/images/posts/js/js-function01.png){:height="100%" width="100%"}

函数```foo``` ```bar```都是函数声明，函数声明在创建词法环境的时候，会被初始化，所以上图```fooFunctionEnviroment```和```barFunctionEnviroment```都在内存中都已经初始化了，这就是我们所说的**函数提升**。


全局变量```var a=2```是变量声明，变量声明在创建词法环境的时候，会被初始化为undefined，所以上图中的```a=undefined```，这就是我们所说的**变量提升**。函数```bar```中的变量a也类似，只不过它被初始化在```barFunctionEnviroment```词法作用域里。


**未完待续**