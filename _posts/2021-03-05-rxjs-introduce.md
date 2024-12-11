---
title: RxJS入门和概览
tags: RxJS
layout: post
---

RxJS 是```Reactive Extensions For JavaScript```的简写，它是一个强大的 JavaScript Reactive 编程库。Reactive 是指响应式编程（Reactive Programming）。


## 什么是响应式编程（Reactive Programming）？

任何异步事件（比如页面鼠标 click 事件），在响应式编程都是异步事件流。不仅仅是 click、hover 这种事件，任何变量、用户输入、属性、缓存、数据结构等，响应式编程把所有事物都看成是数据流。数据流是类似数组一样的序列，可以像数组一样，用 merge、map、concat 等方法操作。简单来说就是：把所有事物都事件流化，然后把这些事件流像数组一样去操作，就是响应式编程。

RxJS 提供了各种 API 来创建数据流：
- 单值：of, empty, never
- 多值：from
- 定时：interval, timer
- 从事件创建：fromEvent
- 从 Promise 创建：fromPromise
- 自定义创建：create

创建出来的数据流，是一种类似数组一样的序列，可以被订阅，也可以用如下 API 操作控制：
- 改变数据形态：map, mapTo, pluck
- 过滤一些值：filter, skip, first, last, take
- 时间轴上的操作：delay, timeout, throttle, debounce, audit, bufferTime
- 累加：reduce, scan
- 异常处理：throw, catch, retry, finally
- 条件执行：takeUntil, delayWhen, retryWhen, subscribeOn, ObserveOn
- 转接：switch
- concat，保持原来的序列顺序连接两个数据流
- merge，合并序列
- race，预设条件为其中一个数据流完成
- forkJoin，预设条件为所有数据流都完成
- zip，取各来源数据流最后一个值合并为对象
- combineLatest，取各来源数据流最后一个值合并为数组

## 异步事件流

![rxjs-data-stream](https://limeii.github.io/assets/images/posts/rxjs/rxjs-stream.png){:height="70%" width="70%"}

在上图中有如下几种东西：
- click 事件流
- 事件流产生的值
- 错误
- 事件流结束
- 时间轴

点击一个按钮事件，随着时间推移，这个点击事件会产生三个不同的结果：值，发生错误，事件完成。我们可以定义方法用来：捕获值，捕获错误，捕获点击事件结束。在这个过程中，涉及到以下几个 RxJS 的基本概念：
<blockquote>
<p>
<strong>Observable(可观察对象)</strong>：就是点击事件流。
</p>

<p>
<strong>Observers(观察者)</strong>：就是捕获值/错误/事件结束的方法（其实就是回调函数集合)。
</p>

<p>
<strong>Subscription(订阅)</strong>：Observable 产生的值都需要通过一个‘监听’把值传给 Observers，这个‘监听’就是 Subscription。
</p>

<p>
 <strong>生产者 (Producer)</strong>：就是点击事件，是事件的生产者。
</p>
</blockquote>

我们也可以用 ASCII 来描述这个事件流：

```
--a---b-c---d---X---|->

a b c d 是产生的值
X 是错误
| 是事件结束标志
---> 是时间线

```

## 为什么要有响应式编程？

我们先来看一个实际的例子：页面上有一个按钮，现在需要统计按钮点击次数。

在没有 RxJS，代码可以写成这样：

```js
var counter = 0;

document.getElementById("myBtn").addEventListener("click", function () {
    counter = counter + 1;
});

```

用 RxJS，代码如下：

```js
let button = document.getElementById("myBtn");

let clickStream$ = fromEvent(button, "click");

let counterStream$ = clickStream$.pipe(
    map((data) => { return 1 }),
    scan((acc, curr) => acc + curr, 0)
);

counterStream$.subscribe(data => {
    console.log("this is the click counter: " + data);
});
```

整个事件流可以用 ASCII 描述如下：

```
 clickStream: ---c----c--c----c------c-->
                   map(c becomes 1) 
               ---1----1--1----1------1-->
                   scan(+) 
counterStream: ---1----2--3----4------5-->
```

点击事件可以看成是数据流（clickStream），在 clickStream 事件流基础上用方法```map```把每次点击事件转化成1，然后用 ```scan```把所有的点击次数加起来，当我们执行 map 或者 scan 的时候都会在原来的数据流基础上生产一个新的数据流，原来的数据流不变。

从上面的这个例子还不能看出 RxJS 的强大和优势，我们现在再来看下：统计出双击事件次数，或者多次点击（两次或两次以上）都统计为双击次数。如果要用传统的代码实现这个需求，肯定要有很多变量来声明各种状态而且还要用到 intervals，代码逻辑复杂且容易出错。但是在响应式编程里十分简单，实际只需要四行代码！


我们先用 ASCII 把流程画一下：

```
   clickStream: ----c----c-c---c-cc---ccc---c----|->
                buff(clickStream.throttleTime(250ms))
                ----c----cc-----ccc---ccc---c----|->
                    map('get length of lists')
                ----1-----2------3-----3----1----|->
                    filter(x>=2)                  
mulClickStream: ----------2------3-----3---------|->

```

具体代码如下：

```ts
let button = document.getElementById("myBtn");

let clickStream$ = fromEvent(button, "click");

let doubleClickStream$ = clickStream$
    .pipe(
        buffer(clickStream$.pipe(throttleTime(250))),
        map(click => { return click.length }),
        filter(num => num >= 2)
    )

doubleClickStream$.subscribe(data => {
    console.log("the number of double click is: " + data);;
});
```

从上面这个简单的例子，我们可以看到 RxJS 提供大量的操作符，处理不同的业务需求，短短几行代码可以涵盖很复杂的代码逻辑，在前端交互非常复杂的系统中，客户端都是基于事件编程的，对事件处理非常多，用 RxJS 比较有优势。当然响应式编程不仅仅是在 JS 里存在，它还支持各种语言，比如：RxJava、Rx.NET、RxPY、RxGo 等等。