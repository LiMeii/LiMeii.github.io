---
title: JS：详解Event Loop运行机制，以及microtasks和macrotask的执行顺序
tags: JS
layout: post
---

在这篇文章中会介绍以下内容：
- engine runtime和call stack简介（以V8引擎为例）

- Event Loop运行机制的详解

- microtasks和macrotask的执行顺序

## engine runtime和call stack简介

在chrome浏览器和nodejs里都是用V8引擎解析和运行JS代码，我们先来看下V8引擎的简化图：

![js-eventloop](/assets/images/posts/js/js-eventloop01.png){:height="80%" width="80%"}

上图中Heap是用来做内存分配，```Call Stack```是用来执行JS代码，由于JS是单线程所以只有一个```Call Stack```。实际我们写网页开发的时候，除了一些JS代码，我们还会大量用到：DOM事件、AJAX(XMLHttpRequest)、setTimeout等等一些异步事件。从上图可以看出，这些异步事件都没有在V8引擎里，事实上这些异步事件不属于V8引擎，而是属于浏览器，并且DOM事件、AJAX(XMLHttpRequest)、setTimeout都分别有单独的线程来处理。由于```Call Stack```执行（JS运行线程）和页面渲染线程是互斥的，如果所有的事情都由V8引擎处理，这样肯定会导致页面卡顿。


浏览器多线程和callback机制完美避免了页面卡顿的问题。DOM事件、AJAX(XMLHttpRequest)、setTimeout这些异步事件在各自单独的线程处理完以后，每个异步事件都有callback回调函数，V8引擎再把这些回调函数放在```Call Stack```执行。上述整个运行机制可以称为是runtime，可以简化如下图：

![js-eventloop](/assets/images/posts/js/js-eventloop02.png){:height="80%" width="80%"}

如上图所示，web异步事件结束以后，会有callback，然后runtime把这些callback事件放到```Callback Queue```里，一旦```Call Stack```所有的方法都执行完以后，```Event Loop```会依次把 ```Callback Queue```里的回调函数放到```Call Stack```里执行。

## Event Loop运行机制的详解

Event Loop实际上就是一个job，用来检测Call Stack和Callback Queue，一旦Call Stack里代码执行完以后，就会把Callback Queue里第一个callback函数放到Call Stack里执行。我们来看个例子：

```js
console.log('script start');

setTimeout(function() {
  console.log('setTimeout');
}, 1000);

console.log('script end');
```
运行运行结果如下:

```
script start
script end
setTimeout
```
我们具体一步一步看下整个流程：


1，代码没有运行之前，```Call Stack``` ```Callback Queue```都是空的 

![js-eventloop](/assets/images/posts/js/js-eventloop03.png){:height="80%" width="80%"}


2，把```console.log('script start')```加到Call Stack

![js-eventloop](/assets/images/posts/js/js-eventloop04.png){:height="80%" width="80%"}

3，执行```console.log('script start')```，在console里打印出```script start```，执行结束后把它移出Call Stack

![js-eventloop](/assets/images/posts/js/js-eventloop05.png){:height="80%" width="80%"}

4，把setTimeout放到Call Stack

![js-eventloop](/assets/images/posts/js/js-eventloop06.png){:height="80%" width="80%"}

5, 执行setTimeout，用setTimout线程执行timeout时间，Call Stack中setTimeout执行结束，把它移出Call Stack

![js-eventloop](/assets/images/posts/js/js-eventloop07.png){:height="80%" width="80%"}

6, 把```console.log('script end')```加到Call Stack

![js-eventloop](/assets/images/posts/js/js-eventloop08.png){:height="80%" width="80%"}

7，执行```console.log('script end')```，在console里打印出```script end```

![js-eventloop](/assets/images/posts/js/js-eventloop09.png){:height="80%" width="80%"}


8，```console.log('script end')```执行结束，把它移出Call Stack


![js-eventloop](/assets/images/posts/js/js-eventloop10.png){:height="80%" width="80%"}

9，1000毫秒以后，计时结束，把callback```cb1```函数放到Callback Queue里

![js-eventloop](/assets/images/posts/js/js-eventloop11.png){:height="80%" width="80%"}

10，此时Callback Stack是空的，Event Loop把cb1拿到Callback Stack里

![js-eventloop](/assets/images/posts/js/js-eventloop12.png){:height="80%" width="80%"}

11，执行cb1，cb1里有```console.log('setTimeout')```，把```console.log('setTimeout')```放到Call Stack里

![js-eventloop](/assets/images/posts/js/js-eventloop13.png){:height="80%" width="80%"}

12，执行```console.log('setTimeout')```，在console里打印出```setTimeout```，```console.log('setTimeout')```执行结束，把它移出Call Stack

![js-eventloop](/assets/images/posts/js/js-eventloop14.png){:height="80%" width="80%"}

13，```cb1```执行结束，把它移出Call Stack

![js-eventloop](/assets/images/posts/js/js-eventloop15.png){:height="80%" width="80%"}


总结来说就是，JS是单线程的，只有一个Call Stack，浏览器是多线程的，并且DOM事件、AJAX(XMLHttpRequest)、setTimeout都是有单独的线程处理。在这些异步事件结束，runtime会把它们的callback按顺序放到Callback Queue里，Event Loop会检测Call Stack，一旦它为空，就会把Callback Queue里的回调函数依次放到Call Stack里执行，直到Callback Queue为空。

## microtasks和macrotask的执行顺序

刚才用setTimeout为例，解释了JS中Event Loop机制是怎么运行的，也提到过runtime会把回调函数依次按时间先后顺序放到Callback Queue里，然后Event Loop再依次把这些回调函数放到Call Stack里运行。我们在浏览器Console运行以下代码，看下结果：

```js
console.log('script start');

setTimeout(function () {
    console.log('setTimeout');
}, 0);

Promise.resolve().then(function () {
    console.log('promise1');
}).then(function () {
    console.log('promise2');
});

console.log('script end');
```
执行结果如下：

```
script start
script end
promise1
promise2
setTimeout
```
<blockquote>
<p>
上述代码虽然setTimeout延时为0，其实还是异步的。因为H5标准规定setTimeout函数的第二个参数不能小于4毫秒，不足会自动增加。
</p>
</blockquote>

setTimeout和promise都是异步事件，而且setTimeout写在promise之前，为什么setTimeout的回调要比promise后执行呢？那是因为promise属于微任务（microtasks）而setTimeout属于宏任务（macrotask），微任务（microtasks）的优先级要高于宏任务（macrotask）。


整个Event Loop的执行顺序如下：
- 1：执行同步代码，这属于宏任务
- 2：执行栈（Call Stack）为空，查询是否有微任务需要执行
- 3：执行所有的微任务
- 4：必要的话渲染UI
- 5： 然后开始下一轮Event Loop，执行宏任务中的异步回调代码


Callback Queue里的回调事件称为macrotask，每次异步事件结束后，它们的回调函数会依次按时间顺序放在Callback Queue里，等待Event Loop依次把它们放到Call Stack里执行。比如：```setInterval``` ```setTimeout``` ```script``` ```setImmediate``` ```I/O``` ```UI rendering```就是macrotask。


microtasks是指异步事件结束后，回调函数不会放到Callback Queue，而是放到一个微任务队列里，在Call Stack为空时，Event Loop会先查看微任务队列里是否有任务，如果有就会先执行微任务队列里的回调事件；如果没有微任务，才会到Callback Queue执行回到事件。比如：```promise``` ```process.netTick``` ```Object.observe``` ```MutationObserver```就是microtasks。

<blockquote>
<p>
在 ES6 规范中，microtask 称为 jobs，macrotask 称为 task。
</p>
</blockquote>


我们再把代码改一下，在创建promise的时候，加一行```console.log('Promise')```，而且在第一个promise resolve的时候再加一个setTimeout，代码如下：
```js
console.log('script start');

setTimeout(function () {
    console.log('setTimeout');
}, 0);

new Promise(resolve => {
    console.log('Promise');
    resolve();
}).then(function () {
    setTimeout(function () {
        console.log('setTimeout in promise1');
    }, 0);
    console.log('promise1');
}).then(function () {
    console.log('promise2');
});

console.log('script end');
```
执行结果如下：

```
script start
Promise
script end
promise1
promise2
setTimeout
setTimeout in promise1
```

```console.log('Promise')```在这里是微任务，```console.log('script end')```是宏任务，所以```Promise```在```script end```前面，而且在微任务（microtasks）里有宏任务（macrotask），macrotask还是会依次被放到Callback Queue等待执行。


