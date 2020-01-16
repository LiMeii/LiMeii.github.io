---
title: JS：深入理解JavaScript-词法环境
tags: JS
layout: post
---

在文章【[JS：详解Event Loop运行机制，以及microtasks和macrotask的执行顺序](https://limeii.github.io/2019/05/js-eventloop/)】这篇文章中简单介绍了Call Stack（调用栈），在JS中所有代码都是在调用栈中执行的，遵循后进先出的原则。


在了解Event Loop和调用栈的运行机制之后，仔细想了一下又觉得很疑惑:
- 只有function可以被压入调用栈执行吗？
- JS引擎在执行代码时，是从哪里找到要引用的变量值，函数调用又是怎么找到对应的函数的呢？
- 图中每个蓝色的方块代表一个function？在调用栈里是什么？里面又到底有什么？
- 每个蓝色方块之间可以通信交流吗？如果可以，又是怎么做到的呢？
- 方法里每行代码到底是怎么执行的呢？

然后google查了下，发现执行栈中的每个蓝色方块有个专业名称叫```执行上下文(Execution Context)```，紧接着就是一大串的名词：```Lexical Environment``` ```Execution Context``` ```变量对象``` ```作用域链``` ```原型链``` ```this``` ```闭包```等等。刚开始有点懵，在彻底把这些弄清楚以后，发现简直打开了新世界大门，所有的知识点就像拼图一样，一小块一小块的，突然之间就串起来了。 


这篇文章先来介绍```Lexical Environment```到底是什么。

## Lexical Environment

在介绍```Lexical Environment```之前，我们先看下在V8里JS的编译执行过程，大致上可以分为三个阶段：
- 第一步：V8引擎刚拿到```执行上下文```的时候，会把代码从上到下一行一行的先做分词/词法分析(Tokenizing/Lexing)。分词是指：比如```var a = 2；```这段代码，会被分词为：```var``` ```a``` ```2```和```;```这样的原子符号(atomic token)；词法分析是指：登记变量声明、函数声明、函数声明的形参。

- 第二步：在分词结束以后，会做代码解析，引擎将 token 解析翻译成一个AST(抽象语法树)， 在这一步的时候，如果发现语法错误，就会直接报错不会再往下执行。

```js
var greeting = "Hello";
console.log(greeting);
greeting = ."Hi";
// SyntaxError: unexpected token .
// 没有打印出 hello，而是先报错，说明JS引擎在真正执行代码之前，会做代码解析。
```

- 第三步：引擎生成CPU可以执行的机器码。


在第一步里有个词法分析，它用来登记变量声明、函数声明、函数声明的形参，后续代码执行的时候就知道去哪里拿变量的值和函数了，这个登记的地方就是```Lexical Environment（词法环境）```。


词法环境有两个组成部分：
- **1：环境记录（Environment Record）**，这个就是真正登记变量的地方。
   - **1.1：声明式环境记录（Declarative Environment Record）**：用来记录直接有标识符定义的元素，比如变量、常量、let、class、module、import以及函数声明。
  - **1.2：对象式环境记录（Object Environment Record）**：主要用于with和global的词法环境。
- **2：对外部词法环境的引用（outer）**，它是作用域链能够连起来的关键。


其中 **声明式环境记录（Declarative Environment Record）**，又分为两种类型：
- **函数环境记录（Function Environment Record）**：用于函数作用域。
- **模块环境记录（Module Environment Record）**：模块环境记录用于体现一个模块的外部作用域，即模块export所在环境。

词法环境与我们自己写的代码结构相对应，也就是我们自己代码写成什么样子，词法环境就是什么样子。词法环境是在代码定义的时候决定的，跟代码在哪里调用没有关系。所以说JavaScript采用的是词法作用域（静态作用域）。


我们来看个例子：

```js
var a = 2;
let x = 1;
const y = 5;

function foo() {
    console.log(a);

    function bar() {
        var b = 3;
        console.log(a * b);
    }

    bar();
}
function baz() {
    var a = 10;
    foo();
}
baz();

```
它的词法环境关系图如下：
![js-lexical-environment](/assets/images/posts/js/js-lexical-environment03.png){:height="100%" width="100%"}

我们可以用伪代码来模拟上面代码的词法环境：

```js
// 全局词法环境
GlobalEnvironment = {
    outer: null, //全局环境的外部环境引用为null
    GlobalEnvironmentRecord: {
        //全局this绑定指向全局对象
        [[GlobalThisValue]]: ObjectEnvironmentRecord[[BindingObject]],
        //声明式环境记录，除了全局函数和var，其他声明都绑定在这里
        DeclarativeEnvironmentRecord: {
            x: 1,
            y: 5
        },
        //对象式环境记录，绑定对象为全局对象
        ObjectEnvironmentRecord: {
            a: 2,
            foo:<< function>>,
            baz:<< function>>,
            isNaNl:<< function>>,
            isFinite: << function>>,
            parseInt: << function>>,
            parseFloat: << function>>,
            Array: << construct function>>,
            Object: << construct function>>
            ...
            ...
        }
    }
}
//foo函数词法环境
fooFunctionEnviroment = {
    outer: GlobalEnvironment,//外部词法环境引用指向全局环境
    FunctionEnvironmentRecord: {
        [[ThisValue]]: GlobalEnvironment,//this绑定指向全局环境
        bar:<< function>> 
    }
}
//bar函数词法环境
barFunctionEnviroment = {
    outer: fooFunctionEnviroment,//外部词法环境引用指向foo函数词法环境
    FunctionEnvironmentRecord: {
        [[ThisValue]]: GlobalEnvironment,//this绑定指向全局环境
        b: 3
    }
}

//baz函数词法环境
bazFunctionEnviroment = {
    outer: GlobalEnvironment,//外部词法环境引用指向全局环境
    FunctionEnvironmentRecord: {
        [[ThisValue]]: GlobalEnvironment,//this绑定指向全局环境
        a: 10
    }
}

```
我们可以看到词法环境和我们代码的定义一一对应，每个词法环境都有一个```outer```指向上一层的词法环境，当运行上面代码，函数bar的词法环境里没有变量a，所以就会到它的上一层词法环境（foo函数词法环境）里去找，foo函数词法环境里也没有变量a，就接着去foo函数词法环境的上一层（全局词法环境）去找，在全局词法环境里```var a=2```，沿着```outer```一层一层词法环境找变量的值就是**作用域链**。在沿着作用域链向上找变量的时候，找到第一个就停止往上找，如果到全局词法环境里还是没有找到，因为全局词法环境里的```outer```是null，没办法再往上找，就会报ReferenceError。


这段代码的输出为：
```
2
6
```

## 变量提升vs函数提升

在前面我们提到过，V8引擎执行代码的大致可以分为三步，先做分词和词法分析，然后解析生成AST，最后生成机器码执行代码。在词法分析的时候会生成```词法环境```登记变量，对于变量声明和函数声明，词法环境的处理是不一样的。


在词法分析的时候：
- 对于变量声明```var a=2;``` ```let x=1;```，给变量分配内存并初始化为undefined，赋值语句是在第三步生成机器码真正执行代码的时候才执行。
- 对于函数声明```function foo(){...}```，会在内存里创建函数对象，并且直接初始化为该函数对象。

这就是JS的**变量提升和函数提升**，我们来看个例子;

```js
var c;

function functionDec() {
    console.log(c)
    c = 30;
}

functionDec();
```
最后运行结果是：undefined

从词法分析到代码执行，变量提升和变量赋值变化如下：

![js-lexical-environment](/assets/images/posts/js/js-lexical-environment04.png){:height="100%" width="100%"}


如果整个变量就没有定义，如下：

```js
function functionDec() {
    console.log(c)
    c = 30;
}

functionDec();
```
运行代码，会有ReferenceError，运行结果如下：
```
Uncaught ReferenceError: c is not defined
    at functionDec (<anonymous>:4:17)
    at <anonymous>:8:1
```

在这篇文章里，介绍了```Lexical Environment```，它是在V8引擎词法分析阶段用来登记变量的，这样在引擎真正执行代码的时候，就知道去哪里拿变量的值，那代码在执行的过程中，具体又做了什么呢？在下篇文章【[JS：深入理解JavaScript-执行上下文](https://limeii.github.io/2019/05/js-execution-context/)】会详细介绍。
