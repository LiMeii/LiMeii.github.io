---
title: RxJS：Cold vs Hot Observables
tags: RxJS
layout: post
---

RxJS中Observables分为两种：Cold Observables和Hot Observables，这两个到底有什么区别呢？我们先来看下[RxJS官方](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/gettingstarted/creating.md#cold-vs-hot-observables)给出的解释：
<blockquote>
<p>
Cold observables start running upon subscription, i.e., the observable sequence only starts pushing values to the observers when Subscribe is called. (…) This is different from hot observables such as mouse move events or stock tickers which are already producing values even before a subscription is active.
</p>
</blockquote>

翻译过来就是：
- Cold Observables只有被observers订阅的时候，才会开始产生值。是单播的，有多少个订阅就会生成多少个订阅实例，每个订阅都是从第一个产生的值开始接收值，所以每个订阅接收到的值都是一样的。

- Hot Observables不管有没有被订阅都会产生值。是多播的，多个订阅共享同一个实例，是从订阅开始接受到值，每个订阅接收到的值是不同的，取决于它们是从什么时候开始订阅。

单看文字解释，还是不太能理解，那自己先动手写几个Observables看看。具体代码在这里：[angular-rxjs](https://github.com/LiMeii/angular-rxjs/tree/master/src/app/modules/rxjs-cold-hot)

## Cold Observables

```ts
let obs$ = from([1, 2, 3, 4, 5]);

obs$.subscribe(data => { console.log("1st subscriber:" + data) });
obs$.subscribe(data => { console.log("2nd subscriber:" + data) });
```
把一个数组转换成observables，然后分别用不同的Subscription订阅它，在console里结果如下：

```
1st subscriber:1
1st subscriber:2
1st subscriber:3
1st subscriber:4
1st subscriber:5
2nd subscriber:1
2nd subscriber:2
2nd subscriber:3
2nd subscriber:4
2nd subscriber:5
```
我们把代码改一下，让第二个Subscription延迟1秒订阅：

```ts
let obs$ = from([1, 2, 3, 4, 5]);

obs$.subscribe(data => { console.log("1st subscriber:" + data) });
setTimeout(() => {
    obs$.subscribe(data => { console.log("2nd subscriber:" + data) });
}, 1100);
```
结果还是跟上面一样，没有任何区别。这个是Cold Observables，两个订阅分别有两个订阅实例，是单播的，在没有开始订阅之前，obs$是不发送值，一旦开始订阅，不管是从什么开始，obs$都是从第一个值开始发送值，所以两个订阅接收到值没有区别。

## 把Cold Observables 转换成Hot Observables

在RxJS中有一个publish操作符，根据官方文档的解释，publish()可以生成一个ConnectableObservable。ConnectableObservable是个共享源Observables实例，可以被多个Subscription订阅，达到多播的效果。但是publish()只是生成了一个共享源Observables，通过它建立了多个订阅和source Observables之间的关联，还不能真正被订阅，需要通过调用ConnectableObservable.prototype.connect方法真正实现被多个Subscription订阅。具体代码如下：

```
let obs$ = from([1, 2, 3, 4, 5]).pipe(
    publish()
) as ConnectableObservable<any>;
obs$.connect();

obs$.subscribe(data => { console.log("1st subscriber:" + data) });
setTimeout(() => {
    obs$.subscribe(data => { console.log("2st subscriber:" + data) });
}, 2100);
```
期待的效果是第一个订阅接收到1，2，3，4，5；第二个订阅接收到3，4，5。实际上两个订阅都没有接收任何值！想想其实也挺正常的，因为Hot Observables是不管有没有被订阅，都会发送值，我们数组里有五个值，等到开始订阅的时候，这五个值都已经发送完了，所以在订阅里接收不到任何值了。


为了看到效果，我们把数组替换成interval，每隔一秒就发送一个值，第一个订阅是等了两秒开始接受值，第二个订阅是等了三秒开始接受值，代码如下：

```
let obs$ = interval(1000).pipe(
    publish()
) as ConnectableObservable<any>;
obs$.connect();
setTimeout(() => {
    obs$.subscribe(data => { console.log("1st subscriber:" + data) });
    setTimeout(() => {
        obs$.subscribe(data => { console.log("2st subscriber:" + data) });
    }, 1100);
}, 2100);
```

效果是，第一个订阅从2开始接受值，第二个订阅从3开始接受值：

```
1st subscriber:2
1st subscriber:3
2st subscriber:3
1st subscriber:4
2st subscriber:4
1st subscriber:5
2st subscriber:5
1st subscriber:6
2st subscriber:6

....
```