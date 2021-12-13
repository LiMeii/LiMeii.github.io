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

定义函数的方式不同，它们的词法环境会不一样，作用域链也不一样。

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
在文章【[JS：深入理解JavaScript-词法环境](https://limeii.github.io/2019/05/js-lexical-environment/)】提到过JavaScript是静态作用域，词法环境是由代码结构决定的，开发把代码写成什么样，词法环境就是怎么样，跟方法在哪里调用没有关系。在文章【[JS：深入理解JavaScript-执行上下文](https://limeii.github.io/2019/05/js-execution-context/)】介绍了执行上下文会给每个方法创建词法环境。

我们来看下上面代码在创建执行上下文但是还没有被执行之前的词法环境：

![js-function](/assets/images/posts/js/js-function01.png){:height="100%" width="100%"}

函数```foo``` ```bar```都是函数声明，函数声明在创建词法环境的时候，会被初始化，所以上图```fooFunctionEnviroment```和```barFunctionEnviroment```都在内存中都已经初始化了，这就是我们所说的**函数提升**。


全局变量```var a=2```是变量声明，变量声明在创建词法环境的时候，会被初始化为undefined，所以上图中的```a=undefined```，这就是我们所说的**变量提升**。函数```bar```中的变量a也类似，只不过它被初始化在```barFunctionEnviroment```词法作用域里。


当执行完第一行代码```var a=2```，给全局变量a赋值，此时的执行上下文和词法环境如下：
![js-function](/assets/images/posts/js/js-function02.png){:height="100%" width="100%"}

当执行函数```bar()```里代码```var a=3```，给bar中的变量赋值，此时的执行上下文和词法环境如下：
![js-function](/assets/images/posts/js/js-function03.png){:height="100%" width="100%"}

当执行函数```foo()```里代码```console.log(a)```，此时的执行上下文和词法环境如下：
![js-function](/assets/images/posts/js/js-function04.png){:height="100%" width="100%"}

这时候发现在```fooFunctionEnviroment```词法环境里没有变量a，就会到它的上一层词法环境去找，**函数的```scope```里记录了它上一层词法环境**，foo函数的上一层词法环境是```GlobalEnvironment```全局词法环境，所以输出的是```2```而不是```3```。


**从上面的例子可以看出，对于函数声明，会有函数提升，函数的初始化发生在词法环境创建的时候，函数表达式的词法环境还是由代码结构决定的，开发把代码写成什么样，词法环境就是怎么样，跟方法在哪里调用没有关系。**


## 函数表达式

我们来看个函数表达式的例子：

```js

foo(); //TypeError: foo is not a function
bar(); //TypeError: bar is not a function

console.log(foo); // undefined
console.log(bar); // undefined

var a = 2;

var foo = function () {
    console.log(a);
    console.log(b);

}

var bar = function _bar() {
    var a = 3;
    var b =4
    foo();
}
```
我们可以看到不管是匿名函数表达式还是命名函数表达式，```foo``` 和```bar```这两个变量有提升初始化为undefined，但是函数体并没有**函数提升**。


我们把代码改一下：
```js
var a = 2;

var foo = function () {
    console.log(a); // 2
    console.log(b); // Uncaught ReferenceError: b is not defined

}

var bar = function _bar() {
    var a = 3;
    var b =4
    foo();
}

bar();
```
运行上面代码，a输出的是全局变量a的值，全局变量里没有b，就报了ReferenceError。


我们来看下上面代码在创建执行上下文但是还没有被执行之前的词法环境：
![js-function](/assets/images/posts/js/js-function05.png){:height="100%" width="100%"}

当执行完第一行代码```var a=2```，给全局变量a赋值，此时的执行上下文和词法环境如下：
![js-function](/assets/images/posts/js/js-function06.png){:height="100%" width="100%"}

当执行```bar()```，会给bar的函数表达式新创建一个执行上下文和词法环境，此时的执行上下文和词法环境如下：
![js-function](/assets/images/posts/js/js-function07.png){:height="100%" width="100%"}

当执行```foo()```，会给foo的函数表达式新创建一个执行上下文和词法环境，此时的执行上下文和词法环境如下：
![js-function](/assets/images/posts/js/js-function08.png){:height="100%" width="100%"}

这时候发现在```fooFunctionEnviroment```词法环境里没有变量a和b，就会到它的上一层词法环境去找，函数的```scope```里记录了它上一层词法环境，foo函数的上一层词法环境是```GlobalEnvironment```全局词法环境，所以输出的是```2```和```ReferenceError```。


我们再把上面的代码改一下：

```js
var a = 2;

var bar = function _bar() {
    var a = 3;
    var b =4
    var foo = function () {
        console.log(a); // 3
        console.log(b); // 4
    
    }
    foo();
}

bar();
```
运行上面代码，a输出的是bar中变量a和b的值。


我们来看下上面代码在创建执行上下文但是还没有被执行之前的词法环境：
![js-function](/assets/images/posts/js/js-function09.png){:height="100%" width="100%"}

当执行完第一行代码```var a=2```，给全局变量a赋值，此时的执行上下文和词法环境如下：
![js-function](/assets/images/posts/js/js-function10.png){:height="100%" width="100%"}

当执行```bar()```，会给bar的函数表达式新创建一个执行上下文和词法环境，此时的执行上下文和词法环境如下：
![js-function](/assets/images/posts/js/js-function11.png){:height="100%" width="100%"}

当执行```foo()```，会给foo的函数表达式新创建一个执行上下文和词法环境，它的上一层词法环境是```barFunctionEnviroment```，此时的执行上下文和词法环境如下：
![js-function](/assets/images/posts/js/js-function12.png){:height="100%" width="100%"}

在执行```foo()```，发现没有变量a和b，就到它的上一层词法环境```barFunctionEnviroment```去找，所以输出的是3和4。


**从上面的例子可以看出，对于函数表达式，它们的函数体不会函数提升，函数的初始化发生在代码执行的时候，函数表达式的词法环境还是由代码结构决定的，开发把代码写成什么样，词法环境就是怎么样，跟方法在哪里调用没有关系。**

## 立即执行函数

立即执行函数和函数表达式是一样的，不会函数提升，函数的初始化发生在代码执行的时候，词法环境还是由代码结构决定的。


## new Funcion(arg1,arg2...,argn,body)创建函数

```js
var c = 10;
var sum = new Function('a', 'b', 'return a + b+c');

console.log(sum(2, 6)); //18
```

用new Function(arg1,arg2,...,argn,body) 创建函数的过程有和上面函数表达式类似，不同地方在于，创建函数使用的scope是直接使用全局词法环境(glbal enviroment),而不管当前运行上下文，一律取全局词法环境(glbal enviroment)。


## 思考题

```js
console.log(foo);

function foo(){
    console.log("foo");
}

var foo = 1;
```

在这里会打印函数，而不是 undefined。


这是因为在进入执行上下文的时候，首先会处理函数声明，其次会处理变量声明，**如果变量名称已经跟已经声明的形式参数或函数相同，则变量声明不会干扰已经存在的这类属性。**

```js
function foo() {
    // 函数声明，必须要有函数名字
    console.log('this is function declaration');
    console.log(foo); // [function foo]
    foo = 20;
    console.log(foo); // 20
    console.log(window.foo); // 20
}
foo();


var foo = 10;
(function foo() {
    console.log(foo); // [function foo]
    // 内部作用域，会先去查找是已有变量foo的声明，有就直接赋值20，确实有了呀，发现了具名函数，拿此foo做赋值
    // IIFE的函数无法进行赋值（内部机制，类似const定义的常量）所以无效
    foo = 20;
    console.log(foo); // [function foo]
    console.log(window.foo)//10
})()

// 所以严格模式下能看到错误：uncaught typeerror: assignment to constant variable
var foo = 10;
(function foo() {
    'use strict'
    console.log(foo); 
    foo = 20; // uncauth typerror: assignment to constant variable
    console.log(foo); 
    console.log(window.foo)
})()


var b = 10;
(
    function b() {
        window.b = 20;
        console.log(b); //【function b】
        console.log(window.b) // 20;
    }
)()

var b =10;
(function b(){
    var b = 20; // IIFE内部变量
    console.log(b); //20
    console.log(window.b) //10
})


var a = 10;
(function () {
    console.log(a) // undefined
    a = 5
    console.log(a) // 5
    console.log(window.a) //  10
    var a = 20;
    console.log(a) //20
})()
// var a = 20, 这个 a 会有变量提升，所以在 IIFE 内部，会先有 a 的声明并赋值为undefined

//如果把 var a = 20，这段去掉，那么就只能去拿外部a的值了
var a = 10;
(function () {
    console.log(a) // 10
    a = 5
    console.log(a) // 5
    console.log(window.a) //  5
})()


```

## 总结

- 函数声明，会有变量提升，函数初始化是发生在函数创建时运行上下文的词法环境里。

- 函数表达式/匿名函数/立即执行函数，没有变量提升，函数初始化是发生在代码执行的时候。

- 函数的词法环境中的scope，是用来记录上一层的词法环境。

- 如果函数有形参，那么这些形参都属于函数的词法环境。

