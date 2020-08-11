---
title: RxJS：Cold vs Hot Observables
tags: RxJS
layout: post
---

RxJS 中 Observables 分为两种：Cold Observables 和 Hot Observables，这两个到底有什么区别呢？我们先来看下【[RxJS官方](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/gettingstarted/creating.md#cold-vs-hot-observables)】给出的解释：
<blockquote>
<p>
Cold observables start running upon subscription, i.e., the observable sequence only starts pushing values to the observers when Subscribe is called. (…) This is different from hot observables such as mouse move events or stock tickers which are already producing values even before a subscription is active.
</p>
</blockquote>

翻译过来就是：
- Cold Observables 只有被 observers 订阅的时候，才会开始产生值。是单播的，有多少个订阅就会生成多少个订阅实例，每个订阅都是从第一个产生的值开始接收值，所以每个订阅接收到的值都是一样的。

- Hot Observables 不管有没有被订阅都会产生值。是多播的，多个订阅共享同一个实例，是从订阅开始接受到值，每个订阅接收到的值是不同的，取决于它们是从什么时候开始订阅。

单看文字解释，还是不太能理解，那自己先动手写几个 Observables 看看。具体代码在这里：【[angular-rxjs](https://github.com/LiMeii/angular-rxjs/tree/master/src/app/modules/rxjs-cold-hot)】

## Cold Observables

```ts
let obs$ = from([1, 2, 3, 4, 5]);

obs$.subscribe(data => { console.log("1st subscriber:" + data) });
obs$.subscribe(data => { console.log("2nd subscriber:" + data) });
```
把一个数组转换成 observables，然后分别用不同的 Subscription 订阅它，在 console 里结果如下：

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
我们把代码改一下，让第二个 Subscription 延迟1秒订阅：

```ts
let obs$ = from([1, 2, 3, 4, 5]);

obs$.subscribe(data => { console.log("1st subscriber:" + data) });
setTimeout(() => {
    obs$.subscribe(data => { console.log("2nd subscriber:" + data) });
}, 1100);
```
结果还是跟上面一样，没有任何区别。这个是 Cold Observables，两个订阅分别有两个订阅实例，是单播的，在没有开始订阅之前，obs$ 是不发送值，一旦开始订阅，不管是从什么时候开始，obs$ 都是从第一个值开始发送值，所以两个订阅接收到值没有区别。


## Hot Observables

我先把前面 Cold Observables 改成 Hot Observables，代码如下，先忽略 publish，ConnectableObservable，connect，稍后再详细解释。

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
根据 Hot Observables 的定义，期待的效果是第一个订阅接收到1，2，3，4，5；第二个订阅接收到3，4，5。实际上两个订阅都没有接收任何值！仔细想想其实也是正常的，因为 Hot Observables 是不管有没有被订阅，都会发送值，我们数组里有五个值，等到开始订阅的时候，这五个值都已经发送完了，所以在订阅里接收不到任何值了。


为了看到效果，我们把数组替换成 interval，每隔一秒就发送一个值，第一个订阅是等了一秒开始接受值，第二个订阅是等了两秒开始接受值，代码如下：

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

......
```

### Cold Observables　vs　Hot Observables

- 我们可以把 Cold Observables 理解为在手机网易云音乐 APP 上听歌：APP 里的歌曲资源是 Cold Observables，听歌的人是 observers。如果没有人打开 APP 去播放这首歌，这首歌不会自己播放。每个人从自己 APP 打开播放这首歌的时候，先把歌曲从网上下载到本地，每份都是单独的歌曲实例，都是从头开始听，互相不会影响。

- 而 Hot Observables 可以理解为演唱会：比如我们去看一场演唱会，没有迟到的小伙伴（A）可以从第一首歌开始听，迟到的小伙伴就（B）只能从第二首或者更晚的歌开始听；演唱会就是 Hot Observables，小伙伴 A 和 B 就是 observers。小伙伴 A 和 B 共享同一个演唱会实例，是从订阅开始接受到值，每个订阅接收到的值是不同的，取决于它们是从什么时候开始订阅。

### 那怎么创建 Hot Observables 呢？

在上面的 Hot Observables 代码里，用到了 publish 操作符，ConnectableObservable，以及 connect() 方法创建 Hot Observables；我们来看下它们到底是什么意思。

<blockquote>
<p>
<strong>publish：</strong>这个操作符把正常的 Observable（Cold Observables ）转换成 ConnectableObservable。
</p>

<p>
<strong>ConnectableObservable：</strong>ConnectableObservable 是多播的共享 Observable，可以同时被多个 observers共享订阅，它是 Hot Observables。ConnectableObservable 是订阅者和真正的源头 Observables（上面例子中的 interval，每隔一秒发送一个值，就是源头 Observables）的中间人，ConnectableObservable 从源头 Observables 接收到值然后再把值转发给订阅者。
</p>

<p>
<strong>connect()：</strong>ConnectableObservable 并不会主动发送值，它有个 connect 方法，通过调用 connect 方法，可以启动共享 ConnectableObservable 发送值。当我们调用 ConnectableObservable.prototype.connect 方法，不管有没有被订阅，都会发送值。订阅者共享同一个实例，订阅者接收到的值取决于它们何时开始订阅。在我们的例子中，第一个订阅等了一秒从2开始接受值，第二个订阅等了两秒从3开始接受值。
</p>
</blockquote>


## connect vs refCount

除了 connect，还有一个 refCount 方法，在比较这两个区别之前，我们先来看下 refCount 的用法和效果：

```ts
let obs$ = interval(1000).pipe(
    publish(),
    refCount()
)

setTimeout(() => {
    obs$.subscribe(data => { console.log("1st subscriber:" + data) });
    setTimeout(() => {
        obs$.subscribe(data => { console.log("2st subscriber:" + data) });

    }, 1100);

}, 2000);
```
按之间对 publish 会生成一个 ConnectableObservable，它是一个 Hot Observables，那么预期结果是：第一个订阅者是从2开始接收值，第二个订阅者是从3开始接受值？


我们来看下实际效果：

```
1st subscriber:0
1st subscriber:1
2st subscriber:1
1st subscriber:2
2st subscriber:2
1st subscriber:3
2st subscriber:3
1st subscriber:4
2st subscriber:4
1st subscriber:5
2st subscriber:5

......
```
跟预想的不一样，第一个订阅者从0开始接收值，第二个订阅者比第一个订阅者晚了一秒，从1开始接受值。


当使用 refCount，是引用计数的 observable。它表示当第一个订阅者开始订阅的时候，开始发送和产生值；第二个订阅者（之后的订阅者）共享第一个订阅者的 Observables 实例，没有订阅者的时候，会自动取消订阅；之后再重新订阅，又从头开始发送值。


它不是 Hot Observables 也不是 Cold Observables，因为它是从有第一个订阅者的时候才开始发送值，没有订阅者的时候会自动取消订阅，而且之后的订阅者共享第一个订阅者的 Observables 实例。它是基于 Hot Observables 与 Cold Observables 之间的 Observables，可以理解为 Warm Observables。

## share

publish 和 refCount 可以生成一个 Warm Observables，实际单独使用 share 操作符可以达到同样的效果，share 实际就是 publish().refCount() 的简写，这次把第二个订阅比第一个订阅晚五秒再开始订阅，具体代码如下：
```ts
let obs$ = interval(1000).pipe(
    share()
)

setTimeout(() => {
    obs$.subscribe(data => { console.log("1st subscriber:" + data) });
    setTimeout(() => {
        obs$.subscribe(data => { console.log("2st subscriber:" + data) });

    }, 5100);

}, 2000);
```
输出结果：
```
1st subscriber:0
1st subscriber:1
1st subscriber:2
1st subscriber:3
1st subscriber:4
1st subscriber:5
2st subscriber:5
1st subscriber:6
2st subscriber:6
1st subscriber:7
2st subscriber:7

......
```
和使用 publish().refCount() 效果完全一样。

## shareReplay
在用 share() 的时候，第二个或者更后面的订阅者开始订阅的时候，都是共享第一订阅者的 Observables，比如在上面的例子中，第二个订阅比第一个订阅晚五秒再开始订阅，那么第二个订阅者从5开始接收值。但是实际情况中，如果我想让第二个订阅者也能拿到前面的值，那怎么办呢？用 shareReplay() 可以实现。具体代码如下：

```ts
let obs$ = interval(1000).pipe(
    shareReplay(1)
)

setTimeout(() => {
    obs$.subscribe(data => { console.log("1st subscriber:" + data) });
    setTimeout(() => {
        obs$.subscribe(data => { console.log("2st subscriber:" + data) });

    }, 5100);

}, 2000);
```

输出结果：
```
1st subscriber:0
1st subscriber:1
1st subscriber:2
1st subscriber:3
1st subscriber:4
2st subscriber:4
1st subscriber:5
2st subscriber:5
1st subscriber:6
2st subscriber:6
1st subscriber:7
2st subscriber:7

......
```
 shareReplay(1) 中的1表示拿到错过的前一个值，在我们的例子就是第二个订阅从4开始接受值。如果改成 shareReplay(2 )就表示从错过的前两个值开始接受值，也就是第二个订阅会从3开始接受值。


 shareReplay(1) 其实也是```publishReplay(1).refCount()```的简写，用```publishReplay(1).refCount()```有同样的效果。

## 总结

 在这篇文章中，我们介绍了：
 - 什么是 Cold Observables 和 Hot Observables，以及两者的区别
 - 怎么创建 Hot Observables
 - 什么是 Warm Observables，以及怎么用 shareReplay() 去拿到订阅者错过的值