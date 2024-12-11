---
title: RxJS：所有订阅都需要调用 unsubscribe 取消订阅？
tags: RxJS
layout: post
---

最开始用 RxJS 的时候，看官方文档对 Subscription 的介绍如下：

<blockquote>
<p>
What is a Subscription? A Subscription is an object that represents a disposable resource, usually the execution of an Observable. A Subscription has one important method,unsubscribe, that takes no argument and just disposes the resource held by the subscription
</p>
</blockquote>

从此用 RxJS 订阅的时候，时刻都不忘调用 unsubscribe() 以防内存泄漏。对于结束 Observable，释放内存的方式有三种方式：

- 第一种，Observable 完成值的发送，执行 Observable.onComplete()

- 第二种，Observable 发生错误，执行 Observable.OnError()

- 第三种，订阅者主动取消订阅，执行 subscription.unsubscribe()

对于```Observable.onComplete()```和```Observable.OnError()```，RxJS 自身会处理这两种情况，所以不需要在代码里再手动取消订阅释放内存。对于第三种方式，Observable 还在源源不断的发送值，订阅者想主动取消订阅，那就需要在代码里调用```unsubscribe()```取消订阅释放内存。


**那么显然在代码中，并不是所有的 Observable 需要手动调用 unsubscribe() 取消订阅。**


在 Angular 项目中，常用到的订阅以及是否需要调用 unsubscribe() 取消订阅，有以下几种：

- Angular 中通过 HttpClient 执行 Http Request 返回的 Observables，订阅这些 Observables 拿到 API 返回的数据，不需要调用 unsubscribe() 取消订阅。

- Angular AsyncPipe，不需要调用 unsubscribe()取消订阅。

- 通过 Subject，BehaviorSubject，AsyncSubject，ReplaySubject 在各个 Component 之间通信，需要调用 unsubscribe()取消订阅。

- RxJS 自带的一些操作符：take，takeWhile，first 等等，不需要调用 unsubscribe()取消订阅。

接下来，详细解释下以上几种订阅以及为什么有的需要手动调用 unsubscribe()取消订阅，而有些不需要。

## Angular 中通过 HttpClient 执行 Http Request 返回的 Observables

不需要调用 unsubscribe()取消订阅的原因有以下两点：

- Angular HttpClient 源码中，在 Http Response 结束时，如果 Request 成功会调用 responseObserver.complete()方法结束当前的 HttpResponse Observable，如果 Request 失败会调用 responseObserver.error(response)结束当前的 HttpResponse Observable。

- Angular 中通过 HttpClient 执行 Http Request 返回的 Observables 是 Cold Observable 并且只发送一个值，Cold Observable 在值发送完成以后，RxJS 会执行 OnCompleted 方法，表示这个 Observable 已经结束了，自动释放资源。


**对于第一个原因：Aangular 源码会处理完成的 Http Response Observables。**我们可以直接看 Angular 的源码，源码地址：[/packages/http/src/backends/xhr_backend.ts](https://github.com/angular/angular/blob/master/packages/http/src/backends/xhr_backend.ts)，关键代码如下：

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
通过源码我们能看到：在 Http Response 结束时，如果 Request 成功会调用```responseObserver.complete()```，如果 Request 失败会调用```responseObserver.error(response)```，complete()/error() 方法会结束当前 responseObserver。


**对于第二个原因：Angular 中通过 HttpClient 执行 Http Request 返回的 Observables 是 Cold Observable 并且只发送一个值，Cold Observable 在值发送完成以后，RxJS 会执行 OnCompleted 方法，自动释放资源。**我们可以通过代码来验证。


对于 HttpClient 返回的 Observables，在 component 中有两种订阅方式：

- 第一种方式是直接在 component 中显示订阅 Http Response Observable 拿到 API 返回的数据。

- 第二种是通过 Angular AsyncPipe。

我们先来看第一种订阅方式，AsyncPipe 的方式稍后再介绍。在 component 中显示订阅 Http Response Observable 拿到 API 返回的数据,我们来看下具体代码：


定义一个 service，调用 Github 的 get all users API：[Github API](https://developer.github.com/v3/users/#get-all-users)，拿到30位 github 用户的信息。

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
再定义一个 component，显示这30位 github 用户的信息：

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

对应的 rxjs-unsubscribe.component.html 如下：

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

在 console 里会有：getuser http request has been complete! 表明 Http Response Observable 会执行 onComplete 方法，结束当前的 Observable。

## Angular AsyncPipe

在 Angular 官方文档里，关于【[AsyncPipe](https://angular.io/api/common/AsyncPipe)】介绍的章节里，明确写了在离开页面销毁 component 的时候，会自动销毁 AsyncPipe 订阅的 Observables。

![rxjs-unsubscribe](https://limeii.github.io/assets/images/posts/rxjs/rxjs-unsubscribe02.png){:height="100%" width="100%"}

我们也可以在 Git 上看 AsyncPipe 的源码，源码地址：[/packages/common/src/pipes/async_pipe.ts ](https://github.com/angular/angular/blob/master/packages/common/src/pipes/async_pipe.ts)，关键代码如下：

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
在 ngOnDestroy 方法中，会执行```this._dispose()```把 AsyncPipe 的订阅销毁。

## 通过四种 Subject 在各个 Component 之间通信

在 component 之间通信，我们会用到 Subject，BehaviorSubject，AsyncSubject，ReplaySubject 这四种 subject，它们都是Hot Observable，Hot Observable 不管有没有被订阅都会源源不断的发送值。如果订阅者要主动取消订阅，就必须手动调用unsubscribe() 取消订阅。在 Angular component 有个钩子函数：ngOnDestroy，在 commponet 被销毁之前执行，所以一般都是把 Subscription 的 unsubscribe 放在这个函数里执行，代码如下：

```ts
    ngOnDestroy() {
        this.subscription.unsubscribe();
    }
```

## RxJS 自带的一些操作符：take，takeWhile，first

在用这些操作符，比如 take(1)，拿到 Observable 发送的第一个值之后，RxJs 会主动的停止当前的 Observable，也就是销毁当前 Observable，并不需要手动再调用 unsubscribe()取消订阅。