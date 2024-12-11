---
title: Angular：生命周期和钩子函数
tags: Angular
layout: post
---


Angular 中每个 component/directive 都有它自己的生命周期。包括创建组件，渲染组件，创建渲染子组件，检测绑定属性变化，回收和从 DOM 中移除。
<blockquote>
<p>
生命周期有这几种：OnChanges，OnInit，DoCheck，AfterContentInit，AfterContentChecked，AfterViewInit，AfterViewChecked，OnDestroy。
</p>
<p>钩子函数就是在对应的生命周期前面加上前缀 ng，比如 OnInit，对应的钩子函数是 ngOnInit()。</p>
</blockquote>

下图列出了 constructor 和钩子函数，以及它们的执行顺序：

![angular-lifecycle-hooks](https://limeii.github.io/assets/images/posts/angular/angular-lifecycle-hooks.png){:height=“60%" width="60%"}

**ngOnChanges**

当前 component/directive 的 @Input/@Output 绑定的值发生变化的时候会触发这个函数，在 ngOnInit 之前执行。需要注意的是：如果 @Input 是个对象，对象里面的数据改变但是引用没有变化是不会触发这个函数。

**ngOnInit**

在第一个 ngOnChanges 之后触发，只执行一次。这个函数用来初始化页面内容。

**ngDoCheck**

第一种情况是：只要有任何 change detection（比如click handlers, http requests, route changes...）都会执行 ngDoCheck；第二种情况是：状态发生变化，Angular 自己又不能捕获这个变化会触发 ngDoCheck。在用这个函数的时候要特别小心，里面的代码也尽量的精简，一般建议在开发 debug 的时候用这个函数。关于 ngDoCheck 触发时机详细讲解可以参考这篇文章【[Angular:ngDoCheck执行时机](https://limeii.github.io/2019/06/angular-ngdocheck-onpush-strategy/)】

**ngAfterContentInit**

页面有用 ng-content 进行组件内容投射，在初始化的时候会执行一次这个函数。

**ngAfterContentChecked**

在每次检查投射内容的时候执行，ngDoCheck 调用之后都会触发这个函数。

**ngAfterViewInit**

component 的页面或者是它的子页面初始的时候会执行一次这个函数。

7. **ngAfterViewChecked：**在每次检查 compoent 页面或者它的子页面的时候执行，ngDoCheck 调用之后都会触发这个函数。

8. **ngOnDestroy：**在 commponet 被销毁之前执行。

### 在使用钩子函数的时候需要注意以下几点

**constructor vs ngOnInit：**

在 constructor 里并不是所有数据都已经存在，比如```@ContentChildren``` ```@ContentChild``` ```@ViewChildren``` ```@ViewChild``` ```@Input```在执行 constructor 的时候并不存在，相关代码最好都放在 ngOnInit。

**ngOnChanges vs ngDoCheck：**

ngOnChanges 是在 @Input 的值发生变化时触发；而 ngDoCheck 在每次 change detection 的时候都会触发或者是在状态发生变化 Angular 自己又不能捕获时被触发。在用 ngDoCheck 的时候要非常小心，ngDoCheck 被触发的频率非常高，代码尽量精简，避免导致页面性能问题。

**ngAfterContentInit vs ngAfterViewInit：**

跟 ng-content 相关的就用 ngAfterContentInit，当前 component 或者它的 child componet 相关的就用 ngAfterViewInit。

**ngAfterContentChecked vs ngAfterViewChecked：**

这两个函数也是每次都在 change detection 会被执行，用的时候也需要小心，避免页面性能问题。

**ngOnChanges vs ngAfterViewChecked：**

可以复用的 Component 考虑 ngOnChanges，不可以复用的 component 考虑 ngAfterViewChecked。
