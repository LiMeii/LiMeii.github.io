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
结果还是跟上面一样，没有任何区别。这个是Cold Observables，两个订阅分别有两个订阅实例，是单播的，在没有开始订阅之前，obs$是不发送值，一旦开始订阅，不管是从什么时候开始，obs$都是从第一个值开始发送值，所以两个订阅接收到值没有区别。


## Hot Observables

我先把前面Cold Observables改成Hot Observables，代码如下，先忽略publish，ConnectableObservable，connect，稍后再详细解释。

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
根据Hot Observables的定义，期待的效果是第一个订阅接收到1，2，3，4，5；第二个订阅接收到3，4，5。实际上两个订阅都没有接收任何值！仔细想想其实也是正常的，因为Hot Observables是不管有没有被订阅，都会发送值，我们数组里有五个值，等到开始订阅的时候，这五个值都已经发送完了，所以在订阅里接收不到任何值了。


为了看到效果，我们把数组替换成interval，每隔一秒就发送一个值，第一个订阅是等了一秒开始接受值，第二个订阅是等了两秒开始接受值，代码如下：

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

- 我们可以把Cold Observables理解为在手机网易云音乐APP上听歌：APP里的歌曲资源是Cold Observables，听歌的人是observers。如果没有人打开APP去播放这首歌，这首歌不会自己播放。每个人从自己APP打开播放这首歌的时候，先把歌曲从网上下载到本地，每份都是单独的歌曲实例，都是从头开始听，互相不会影响。

- 而Hot Observables可以理解为演唱会：比如我们去看一场演唱会，没有迟到的小伙伴（A）可以从第一首歌开始听，迟到的小伙伴就（B）只能从第二首或者更晚的歌开始听；演唱会就是Hot Observables，小伙伴A和B就是observers。小伙伴A和B共享同一个演唱会实例，是从订阅开始接受到值，每个订阅接收到的值是不同的，取决于它们是从什么时候开始订阅。

### 那怎么创建Hot Observables呢？

在上面的Hot Observables代码里，用到了publish操作符，ConnectableObservable，以及connect()方法创建Hot Observables；我们来看下它们到底是什么意思。

<blockquote>
<p>
<strong>publish：</strong>这个操作符把正常的Observable（Cold Observables ）转换成ConnectableObservable。
</p>

<p>
<strong>ConnectableObservable：</strong>ConnectableObservable是多播的共享Observable，可以同时被多个observers共享订阅，它是Hot Observables。ConnectableObservable是订阅者和真正的源头Observables（上面例子中的interval，每隔一秒发送一个值，就是源头Observables）的中间人，ConnectableObservable从源头Observables接收到值然后再把值转发给订阅者。
</p>

<p>
<strong>connect()：</strong>ConnectableObservable并不会主动发送值，它有个connect方法，通过调用connect方法，可以启动共享ConnectableObservable发送值。当我们调用ConnectableObservable.prototype.connect方法，不管有没有被订阅，都会发送值。订阅者共享同一个实例，订阅者接收到的值取决于它们何时开始订阅。在我们的例子中，第一个订阅等了一秒从2开始接受值，第二个订阅等了两秒从3开始接受值。
</p>
</blockquote>


## connect vs refCount

除了connect，还有一个refCount方法，在比较这两个区别之前，我们先来看下refCount的用法和效果：

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
按之间对publish会生成一个ConnectableObservable，它是一个Hot Observables，那么预期结果是：第一个订阅者是从2开始接收值，第二个订阅者是从3开始接受值？


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


当使用refCount，是引用计数的observable。它表示当第一个订阅者开始订阅的时候，开始发送和产生值；第二个订阅者（之后的订阅者）共享第一个订阅者的Observables实例，没有订阅者的时候，会自动取消订阅；之后再重新订阅，又从头开始发送值。


它不是Hot Observables也不是Cold Observables，因为它是从有第一个订阅者的时候才开始发送值，没有订阅者的时候会自动取消订阅，而且之后的订阅者共享第一个订阅者的Observables实例。它是基于Hot Observables与Cold Observables之间的Observables，可以理解为Warm Observables。

## share

publish和refCount可以生成一个Warm Observables，实际单独使用share操作符可以达到同样的效果，share实际就是publish().refCount()的简写，这次把第二个订阅比第一个订阅晚五秒再开始订阅，具体代码如下：
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
和使用publish().refCount()效果完全一样。

## shareReplay
在用share()的时候，第二个或者更后面的订阅者开始订阅的时候，都是共享第一订阅者的Observables，比如在上面的例子中，第二个订阅比第一个订阅晚五秒再开始订阅，那么第二个订阅者从5开始接收值。但是实际情况中，如果我想让第二个订阅者也能拿到前面的值，那怎么办呢？用shareReplay()可以实现。具体代码如下：

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
 shareReplay(1)中的1表示拿到错过的前一个值，在我们的例子就是第二个订阅从4开始接受值。如果改成shareReplay(2)就表示从错过的前两个值开始接受值，也就是第二个订阅会从3开始接受值。


 shareReplay(1)其实也是publishReplay(1).refCount()的简写，用publishReplay(1).refCount()有同样的效果。

## 总结

 在这篇文章中，我们介绍了：
 - 什么是Cold Observables和Hot Observables，以及两者的区别
 - 怎么创建Hot Observables
 - 什么是Warm Observables，以及怎么用shareReplay()去拿到订阅者错过的值