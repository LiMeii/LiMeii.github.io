---
title: RxJS：如何通过 RxJS 实现简单的消息通知机制
tags: RxJS
layout: post
---

在上一篇文章【[RxJS：如何通过RxJS实现缓存](https://limeii.github.io/2019/08/rxjs-caching/)】里介绍了如何在 Anuglar2+ 中结合 HttpClient 和 ReplaySubject 缓存 API Response 数据，减少重复调用 API 从而提高性能，这种方法适用于每次返回的 Response 数据都不变的常量 API。在实际开发过程中，还有一种 API 返回的值随着时间会有变化，这个时候就需要去更新 RxJS缓存里的值，从用户体检的角度出发，先在页面显示一个消息通知用户数据有更新，让用户选择是否需要更新页面内容，而不是直接在每次缓存更新以后直接刷新页面数据。接下来就来介绍如何基于 RxJS 实现简单的消息通知机制。

<blockquote>
<p>
这篇文章里用到的<font color="#0000FF"> RxJS 缓存</font>都是指在 Anuglar2+ 中结合 HttpClient 和 ReplaySubject 缓存 API Response 数据，后续新的订阅者都可以直接从 ReplaySubject 拿 API Response 数据。
</p>
</blockquote>

## 更新 RxJS 缓存

关于在什么时候要更新 RxJS 缓存，不再本文章的讨论范围内，在这里就直接每隔10秒调用一次 API，把新拿到的值赋值给 RxJS 缓存。


还是用 Github 的 get all users API：[Github API](https://developer.github.com/v3/users/#get-all-users)，每10s去调用这个 API 拿到30个不同的 github 用户信息。用 [interval](https://rxjs-cn.github.io/learn-rxjs-operators/operators/creation/interval.html) 可以实现每隔10s调用一次 API，但是会有一个问题，用户在第一次进到页面的时候需要等10s才能看到30位用户信息。我们希望用户第一次进到页面立马看到30位用户信息，之后是每隔10s调用一次API更新30位 git 用户信息，[timer](https://rxjs-cn.github.io/learn-rxjs-operators/operators/creation/timer.html)操作符可以满足这个要求，```timer(0,10000)```表示首次不用等直接调用 API 拿到30位 git 用户信息，之后每隔10s调用一次 API。Service 具体代码如下：

```ts
const CACHE_SIZE = 1;
const REFRESH_INTERVAL = 10000;
const API_ENDPOINT = "https://api.github.com/users?since=";
@Injectable()
export class RxjsNotificationService {
    private cacheUsers$: Observable<Array<User>>;
    private userStartId: number = 0;

    constructor(private http: HttpClient) { }

    get users() {
        if (!this.cacheUsers$) {
            const timer$ = timer(0, REFRESH_INTERVAL);
            this.cacheUsers$ = timer$
                .pipe(
                    switchMap(() => this.requestUsers()),
                    shareReplay(CACHE_SIZE)
                );
        }
        return this.cacheUsers$;
    }

    private requestUsers() {
        this.userStartId = this.userStartId + 30;
        return this.http.get<Array<User>>(API_ENDPOINT + this.userStartId)
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

定义一个 RxjsNotificationComponent，具体代码如下：

```ts

@Component({
    templateUrl: "./rxjs-notification.component.html"

})

export class RxjsNotificationComponent {
    users$: Observable<Array<User>>;

    constructor(private rxjsNotificationService: RxjsNotificationService) { }

    ngOnInit() {
        this.users$ = this.rxjsNotificationService.users.pipe();
    }

}
```
rxjs-notification.component.html 代码如下：

```html
<div class="container" style="margin-top:30px;width: 40%;">
    <div class="row justify-content-md-center">
        <div style="margin: 10px;" class="card w-100" *ngFor="let user of users$ |async">
            <div class="card-body">
                <h5 class="card-title"><strong>User Name:</strong>  { { user.login } } </h5>
                <p class="card-text"><strong>GitHub URL:</strong>  { { user.url } } </p>
            </div>
        </div>
    </div>
</div>
```

运行代码发现，刚进页面会调用一次 API，之后每隔10s会去调用一次 API 更新 RxJs 缓存，页面的用户信息也是每隔10秒就会更新。这个用户体验并不好，我们不希望用户在浏览页面的时候，每10s页面里的信息就自动更新了。而是希望弹出一个消息通知用户有新的用户信息，让用户选择是否需要更新页面内容。

## 基于 RxJS 的简单消息通知机制

我们先来理一下整个消息通知的流程：

![rxjs-notification](https://limeii.github.io/assets/images/posts/rxjs/rxjs-notification01.png){:height="100%" width="100%"}

在我们的例子中，页面需要显示30位 github 用户信息是数据消费者（consumer）。


当用户进到页面（0s）立马去调用 API 拿到30位git用户信息放在 RxJS 缓存里并显示在页面上，之后每隔10s都会去调用一次 API 拿到全新的30位 github 用户信息更新 RxJS 缓存里的数据但不更新页面显示的数据，此时会在页面显示一个消息提醒用户有新数据更新，如果用户点击更新按钮，提醒消息会消失同时新拿到的用户信息会更新在页面上，如果不点击更新按钮，页面列出的 github 用户信息不更新。


我们来看下动图效果：

![rxjs-notification](https://limeii.github.io/assets/images/posts/rxjs/rxjs-notification02.gif){:height="100%" width="100%"}

在前面已经把每隔10s去调用一次 API 的 service 已经写好了，我们只要在 component 和 html 页面里，把剩下的代码写完。


首先需要拿到第一次进入页面，也就是0s调用 API 拿到的30位 github 用户信息，可以通过[take(1)](https://rxjs-cn.github.io/learn-rxjs-operators/operators/filtering/take.html)操作符拿到页面首次加载的30位 github 用户信息。再定义``` updateClick$ = new Subject<void>();```用户每次点击更新按钮，会再去拿到最后一次API返回的用户信息，然后在通过 merge 操作符把两个 Observable 合并，具体代码可以如下：

```ts
    users$: Observable<Array<User>>;
    updateClick$ = new Subject<void>();

    constructor(private rxjsNotificationService: RxjsNotificationService) { }

    ngOnInit() {
        const initialUsers$ = this.getUserOnce();

        const updateUsers$ = this.updateClick$.pipe(
            mergeMap(() => this.getUserOnce())
        );

        this.users$ = merge(initialUsers$, updateUsers$);
    }

    getUserOnce() {
        return this.rxjsNotificationService.users.pipe(take(1));
    }
```
页面代码如下：

```html
<div class="container" style="margin-top:30px;width: 40%;">
    <div class="row justify-content-md-center">
        <div class="alert alert-warning w-100" role="alert">
            <strong>Warning!</strong> new user infor available, please click to update!
            <button type="button" style="margin-left: 20px;" class="btn btn-warning"
                (click)="updateClick$.next()">Update</button>
        </div>
    </div>
    <div class="row justify-content-md-center">
        <div style="margin: 10px;" class="card w-100" *ngFor="let user of users$ |async">
            <div class="card-body">
                <h5 class="card-title"><strong>User Name:</strong>  { { user.login } } </h5>
                <p class="card-text"><strong>GitHub URL:</strong>  { { user.url } } </p>
            </div>
        </div>
    </div>
</div>
```

关于显示和隐藏更新按钮，我们可以通过[mapTo](https://rxjs-cn.github.io/learn-rxjs-operators/operators/transformation/mapto.html)实现:

component 的完整代码如下：

```ts
import { Component } from "@angular/core";
import { Observable, Subject, merge } from "rxjs";

import { User } from "./interface/rxjs-notification.interface";

import { RxjsNotificationService } from "./service/rxjs-notification.service";
import { take, mergeMap, skip, mapTo } from 'rxjs/operators';

@Component({
    templateUrl: "./rxjs-notification.component.html"

})

export class RxjsNotificationComponent {
    users$: Observable<Array<User>>;
    updateClick$ = new Subject<void>();
    showNotificatoin$: Observable<boolean>;

    constructor(private rxjsNotificationService: RxjsNotificationService) { }

    ngOnInit() {
        const initialUsers$ = this.getUserOnce();

        const updateUsers$ = this.updateClick$.pipe(
            mergeMap(() => this.getUserOnce())
        );

        this.users$ = merge(initialUsers$, updateUsers$);

        const initNotification$ = this.getNotifications();
        const show$ = initNotification$.pipe(mapTo(true));
        const hide$ = this.updateClick$.pipe(mapTo(false));
        this.showNotificatoin$ = merge(show$, hide$);

    }

    getUserOnce() {
        return this.rxjsNotificationService.users.pipe(take(1));
    }

    getNotifications() {
        return this.rxjsNotificationService.users.pipe(skip(1));
    }
}
```

html 完整代码如下：

```html
<div class="container" style="margin-top:30px;width: 40%;">
    <div class="row justify-content-md-center" *ngIf="showNotificatoin$ | async">
        <div class="alert alert-warning w-100" role="alert">
            <strong>Warning!</strong> new user infor available, please click to update!
            <button type="button" style="margin-left: 20px;" class="btn btn-warning"
                (click)="updateClick$.next()">Update</button>
        </div>
    </div>
    <div class="row justify-content-md-center">
        <div style="margin: 10px;" class="card w-100" *ngFor="let user of users$ |async">
            <div class="card-body">
                <h5 class="card-title"><strong>User Name:</strong>  { { user.login } } </h5>
                <p class="card-text"><strong>GitHub URL:</strong>  { { user.url } } </p>
            </div>
        </div>
    </div>
</div>
```

service 完整代码如下：

```ts
import { Injectable } from "@angular/core";
import { HttpClient } from "@angular/common/http";

import { User } from "../interface/rxjs-notification.interface";
import { map, catchError, shareReplay, switchMap } from 'rxjs/operators';
import { of, Observable, timer } from 'rxjs';


const CACHE_SIZE = 1;
const REFRESH_INTERVAL = 10000;
const API_ENDPOINT = "https://api.github.com/users?since=";

@Injectable()

export class RxjsNotificationService {

    private cacheUsers$: Observable<Array<User>>;
    private userStartId: number = 0;

    constructor(private http: HttpClient) { }

    get users() {
        if (!this.cacheUsers$) {
            const timer$ = timer(0, REFRESH_INTERVAL);
            this.cacheUsers$ = timer$
                .pipe(
                    switchMap(() => this.requestUsers()),
                    shareReplay(CACHE_SIZE)
                );
        }
        return this.cacheUsers$;
    }

    private requestUsers() {
        this.userStartId = this.userStartId + 30;
        return this.http.get<Array<User>>(API_ENDPOINT + this.userStartId)
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