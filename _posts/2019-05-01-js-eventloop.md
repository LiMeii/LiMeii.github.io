---
title: JS：详解Event Loop运行机制
tags: JS
layout: post
---

在这篇文章中会介绍以下内容：
- engine runtime 和 call stack 简介（以 V8 引擎为例）

- Event Loop 运行机制的详解

- microtasks 和 macrotask 的执行顺序

## engine runtime 和 call stack 简介

在 chrome 浏览器和 nodejs 里都是用 V8 引擎解析和运行 JS 代码，我们先来看下 V8 引擎的简化图：

![js-eventloop](/assets/images/posts/js/js-eventloop01.png){:height="80%" width="80%"}

上图中 Heap 是用来做内存分配，```Call Stack```是用来执行 JS 代码，由于 JS 是单线程所以只有一个```Call Stack```。实际我们写网页开发的时候，除了一些 JS 代码，我们还会大量用到：DOM 事件、AJAX(XMLHttpRequest)、setTimeout 等等一些异步事件。从上图可以看出，这些异步事件都没有在 V8 引擎里，事实上这些异步事件不属于 V8 引擎，而是属于浏览器，并且 DOM 事件、AJAX(XMLHttpRequest)、setTimeout 都分别有单独的线程来处理。由于```Call Stack```执行（JS 运行线程）和页面渲染线程是互斥的，如果所有的事情都由 V8 引擎处理，这样肯定会导致页面卡顿。


浏览器多线程和 callback 机制完美避免了页面卡顿的问题。DOM 事件、AJAX(XMLHttpRequest)、setTimeout 这些异步事件在各自单独的线程处理完以后，每个异步事件都有 callback 回调函数，V8 引擎再把这些回调函数放在```Call Stack```执行。上述整个运行机制可以称为是 runtime，可以简化如下图：

![js-eventloop](/assets/images/posts/js/js-eventloop02.png){:height="80%" width="80%"}

如上图所示，web 异步事件结束以后，会有 callback，然后 runtime 把这些 callback 事件放到```Callback Queue```里，一旦```Call Stack```所有的方法都执行完以后，```Event Loop```会依次把 ```Callback Queue```里的回调函数放到```Call Stack```里执行。

## Event Loop 运行机制的详解

Event Loop 实际上就是一个 job，用来检测 Call Stack 和 Callback Queue，一旦 Call Stack 里代码执行完以后，就会把 Callback Queue 里第一个 callback 函数放到 Call Stack 里执行。我们来看个例子：

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

![js-eventloop](/assets/images/posts/js/js-eventloop03.png){:height="60%" width="60%"}


2，把```console.log('script start')```加到 Call Stack

![js-eventloop](/assets/images/posts/js/js-eventloop04.png){:height="60%" width="60%"}

3，执行```console.log('script start')```，在 console 里打印出```script start```，执行结束后把它移出 Call Stack

![js-eventloop](/assets/images/posts/js/js-eventloop05.png){:height="60%" width="60%"}

4，把 setTimeout 放到 Call Stack

![js-eventloop](/assets/images/posts/js/js-eventloop06.png){:height="60%" width="60%"}

5, 执行 setTimeout，用 setTimout 线程执行 timeout 时间，Call Stack 中 setTimeout 执行结束，把它移出 Call Stack

![js-eventloop](/assets/images/posts/js/js-eventloop07.png){:height="60%" width="60%"}

6, 把```console.log('script end')```加到 Call Stack

![js-eventloop](/assets/images/posts/js/js-eventloop08.png){:height="60%" width="60%"}

7，执行```console.log('script end')```，在 console 里打印出```script end```

![js-eventloop](/assets/images/posts/js/js-eventloop09.png){:height="60%" width="60%"}


8，```console.log('script end')```执行结束，把它移出 Call Stack


![js-eventloop](/assets/images/posts/js/js-eventloop10.png){:height="60%" width="60%"}

9，1000毫秒以后，计时结束，把 callback```cb1```函数放到 Callback Queue 里

![js-eventloop](/assets/images/posts/js/js-eventloop11.png){:height="60%" width="60%"}

10，此时 Callback Stack 是空的，Event Loop 把 cb1 拿到 Callback Stack 里

![js-eventloop](/assets/images/posts/js/js-eventloop12.png){:height="60%" width="60%"}

11，执行 cb1，cb1 里有```console.log('setTimeout')```，把```console.log('setTimeout')```放到 Call Stack 里

![js-eventloop](/assets/images/posts/js/js-eventloop13.png){:height="60%" width="60%"}

12，执行```console.log('setTimeout')```，在 console 里打印出```setTimeout```，```console.log('setTimeout')```执行结束，把它移出 Call Stack

![js-eventloop](/assets/images/posts/js/js-eventloop14.png){:height="60%" width="60%"}

13，```cb1```执行结束，把它移出 Call Stack

![js-eventloop](/assets/images/posts/js/js-eventloop15.png){:height="60%" width="60%"}


总结来说就是，JS 是单线程的，只有一个 Call Stack，浏览器是多线程的，并且 DOM 事件、AJAX(XMLHttpRequest)、setTimeout 都是有单独的线程处理。在这些异步事件结束，runtime会把它们的 callback 按顺序放到 Callback Queue 里，Event Loop 会检测 Call Stack，一旦它为空，就会把 Callback Queue 里的回调函数依次放到 Call Stack 里执行，直到 Callback Queue 为空。

## microtasks 和 macrotask 的执行顺序

刚才用 setTimeout 为例，解释了JS中 Event Loop 机制是怎么运行的，也提到过 runtime 会把回调函数依次按时间先后顺序放到 Callback Queue 里，然后 Event Loop 再依次把这些回调函数放到 Call Stack 里运行。我们在浏览器 Console 运行以下代码，看下结果：

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
上述代码虽然 setTimeout 延时为0，其实还是异步的。因为H5标准规定 setTimeout 函数的第二个参数不能小于4毫秒，不足会自动增加。
</p>
</blockquote>

setTimeout 和 promise 都是异步事件，而且setTimeout 写在 promise 之前，为什么 setTimeout 的回调要比 promise 后执行呢？那是因为 promise 属于微任务（microtasks）而 setTimeout 属于宏任务（macrotask），微任务（microtasks）的优先级要高于宏任务（macrotask）。


首先我们需要明白以下几件事情：
- JS 分为同步任务和异步任务
- 同步任务都在主线程上执行，形成一个执行栈
- 主线程之外，事件触发线程管理着一个任务队列，只要异步任务有了运行结果，就在任务队列之中放置一个事件。
- 一旦执行栈中的所有同步任务执行完毕（此时JS引擎空闲），系统就会读取任务队列，将可运行的异步任务添加到可执行栈中，开始执行。

根据规范，事件循环是通过任务队列的机制来进行协调的。一个 Event Loop 中，可以有一个或者多个任务队列(task queue)，一个任务队列便是一系列有序任务(task)的集合；每个任务都有一个任务源(task source)，源自同一个任务源的 task 必须放到同一个任务队列，从不同源来的则被添加到不同队列。 setTimeout/Promise 等 API 便是任务源，而进入任务队列的是他们指定的具体执行任务。

![js-eventloop](/assets/images/posts/js/js-eventloop16.png)

Callback Queue（Task Queue）里的回调事件称为宏任务（macrotask），每次异步事件结束后，它们的回调函数会依次按时间顺序放在 Callback Queue 里，等待 Event Loop 依次把它们放到 Call Stack 里执行。比如：```setInterval``` ```setTimeout``` ```script``` ```setImmediate``` ```I/O``` ```UI rendering```就是宏任务（macrotask）。


微任务（microtasks）是指异步事件结束后，回调函数不会放到 Callback Queue，而是放到一个微任务队列里（Microtasks Queue），在 Call Stack 为空时，Event Loop 会先查看微任务队列里是否有任务，如果有就会先执行微任务队列里的回调事件；如果没有微任务，才会到 Callback Queue 执行回到事件。比如：```promise``` ```process.netTick``` ```Object.observe``` ```MutationObserver```就是微任务（microtasks）。

<blockquote>
<p>
在 ES6 规范中，microtask 称为 jobs，macrotask 称为 task。
</p>
</blockquote>

整个 Event Loop 的执行顺序如下：
- 执行一个宏任务（栈中没有就从事件队列中获取）
- 执行过程中如果遇到微任务，就将它添加到微任务的任务队列中
- 宏任务执行完毕后，立即执行当前微任务队列中的所有微任务（依次执行）
- 当前宏任务执行完毕，开始检查渲染，然后GUI线程接管渲染
- 渲染完毕后，JS线程继续接管，开始下一个宏任务（从事件队列中获取，也就是 callbacke queue）

流程图如下：
![js-eventloop](/assets/images/posts/js/js-eventloop17.jpg){:height="40%" width="40%"}



我们再把代码改一下，在创建 promise 的时候，加一行```console.log('Promise')```，而且在第一个 promise resolve 的时候再加一个 setTimeout，代码如下：
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

/**
script start
Promise
script end
promise1
promise2
setTimeout
setTimeout in promise1
**/
```

```console.log('Promise')```在这里是同步代码，```console.log('script end')```是同步代码且放在最后，所以```Promise```在```script end```前面，而且在微任务（microtasks）里有宏任务（macrotask），macrotask 还是会依次被放到 Callback Queue 等待执行。


如果有 async  await 呢？再来看一段代码：
```js
//请写出输出内容
async function async1() {
    console.log('async1 start');
    await async2();
    console.log('async1 end');
}
async function async2() {
	console.log('async2');
}

console.log('script start');

setTimeout(function() {
    console.log('setTimeout');
}, 0)

async1();

new Promise(function(resolve) {
    console.log('promise1');
    resolve();
}).then(function() {
    console.log('promise2');
});
console.log('script end');

/**
script start
async1 start
async2
promise1
script end
async1 end
promise2
setTimeout
**/
```

我们知道 Promise 中的异步体现在 then 和 catch 中，所以写在 Promise 中的代码是被当做同步任务立即执行的。而在 async/await 中，在出现 await 出现之前，其中的代码也是立即执行的。那么出现了 await 时候发生了什么呢？

由于因为 async await 本身就是 promise+generator 的语法糖。所以 await 后面的代码是 microtask。所以对于上面代码中的
```js
async function async1() {
	console.log('async1 start');
	await async2();
	console.log('async1 end');
}
```
等价于：
```js
async function async1() {
	console.log('async1 start');
	Promise.resolve(async2()).then(() => {
                console.log('async1 end');
        })
}
```

我们来看一个变式, 将 async2 中的函数也变成了 Promise 函数：
```js
async function async1() {
    console.log('async1 start');
    await async2();
    console.log('async1 end');
}
async function async2() {
    //async2做出如下更改：
    new Promise(function(resolve) {
    console.log('promise1');
    resolve();
}).then(function() {
    console.log('promise2');
    });
}
console.log('script start');

setTimeout(function() {
    console.log('setTimeout');
}, 0)
async1();

new Promise(function(resolve) {
    console.log('promise3');
    resolve();
}).then(function() {
    console.log('promise4');
});

console.log('script end');

/**
script start
async1 start
promise1
promise3
script end
promise2
async1 end
promise4
setTimeout
**/
```

我们再来看一个变式，将 async1 中 await 后面的代码和 async2 的代码都改为异步的，代码如下：
```js
async function async1() {
    console.log('async1 start');
    await async2();
    //更改如下：
    setTimeout(function() {
        console.log('setTimeout1')
    },0)
}
async function async2() {
    //更改如下：
	setTimeout(function() {
		console.log('setTimeout2')
	},0)
}
console.log('script start');

setTimeout(function() {
    console.log('setTimeout3');
}, 0)
async1();

new Promise(function(resolve) {
    console.log('promise1');
    resolve();
}).then(function() {
    console.log('promise2');
});
console.log('script end');

/**
script start
async1 start
promise1
script end
promise2
setTimeout3
setTimeout2
setTimeout1
**/
```

我们再来看一个变式，代码如下：
```js
async function a1 () {
    console.log('a1 start')
    await a2()
    console.log('a1 end')
}
async function a2 () {
    console.log('a2')
}

console.log('script start')

setTimeout(() => {
    console.log('setTimeout')
}, 0)

Promise.resolve().then(() => {
    console.log('promise1')
})

a1()

let promise2 = new Promise((resolve) => {
    resolve('promise2.then')
    console.log('promise2')
})

promise2.then((res) => {
    console.log(res)
    Promise.resolve().then(() => {
        console.log('promise3')
    })
})
console.log('script end')


/**
script start
a1 start
a2
promise2
script end
promise1
a1 end
promise2.then
promise3
setTimeout
**/
```

参考资料：
- [“Event loops”, section in HTML5 spec.](https://html.spec.whatwg.org/multipage/webappapis.html#event-loops)
- [“Help, I’m stuck in an event-loop” by Philip Roberts (video).](https://vimeo.com/96425312)