---
title: JS：深入理解JavaScript(Lexical Environment)
tags: JS
layout: post
---

在文章【[JS：详解Event Loop运行机制，以及microtasks和macrotask的执行顺序](https://limeii.github.io/2019/09/js-eventloop/)】这篇文章中简单介绍了Call Stack（调用栈），在JS中所有代码都是在调用栈中执行的，遵循后进先出的原则。比如有如下代码：

```js
function foo() {
    console.log("hi, i am foo");
}

function bar() {
    foo();
}

function baz() {
    bar();
}

baz();
```
上面代码的调用栈如下：

![js-lexical-environment](/assets/images/posts/js/js-lexical-environment01.png){:height="100%" width="100%"}

调用栈的内存是有限的，比如下面的代码：

```js
function foo() {
    foo();
}

foo();
```
上面这段代码的调用栈如下：
![js-lexical-environment](/assets/images/posts/js/js-lexical-environment02.png){:height="100%" width="100%"}

可以看到如果超过调用栈的最大内存，会报内存溢出的错误：
<blockquote>
<p>
<font color="red">Uncaught RangeError: Maximum call stack size exceeded</font>
</p>
</blockquote>

再了解Event Loop和调用栈的运行机制之后，仔细想了一下又觉得很疑惑:
- 只有function可以被压入调用栈执行吗？
- 方法里的变量是怎么存取的？
- 上图中每个蓝色的方块代表一个function，在调用栈里是什么？里面又到底有什么？
- 每个蓝色方块之间可以通信交流吗？如果可以，又是怎么做到的呢？
- 方法里每行代码到底是怎么执行的呢？

然后google查了下，发现上图执行栈中的每个蓝色方块有个专业名称叫```执行上下文(Execution Context)```，紧接着就是一大串的名词：```Lexical Environment``` ```Execution Context``` ```变量对象``` ```作用域链``` ```原型链``` ```this``` ```闭包```等等。刚开始有点懵，在彻底把这些弄清楚以后，发现简直打开了新世界大门，所有的知识点就像拼图一样，一小块一小块的，突然之间就串起来了。 


这篇文章里先来介绍```Lexical Environment```到底是什么。

## Lexical Environment

在介绍```Lexical Environment```之前，我们先看下在V8里JS的编译执行过程，大致上可以分为五个阶段：
- 第一步：```执行上下文```在调用栈里被执行的时候，V8引擎会把代码从上到下一行一行的先做分词/词法分析(Tokenizing/Lexing)，比如```var a = 2；```这段代码，会被分词为：```var``` ```a``` ```2```和```;```这样的原子符号(atomic token)。
- 第二步：在分词结束以后，会做代码解析，引擎将 token 解析翻译成一个AST(抽象语法树)。
- 第三步：引擎遇到变量声明，会给变量分配内存但不赋值；遇到函数声明会在内存里生成函数对象，函数如果有形参会和变量声明一样处理，分配内存但是不赋值。
- 第四步：引擎生成CPU可以执行的机器码。
- 第五步：执行代码。


那```Lexical Environment```是什么呢？在上面的五个步骤中又起到什么的作用呢？

**未完待续**