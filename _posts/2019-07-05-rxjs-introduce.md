---
title: RxJS入门和概览
tags: RxJS
layout: post
---

RxJS是```Reactive Extensions For JavaScript```的简写，它是一个强大的JavaScript Reactive编程库。我们提到RxJS的时候一般会想到：

- Reactive Programming
- Asynchronous data streams
- Observers
- Observable
- Lodash for events

## 什么是响应式编程（Reactive Programming）？

任何异步事件，比如页面鼠标click事件，实际上都是异步事件流，我们可以检测到这些事件流，并且可以在这些事件流上加一些我们自己的操作代码。响应式编程就是基于这种思想，把所有事物都看成是数据流，不仅仅是click hover这种事件，任何变量、用户输入、属性、缓存、数据结构等，在响应式编程里这些都是数据流。


在响应式编程里，事物不仅是数据流，而且可以像操作JS的数组一样，可以merge、concat、race、filter、last、take等等。


我们来看下，在响应式编程里，事件流是什么样子的：

![rxjs-data-stream](https://limeii.github.io/assets/images/posts/rxjs/rxjs-stream.png){:height="70%" width="70%"}

点击一个按钮事件，随着时间推移，这个点击事件会产生三个不同的结果：值，发生错误，事件完成。我们可以定义方法来捕获这三种结果：捕获值，捕获错误，捕获点击事件结束。这个监听事件就是subscribing， observers是捕获值/错误/事件结束的方法，点击事件流是observable（或者subject）。


我们也可以用ASCII来描述这个事件流：

```
--a---b-c---d---X---|->

a b c d 是产生的值
X 是错误
| 是事件结束标志
---> 是时间线

```

我们来看一个实际的例子：页面上有一个按钮，现在需要统计按钮点击次数。

在没有RxJS，代码可以写成这样：

```js
var counter = 0;

document.getElementById("myBtn").addEventListener("click", function () {
    counter = counter + 1;
});

```

用RxJS，代码如下：

```js
import { fromEvent } from "rxjs";
import { map, scan } from 'rxjs/operators';

const button = document.getElementById("myBtn");

const clickStream = fromEvent(button, "click");

const counterStream = clickStream.pipe(
    map((data) => { return 1 }),
    scan((acc, curr) => acc + curr, 0)
);

counterStream.subscribe(data => {
    console.log('this is the click counter: ' + data);
});
```

整个事件流可以用ASCII描述如下：

```
 clickStream: ---c----c--c----c------c-->
               vvvvv map(c becomes 1) vvvv
               ---1----1--1----1------1-->
               vvvvvvvvv scan(+) vvvvvvvvv
counterStream: ---1----2--3----4------5-->
```

点击事件可以看成是数据流（clickStream），在clickStream事件流基础上用方法```map```把每次点击事件转化成1，然后用 ```scan```把所有的点击次数加起来，当我们执行map或者scan的时候都会在原来的数据流基础上生产一个新的数据流，原来的数据流不变。




