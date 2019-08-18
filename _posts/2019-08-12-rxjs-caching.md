---
title: RxJS：如何通过RxJS实现缓存
tags: RxJS
layout: post
---

在这篇文章中会介绍以下内容：

- Angular中通过HttpClient执行Http Request返回的Observables是Cold Observable。

- HttpClient Observable每次被订阅都需要调用http request，对于公用的API返回同样的值，在不同页面重复调用会浪费http资源降低性能。

- 如何通过ReplaySubject实现缓存效果，提高性能。


## HttpClient返回的Observables是Cold Observable

在Angular2.0以上的版本，都是通过HttpClient跟后端API交互；所有的Http请求方法，比如get，post，put，delete都是返回一个Observable。在文章【[RxJS：Cold vs Hot Observables](https://limeii.github.io/2019/07/rxjs-coldhot-observable/)】里详细介绍了Cold Observables与Hot Observables的区别。


那么在Angular中通过HttpClient执行Http Request返回的Observables是Hot Observables还是Cold Observables？我们先来写段代码测试下：在页面里显示30个Github user的基本信息，测试代码中用到的API在这里：[Github API](https://developer.github.com/v3/users/#get-all-users)。


先定义一个service：RxjsCacheService，在这个service中定义一个方法去拿github的用户信息：

```ts
import { Injectable } from "@angular/core";
import { HttpClient } from "@angular/common/http";

import { User } from "../interface/rxjs-cache.interface";
import { map, catchError } from 'rxjs/operators';
import { of, Observable } from 'rxjs';

@Injectable()

export class RxjsCacheService {
    constructor(private http: HttpClient) { }

    get users() {
        return this.requestUsers();
    }

    private requestUsers() {
        // get the latest 30 github users: start from id =2
        return this.http.get<Array<User>>("https://api.github.com/users?since=1")
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

接口rxjs-cache.interface定义如下：

```ts
export interface User {
    login: string,
    id: number,
    node_id: string,
    avatar_url: string,
    gravatar_id: string,
    url: string,
    html_url: string,
    followers_url: string,
    following_url: string,
    gists_url: string,
    starred_url: string,
    subscriptions_url: string,
    organizations_url: string,
    repos_url: string,
    events_url: string,
    received_events_url: string,
    type: string,
    site_admin: boolean
}

```

然后在定义一个component：RxjsCacheComponent，代码如下：

```ts
import { Component, OnInit } from "@angular/core";

import { RxjsCacheService } from "./service/rxjs-cache.service";
import { User } from "./interface/rxjs-cache.interface";
import { Observable } from 'rxjs';


@Component({
    templateUrl: "./rxjs-cache.component.html"
})

export class RxjsCacheComponent implements OnInit {

    private users$: Observable<Array<User>>;

    constructor(private rxjsCacheService: RxjsCacheService) { }

    ngOnInit() {
        this.users$ = this.rxjsCacheService.users;
    }

}
```

rxjs-cache.component.html代码如下：

```html
<h3>here is the github user lists:</h3>

<div *ngFor="let user of users$ |async">
    <div style=" display: flex;flex-direction: row;">
        <div style="font-size: 0.9em;margin:10px;width: 10%"><strong>User Name:</strong>  { { user.login } } </div>
        <div style="font-size: 0.9em;margin:10px;width: 50%"><strong>GitHub URL:</strong> { { user.url } } </div>
    </div>
</div>
```

运行以上代码，在页面上列出了30位github用户的用户名和相应的Github地址，从上面的代码还是不能看出HttpClient返回的Observable是Hot Observables还是Cold Observables。我们把html页面代码改一下：

```html
<h3>here is the github user lists:</h3>
<div *ngFor="let user of users$ |async">
    <div style=" display: flex;flex-direction: row;">
        <div style="font-size: 0.9em;margin:10px;width: 10%"><strong>User Name:</strong>  { { user.login } } </div>
        <div style="font-size: 0.9em;margin:10px;width: 50%"><strong>GitHub URL:</strong> { { user.url } } </div>
    </div>
</div>

<h3>here is the github user lists2:</h3>
<div *ngFor="let user of users$ |async">
    <div style=" display: flex;flex-direction: row;">
        <div style="font-size: 0.9em;margin:10px;width: 10%"><strong>User Name:</strong>  { { user.login } } </div>
        <div style="font-size: 0.9em;margin:10px;width: 50%"><strong>GitHub URL:</strong> { { user.url } } </div>
    </div>
</div>
```

F12打开浏览器的DevTools，当前页面会调用两次GET API（https://api.github.com/users?since=1），并且两个user list列出的30位用户信息一模一样：

![rxjs-cache](https://limeii.github.io/assets/images/posts/rxjs/rxjs-cache01.png){:height="100%" width="100%"}

两个userlist订阅users$，生成了两个Observable实例并且都是订阅开始之后才开始发送值，也就是说**Angular中通过HttpClient执行Http Request返回的Observabl是Cold Observable**。

## 会有什么样的性能问题？

每次调用API，都会生成一个新的Observable实例，有订阅之后才开始发送值，这也符合现在前端开发要求。但是实际开发过程中，有时候后端会有提供一些公用的常量API，不同页面都需要用这些常量，按现在的调用API的方式，会导致常量API在不同的页面重复多次被调用，这种方式显然性能不好。


那可以利用RxJS实现缓存效果吗？也就是第一次调用常量API以后，后续再调用这个API不需要执行Http Request从后端服务器拿值，直接从缓存里拿值？

## 通过RxJS实现缓存效果

在文章【[RxJS：四种Subject的用法和区别](https://limeii.github.io/2019/07/rxjs-subject/)】中详细介绍了ReplaySubject，用ReplaySubject(size)可以发送之前的旧值给新的订阅者，size是定义发送具体多少个旧值给新的订阅者。那么在示例代码中可以用ReplaySubject实现缓存效果。


shareReplay这个操作符会自动创建一个ReplaySubject，一旦http request执行一次以后，就会在后续的订阅和源头Observable之间建立一个ReplaySubject，ReplaySubject是一个多播的Hot Observable，后续订阅都是从这个中间ReplaySubject拿到最后一个值，从而达到缓存效果。


我们把service：RxjsCacheService改成如下：

```ts
import { Injectable } from "@angular/core";
import { HttpClient } from "@angular/common/http";

import { User } from "../interface/rxjs-cache.interface";
import { map, catchError, shareReplay } from 'rxjs/operators';
import { of, Observable } from 'rxjs';


const CACHE_SIZE = 1;

@Injectable()

export class RxjsCacheService {

    private cacheUsers$:Observable<Array<User>>;

    constructor(private http: HttpClient) { }

    get users() {
        if(!this.cacheUsers$){
            this.cacheUsers$ = this.requestUsers()
            .pipe(
                shareReplay(CACHE_SIZE)
            );
        }
       return this.cacheUsers$;
    }

    private requestUsers() {
        return this.http.get<Array<User>>("https://api.github.com/users?since=1")
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
运行以上代码发现，页面里两个user list都是列出了相同的30位Github用户信息，但是只调用了一次GET API（https://api.github.com/users?since=1），也就是说第二订阅不是从通过后端API拿到用户信息，而是从ReplaySubject中拿到的。

整个流程如下：

![rxjs-cache](https://limeii.github.io/assets/images/posts/rxjs/rxjs-cache02.png){:height="100%" width="100%"}

页面的第一个userlist也就是第一个consumer，是通过调用API拿到30个用户信息，第二个userlist也就是第二个consumer，直接从cacheUsers$拿到这30个用户信息。cacheUsers$是ReplaySubject(1)把最后一个旧值（30个用户信息）发送给新的订阅者（第二个userlist），从而实现了缓存效果。