---
title: RxJS：四种Subject的用法和区别
tags: RxJS
layout: post
---

在RxJS中有四种Subject分别是：Subject，BehaviorSubject，AsyncSubject，ReplaySubject；这四种Subject都是特殊的Observable。


在介绍它们之前，我们先来看下这四种Subject与普通Observable的区别：

Observable | Subject | BehaviorSubject | AsyncSubject | ReplaySubject
:-------: | :-------: | :-------:  | :-------:  | :-------: 
Cold | Hot | Hot | Hot | Hot
数据生成者 | 数据生成者和消费者 | 数据生成者和消费者 | 数据生成者和消费者  | 数据生成者和消费者
单播 | 多播 | 多播 | 多播 |  多播
每次从头开始把值发给观察者| 将值多播给已注册监听该 Subject 的观察者们 | 把最后一个值（当前值）发送给观察者（需要初始值） |  执行的最后一个值发送给观察者相当于last()  |  可以把之前的值发送给观察者（错过的值）

## Cold vs Hot

在文章【[RxJS：Cold vs Hot Observables](https://limeii.github.io/2019/07/rxjs-coldhot-observable/)】里详细介绍了Cold Observables与Hot Observables的区别，除了Observable之外，这四种Subject都是Hot Observables。

## 数据生产者 vs 数据消费者

- **数据生产者**是指Observable(可观察对象)，产生数据的一方。


- **数据消费者**是指Observers(观察者)，接收数据的一方。


普通的Observable只是数据生产者，发送数据。而Subject，BehaviorSubject，AsyncSubject和ReplaySubject即是生产者又是消费者。比如在angular项目中，我们经常用Subject在多个component中共享数据：

```ts
// messageService.ts
private shareData: Subject<string> = new Subject<string>();

sendData(value: string) {
    this.shareData.next(value);
}
getData(): Observable<string> {
    return this.shareData.asObservable();
}

```

```ts
// RxjsSubjectBComponent
// send data from RxjsSubjectBComponent
  this.messageService.sendData("this message from subject b");
```

```ts
// RxjsSubjectCComponent
// get data in RxjsSubjectCComponent
this.subscription = this.messageService.getData()
    .subscribe(val => {
        this.message = val;
        console.log(val);
    });
```
在这两个component中通过定义在messageService的Subject共享数据，通过next()方法使得Subject是数据生产者（可观察对象），通过asObservable()使得Subject是数据消费者（Observers）。

## 单播 vs 多播

在上面的代码例子中，不仅是在component c可以订阅拿到数据，在其他component中也可以订阅拿到数据。比如在RxjsSubjectAComponent中定义如下代码：

```ts
// RxjsSubjectAComponent
// get data in RxjsSubjectAComponent
this.subscription = this.messageService.getData()
    .subscribe(val => {
        console.log("this is in componet A and " + val);
    })
```
从RxjsSubjectBComponent发出值，在RxjsSubjectCComponent和RxjsSubjectAComponent能拿到同样的值，这个就是多播。


普通的Observables是Cold Observables并且是单播的，其他四种Subject是Hot Observables并且是多播的。


单播的意思是，每个普通的Observables实例都只能被一个观察者订阅，当它被其他观察者订阅的时候会产生一个新的实例。也就是普通Observables被不同的观察者订阅的时候，会有多个实例，不管观察者是从何时开始订阅，每个实例都是从头开始把值发给对应的观察者。

## BehaviorSubject

Subject其中的一个变体就是BehaviorSubject，它有一个“当前值”的概念。它保存了发送给消费者的最新值，当有新的观察者订阅时，会立即从BehaviorSubject那接收到“当前值”，在定义一个BehaviorSubject时需要有初始值。


在messageService定义一个BehaviorSubject，初始值是1，updateBehaviorSubject方法把值更新为2，代码如下：
```ts
//messageService.ts
private behaviorSubject: BehaviorSubject<number> = new BehaviorSubject<number>(1);
currentBehaviorSubject = this.behaviorSubject.asObservable();
updateBehaviorSubject() {
    this.behaviorSubject.next(2);
}
```
并在RxjsSubjectBComponent中调用执行updateBehaviorSubject方法，代码如下：
```ts
//RxjsSubjectBComponent
this.messageService.updateBehaviorSubject();
```
在RxjsSubjectAComponent中订阅behaviorSubject的值，代码如下：

```ts
let subscription1 = this.messageService.currentBehaviorSubject
    .subscribe(val => {
        console.log("this is in RxjsSubjectAComponent and Current behavior subject value is  " + val)
    });

this.subscription.add(subscription1);
```

在RxjsSubjectCComponent中，等5秒以后在订阅behaviorSubject的值，代码如下：

```ts
setTimeout(() => {
    let subscription1 = this.messageService.currentBehaviorSubject
        .subscribe(val => {
             console.log("this is in RxjsSubjectCComponent and Current behavior subject value is  " + val)
        });

}, 5000);

```

运行效果：在RxjsSubjectAComponent中会拿到1和2两个值，在RxjsSubjectCComponent中会拿到2这个值。

```
this is in RxjsSubjectAComponent and Current behavior subject value is  1
this is in RxjsSubjectAComponent and Current behavior subject value is  2
this is in RxjsSubjectCComponent and Current behavior subject value is  2
```


如果是正常的Hot Observable在5秒之后，拿不到任何值，因为在五秒内已经把值1和2都发送完了，五秒以后观察者再订阅，没有任何值可以发送了。但是用BehaviorSubject保存了发送给消费者的最新值，当有新的观察者订阅时，会立即从BehaviorSubject那接收到“当前值”，所以RxjsSubjectCComponent的订阅者会拿到2这个值。

## ReplaySubject

类似于BehaviorSubject，可以发送旧值给新的订阅者，但是不仅是‘当前值’，还可以是之前的旧值。

在messageService定义一个ReplaySubject，为新的订阅者缓冲3个值，sendReplaySubject方法把1 2 3 4 5五个值发给订阅者，代码如下：

```ts
//messageService.ts
private replaySubject: ReplaySubject<number> = new ReplaySubject<number>(3);
sendReplaySubject() {
    this.replaySubject.next(1);
    this.replaySubject.next(2);
    this.replaySubject.next(3);
    this.replaySubject.next(4);
    this.replaySubject.next(5);
}

getReplaySubject(): Observable<any> {
    return this.replaySubject.asObservable();
}

```

在RxjsSubjectBComponent中，调用sendReplaySubject方法：

```ts
this.messageService.sendReplaySubject();
```

在RxjsSubjectAComponent中，订阅replaySubject的值，代码如下：

```ts
let subscription2 = this.messageService.getReplaySubject()
    .subscribe(val => {
        console.log("this is in RxjsSubjectAComponent and Current Replay subject value is  " + val)
    });

this.subscription.add(subscription2);
```

在RxjsSubjectCComponent中，等5秒以后在订阅replaySubject的值，代码如下：

```ts
setTimeout(() => {
    let subscription2 = this.messageService.getReplaySubject()
        .subscribe(val => {
            console.log("this is in RxjsSubjectCComponent and Current repaly subject value is  " + val)
        });
    this.subscription.add(subscription2);

}, 5000);
```

效果如下：

```
this is in RxjsSubjectAComponent and Current Replay subject value is  1
this is in RxjsSubjectAComponent and Current Replay subject value is  2
this is in RxjsSubjectAComponent and Current Replay subject value is  3
this is in RxjsSubjectAComponent and Current Replay subject value is  4
this is in RxjsSubjectAComponent and Current Replay subject value is  5

this is in RxjsSubjectCComponent and Current repaly subject value is  3
this is in RxjsSubjectCComponent and Current repaly subject value is  4
this is in RxjsSubjectCComponent and Current repaly subject value is  5

```

## AsyncSubject

只有当 Observable 执行完成时(执行 complete())，它才会将执行的最后一个值发送给观察者。


在messageService定义一个AsyncSubject，sendAsyncSubject方法把10001 10002 10003 三个值发给订阅者，代码如下：

```ts
private asyncSubject: AsyncSubject<number> = new AsyncSubject<number>();

sendAsyncSubject() {
    this.asyncSubject.next(10001);
    this.asyncSubject.next(10002);
    this.asyncSubject.next(10003);
    this.asyncSubject.complete();
}
getAsyncSubject() {
    return this.asyncSubject.asObservable();
}
```


在RxjsSubjectBComponent中，调用sendAsyncSubject方法：

```ts
this.messageService.sendAsyncSubject();
```

在RxjsSubjectAComponent中，订阅asyncSubject的值，代码如下：

```ts
let subscription3 = this.messageService.getAsyncSubject()
    .subscribe(val => {
        console.log("this is in RxjsSubjectAComponent and Current Async subject value is  " + val)
    });

this.subscription.add(subscription3);
```

在RxjsSubjectCComponent中，等5秒以后在订阅asyncSubject的值，代码如下：

```ts
setTimeout(() => {
    let subscription3 = this.messageService.getAsyncSubject()
        .subscribe(val => {
            console.log("this is in RxjsSubjectCComponent and Current async subject value is  " + val)
        });
    this.subscription.add(subscription3);

}, 5000);
```

效果如下：

```
this is in RxjsSubjectAComponent and Current Async subject value is  10003
this is in RxjsSubjectCComponent and Current async subject value is  10003
```