---
title: JS：深入理解JavaScript-执行上下文
tags: JS
layout: post
---

在上一篇文章【[JS：深入理解JavaScript-词法环境](https://limeii.github.io/2019/05/js-lexical-environment/)】详细介绍了词法环境，它是在V8引擎词法分析阶段用来登记变量的，这样在引擎真正执行代码的时候，就知道去哪里拿变量的值。也提到过，每次执行回调函数的时候，会把方法以```执行上下文（Execution Context）```的方式压入```执行栈（Call Stack）```，执行完以后会被弹出执行栈。


比如有代码：

```js
var a;
function foo() {
    a = "hi, i am foo";
    console.log(a);
}
function baz() {
    foo();
}
baz();
```
整个代码执行过程如下：

![js-execution-context](/assets/images/posts/js/js-execution-context01.png){:height="100%" width="100%"}

图中的蓝色方块就是```执行上下文（Execution Context）```，包在蓝色方块的灰色区域就是```执行栈（Call Stack）```，整个执行栈遵循后进先出的原则：
- 在开始执行任何代码之前，都会创建全局上下文压入栈底。
- 创建词法环境，登记变量声明和函数声明。
- 引擎运行到```baz()```的时候，把```baz执行上下文```压入执行栈。
- ```baz```调用```foo```，把```foo执行上下文```压入执行栈顶。
- ```foo```调用```console.log```,把```console.log执行上下文```压入执行栈顶。
- ```console.log执行上下文```是当前正在运行的执行上下文，在console执行完以后，```console.log执行上下文```被弹出执行栈。
- ```foo执行上下文```是当前正在运行的执行上下文，在foo执行完以后，```foo执行上下文```被弹出执行栈。
- ```baz执行上下文```是当前正在运行的执行上下文，在baz执行完以后，```baz执行上下文```被弹出执行栈。

那么只有function代码可以放到执行栈中运行吗？具体执行栈里有什么，它是怎么工作的呢？


## 可执行代码（Executable Code）
事实上不仅仅是function可以作为执行上下文在执行栈中运行，在JS里定义了四种可执行代码：
- global code：整个js文件。
- function code：函数代码。
- module：模块代码
- eval code：放在eval的代码。

执行上下文（Execution Context）有三个组成部分：
- **LexicalEnvironment**：是一个词法环境(Lexical Environment)。
- **VariableEnvironment**：也是一个词法环境(Lexical Environment)，一般和LexicalEnvironment指向同一个词法环境。
- **ThisBinding**：这个就是代码里常用的this。

## 代码执行
JS引擎是按照可执行代码来执行代码的，每次执行步骤如下：
- 1：创建一个新的**执行上下文（Execution Context）**
- 2：创建一个新的**词法环境（Lexical Environment）**
- 3：把**LexicalEnvironment**和**VariableEnvironment**指向新创建的词法环境
- 4：把这个执行上下文压入**执行栈**并成为**正在运行的执行上下文**
- 5：执行代码
- 6：执行结束后，把这个执行上下文弹出执行栈

前面的代码在执行完1-4步以后，整个环境看起来是这样的：
![js-execution-context](/assets/images/posts/js/js-execution-context02.png){:height="100%" width="100%"}
<blockquote>
<p>
每个function都会新创建一个词法环境，function的词法环境中的scope，就是词法环境中的outer，作用域链就是沿着outer往上一层的词法环境里找变量/方法。
</p>
</blockquote>

执行第五步，会先给变量```a```赋值，然后执行```console.log(a)```:
![js-execution-context](/assets/images/posts/js/js-execution-context03.png){:height="100%" width="100%"}

执行第六步，```foo``` ```baz```执行完后被弹出执行栈，这两个function对象还在内存中，等待垃圾回收。
![js-execution-context](/assets/images/posts/js/js-execution-context04.png){:height="100%" width="100%"}

## 为什么要有两个词法环境：LexicalEnvironment和VariableEnvironment

**变量环境组件（VariableEnvironment）**是用来登记```var``` ```function```变量声明，**词法环境组件（LexicalEnvironment）**是用来登记```let``` ```const``` ```class```等变量声明。


在ES6之前都没有块级作用域，ES6之后我们可以用```let``` ```const```来声明块级作用域，有这两个词法环境是为了实现块级作用域的同时不影响```var```变量声明和函数声明，具体如下：

- 1：首先在一个正在运行的执行上下文内，词法环境由LexicalEnvironment和VariableEnvironment构成，用来登记所有的变量声明。
- 2：当执行到块级代码时候，会先LexicalEnvironment记录下来，记录为oldEnv。
- 3：创建一个新的LexicalEnvironment（outer指向oldEnv），记录为newEnv，并将newEnv设置为正在执行上下文的LexicalEnvironment。
- 4：块级代码内的```let``` ```const```会登记在newEnv里面，但是```var```声明和函数声明还是登记在原来的VariableEnvironment里。
- 5：块级代码执行结束后，将oldEnv还原为正在执行上下文的LexicalEnvironment。

<blockquote>
<p>
<font color="red">
块级代码内的函数声明会被当做var声明，会被提升至外部环境，块级代码运行前其值为初始值undefined。
</font>
</p>
</blockquote>

在这篇文章里介绍了执行上下文，它由三部分组成：LexicalEnvironment、VariableEnvironment和ThisBinding，并详细介绍了LexicalEnvironment和VariableEnvironment，在文章【[JS：深入理解JavaScript-this](https://limeii.github.io/2019/05/js-this/)】会详细介绍this。