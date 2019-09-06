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

调用栈的内存是有限的，如果超过它的最大内存，会报错。比如下面的代码：

```js
function foo() {
    foo();
}

foo();
```
上面代码的调用栈如下：
![js-lexical-environment](/assets/images/posts/js/js-lexical-environment02.png){:height="100%" width="100%"}

并且能看到内存溢出的错误：
<blockquote>
<p>
<font color="red">Uncaught RangeError: Maximum call stack size exceeded</font>
</p>
</blockquote>

看到调用栈这么工作，细想一下又觉得很疑惑:
- 只有function可以被压入调用栈执行吗？
- 方法里的变量是怎么存取的？
- 上图中每个蓝色的方块代表一个function，在调用栈里是什么？里面又到底有什么？
- 每个蓝色方块之间可以通信交流吗？如果可以，又是怎么做到的呢？
- 方法里每行代码到底是怎么执行的呢？

然后google查了下，要搞清楚上面这些疑问，需要理解JS中的```Lexical Environment``` ```Execution Context``` ```变量对象``` ```作用域链``` ```原型链``` ```this``` ```闭包```等等。在彻底把这些弄清楚以后，发现简直打开了新世界大门，所有的知识点就像拼图一样，一小块一小块的，突然之间就串起来了。 这篇文章先来介绍```Lexical Environment```到底是什么。

**未完待续**