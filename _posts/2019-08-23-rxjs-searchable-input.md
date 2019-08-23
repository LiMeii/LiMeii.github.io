---
title: RxJS：如何用RxJS实现高效的搜索功能
tags: RxJS
layout: post
---

在项目中，经常会碰到这样的需求：用户在输入框输入数据，需要实时调用后端API，拿到结果显示在页面上。如果用传统方式一般实现方式是在输入框上绑定一个keydown或者keyup事件，然后每次输入值以后都调用一次后端API，拿到返回数据。这样会有一个问题，比如我输入'limei'，```l``` ```li``` ```lim``` ```lime``` ```limei```这五次keydown/keyup分别会调用一次API。这五个API有五个Response，我最后想要'limei'的结果，由于这五个API的response顺序不可控，可能最后返回'li'的结果。这种方法不仅效率低而且结果正确性没办法保证。


这篇文章会介绍如何在RxJS中结合操作符```debounceTime``` ```map```  ```filter```  ```distinctUntilChanged``` 和```switchMap```实现：输入完“limei”之后只调用一次API，最后拿到这个API返回的输入显示在页面上。


我们来实现一个搜索github用户的功能，页面上有一个输入框，在输入框中输入github用户名，然后把搜索结果显示在页面上：

![rxjs-searchable-input](https://limeii.github.io/assets/images/posts/rxjs/rxjs-searchinput01.png){:height="100%" width="100%"}

如果没有匹配的用户，显示如下：

![rxjs-searchable-input](https://limeii.github.io/assets/images/posts/rxjs/rxjs-searchinput02.png){:height="100%" width="100%"}

我们先来定义一个Service如下：


示例代码用的是Angular+RxJS，API是【[Github Search user API](https://developer.github.com/v3/search/#search-users)】, 示例代码在这里：【[angular-rxjs](https://github.com/LiMeii/angular-rxjs)】

```ts
export class RxjsSearchableInputService {

    constructor(private http: HttpClient) { }

    searchUser(val: any) {
        return this.http.get<Observable<SearchResult>>("https://api.github.com/search/users?q=" + val)
            .pipe(
                map(response => response),
                catchError((error) => {
                    console.log("something went wrong, " + error);
                    return of([]);
                })
            )
    }
}
```

定义一个component如下：

```ts
export class RxjsSearchableInputComponent implements OnInit, OnDestroy {

    users: Array<User> = [];
    onSearchUser$ = new Subject<KeyboardEvent>();

    validSearch$: Observable<any>;
    emptySearch$: Observable<any>;

    subscription: Subscription;
    constructor(private rxjsSearchableInputService: RxjsSearchableInputService) { }

    ngOnInit() {
        this.validSearch$ = this.onSearchUser$
            .pipe(
                debounceTime(1000),
                map(event => (<HTMLInputElement>event.target).value),
                filter(input => input !== ""),
                distinctUntilChanged(),
                switchMap(data => this.rxjsSearchableInputService.searchUser(data))
            )

        this.emptySearch$ = this.onSearchUser$.pipe(
            debounceTime(1000),
            map(event => (<HTMLInputElement>event.target).value),
            filter(input => input === ""),
            switchMap(data => of([]))
        )

        this.subscription = merge(this.validSearch$, this.emptySearch$)
            .subscribe(resp => {
                if (resp && resp.items && resp.items.length) {
                    let result = resp as SearchResult;
                    this.users = result.items;
                } else {
                    this.users = [];
                }
            })
    }

    ngOnDestroy() {
        this.subscription.unsubscribe();
    }
}
```
component对应的html文件如下：

```html
<div class="margin-large">
    <div class="container-flex margin-small">
        <div class="margin-right-small"><strong> Search Github user: </strong></div>
        <input type="text" (keyup)="onSearchUser$.next($event)" />
    </div>
    <div *ngFor="let user of users" style="margin: 20px">
        <div class="container-flex">
            <div style="font: 0.9em;width:20%"><strong>User Name: </strong>{{user.login}}</div>
            <div style="font:0.9em;width:10%"><strong>ID: </strong>{{user.id}}</div>
            <div style="font:0.9em;width:30%"><strong>GitHub URL: </strong>{{user.url}}</div>
        </div>
    </div>
    <div *ngIf="!users.length" class="margin-small">
        <label style="color:red">no search result!</label>
    </div>
</div>
```
在componet中先定义一个onSearchUser$ subject，然后在input上绑定一个keyup事件```(keyup)="onSearchUser$.next($event)"```，每次输入框有输入变化的时候，onSearchUser$都会发送当前输入框的值。


然后在component中定义两个Observable：validSearch$和emptySearch$，validSearch$是每隔1秒拿到input框中的非空值，并且是本次拿到的值和上次的值不一样的情况下调用searchUser API把搜索结合显示在页面上。emptySearch$是每隔1秒拿到input框中的空值，并不调用API，直接返回一个空的用户列表。在validSearch$中用switchMap的原因是：本次调用API的时候，上一次的API如果还没有返回，switchMap会取消上一次的API，这样就可以保证每次API返回的结果是正确的。


最后再用merge把emptySearch$，validSearch$两个Observable合并。


用RxJS实现这个功能，代码是不是非常简洁！