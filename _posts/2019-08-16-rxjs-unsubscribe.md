---
title: RxJS：所有订阅都需要调用unsubscribe取消订阅？
tags: RxJS
layout: post
---

最开始用RxJS的时候，看官方文档对Subscription的介绍如下：

<blockquote>
<p>
What is a Subscription? A Subscription is an object that represents a disposable resource, usually the execution of an Observable. A Subscription has one important method,unsubscribe, that takes no argument and just disposes the resource held by the subscription
</p>
</blockquote>

从此用RxJS订阅的时候，时刻都不忘调用unsubscribe()以防内存泄漏。对于结束Observable，释放内存的方式有三种方式：

- 第一种，Observable完成值的发送，执行Observable.onComplete()

- 第二种，Observable发生错误，执行Observable.OnError()

- 第三种，订阅者主动取消订阅，执行subscription.unsubscribe()

对于```Observable.onComplete()```和```Observable.OnError()```，RxJS自身会处理这两种情况，所以不需要在代码里再手动取消订阅释放内存。对于第三种方式，Observable还在源源不断的发送值，订阅者想主动取消订阅，那就需要在代码里调用```unsubscribe()```取消订阅释放内存。


**那么显然在代码中，并不是所有的Observable都需要手动调用unsubscribe()取消订阅。**


在Angular项目中，常用到的订阅以及是否需要调用unsubscribe()取消订阅，有以下几种：

- Angular中通过HttpClient执行Http Request返回的Observables，订阅这些Observables拿到API返回的数据，不需要调用unsubscribe()取消订阅。

- Angular AsyncPipe，不需要调用unsubscribe()取消订阅。

- 通过Subject，BehaviorSubject，AsyncSubject，ReplaySubject在各个Component之间通信，需要调用unsubscribe()取消订阅。

- RxJS自带的一些操作符：take，takeWhile，first等等，不需要调用unsubscribe()取消订阅。

接下来，详细解释下以上几种订阅以及为什么有的需要手动调用unsubscribe()取消订阅，而有些不需要。

## Angular中通过HttpClient执行Http Request返回的Observables

不需要调用unsubscribe()取消订阅的原因有以下两点：

- Angular HttpClient源码中，在Http Response结束时，如果Request成功会调用responseObserver.complete()方法结束当前的HttpResponse Observable，如果Request失败会调用responseObserver.error(response)结束当前的HttpResponse Observable。

- Angular中通过HttpClient执行Http Request返回的Observables是Cold Observable并且只发送一个值，Cold Observable在值发送完成以后，RxJS会执行OnCompleted方法，表示这个Observable已经结束了，自动释放资源。


**对于第一个原因：Aangular源码会处理完成的Http Response Observables。**我们可以直接看Angular的源码，源码地址：[/packages/http/src/backends/xhr_backend.ts](https://github.com/angular/angular/blob/master/packages/http/src/backends/xhr_backend.ts)，关键代码如下：

```ts
// this Angular http source code, code relative path: angular/packages/http/src/backends/xhr_backend.ts 
export class XHRConnection implements Connection {
  constructor(req: Request, browserXHR: BrowserXhr, baseResponseOptions?: ResponseOptions) {
    this.request = req;
    this.response = new Observable<Response>((responseObserver: Observer<Response>) => {
       ......
       ......
      const onLoad = () => {
        const response = new Response(responseOptions);
        response.ok = isSuccess(status);
        if (response.ok) {
          responseObserver.next(response);
          // response sucess, then call complete to end current responseObserver
          responseObserver.complete();
          return;
        }
        responseObserver.error(response);
      };

      const onError = (err: ErrorEvent) => {
       ......
       ......
        // response error, then call error to end current responseObserver
        responseObserver.error(new Response(responseOptions));
      };
  }
}
```
通过源码我们能看到：在Http Response结束时，如果Request成功会调用responseObserver.complete()，如果Request失败会调用responseObserver.error(response)，complete()/error() 方法会结束当前responseObserver。


**对于第二个原因：Angular中通过HttpClient执行Http Request返回的Observables是Cold Observable并且只发送一个值，Cold Observable在值发送完成以后，RxJS会执行OnCompleted方法，自动释放资源。**我们可以通过代码来验证。


对于HttpClient返回的Observables，在component中有两种订阅方式：

- 第一种方式是直接在component中显示订阅Http Response Observable拿到API返回的数据。

- 第二种是通过Angular AsyncPipe。

我们先来看第一种订阅方式，AsyncPipe的方式稍后再介绍。在component中显示订阅Http Response Observable拿到API返回的数据,我们来看下具体代码：


定义一个service，调用Github的get all users API：[Github API](https://developer.github.com/v3/users/#get-all-users)，拿到30位github用户的信息。

```ts
export class RxjsUnsubscribeService {

    constructor(private http: HttpClient) { }

    get users() {
        return this.requestUsers();
    }

    private requestUsers() {
        return this.http.get<Array<User>>("https://api.github.com/users?since=35")
            .pipe(
                map(respone => respone),
                catchError(error => {
                    console.log("something went wrong " + error)
                    return of([]);
                })
            )
    }
}
```
再定义一个component，显示这30位github用户的信息：

```ts
@Component({
    templateUrl: "./rxjs-unsubscribe.component.html"

})
export class RxjsUnsubscribeComponent implements OnInit {

    users: Array<User> = [];
    constructor(private rxjsUnsubscribeService: RxjsUnsubscribeService) { }

    ngOnInit() {
        this.rxjsUnsubscribeService.users
            .subscribe(
                (data) => {
                    this.users = data;
                },
                (error) => {
                    console.log("something went wrong: " + error);
                },
                () => {
                    console.log("getuser http request has been complete!");
                }
            );
    }
}

```

对应的rxjs-unsubscribe.component.html如下：

```ts
<h3>here is the github user lists:</h3>

<div *ngFor="let user of users">
    <div style=" display: flex;flex-direction: row;">
        <div style="font-size: 0.9em;margin:10px;width: 10%"><strong>User Name:</strong>  { { user.login } } </div>
        <div style="font-size: 0.9em;margin:10px;width: 50%"><strong>GitHub URL:</strong> { { user.url } } </div>
    </div>
</div>
```

运行以上代码，结果如下：

![rxjs-unsubscribe](https://limeii.github.io/assets/images/posts/rxjs/rxjs-unsubscribe01.png){:height="100%" width="100%"}

在console里会有：getuser http request has been complete! 表明Http Response Observable会执行onComplete方法，结束当前的Observable。

## Angular AsyncPipe

在Angular官方文档里，关于[AsyncPipe](https://angular.io/api/common/AsyncPipe)介绍的章节里，明确写了在离开页面销毁component的时候，会自动销毁AsyncPipe订阅的Observables。

![rxjs-unsubscribe](https://limeii.github.io/assets/images/posts/rxjs/rxjs-unsubscribe02.png){:height="100%" width="100%"}

我们也可以在Git上看AsyncPipe的源码，源码地址：[/packages/common/src/pipes/async_pipe.ts ](https://github.com/angular/angular/blob/master/packages/common/src/pipes/async_pipe.ts)，关键代码如下：

```ts
......
......
@Injectable()
@Pipe({name: 'async', pure: false})
export class AsyncPipe implements OnDestroy, PipeTransform {

  ......
  private _subscription: SubscriptionLike|Promise<any>|null = null;
  ......
  constructor(private _ref: ChangeDetectorRef) {}

  ngOnDestroy(): void {
    if (this._subscription) {
      this._dispose();
    }
  }
  ......
  ......
```
在ngOnDestroy方法中，会执行```this._dispose()```把AsyncPipe的订阅销毁。

## 通过四种Subject在各个Component之间通信

在component之间通信，我们会用到Subject，BehaviorSubject，AsyncSubject，ReplaySubject这四种subject，它们都是Hot Observable，Hot Observable不管有没有被订阅都会源源不断的发送值。如果订阅者要主动取消订阅，就必须手动调用unsubscribe()取消订阅。在Angular component有个钩子函数：ngOnDestroy，在commponet被销毁之前执行，所以一般都是把Subscription的unsubscribe放在这个函数里执行，代码如下：

```ts
    ngOnDestroy() {
        this.subscription.unsubscribe();
    }
```

## RxJS自带的一些操作符：take，takeWhile，first

在用这些操作符，比如take(1)，拿到Observable发送的第一个值之后，RxJs会主动的停止当前的Observable，也就是销毁当前Observable，并不需要手动再调用unsubscribe()取消订阅。