---
title: Angular：生命周期和钩子函数
tags: Angular
layout: post
---


Angular中每个component/directive都有它自己的生命周期。包括创建组件，渲染组件，创建渲染子组件，检测绑定属性变化，回收和从DOM中移除。
<blockquote>
<p>
生命周期有这几种：OnChanges，OnInit，DoCheck，AfterContentInit，AfterContentChecked，AfterViewInit，AfterViewChecked，OnDestroy。
</p>
<p>钩子函数就是在对应的生命周期前面加上前缀ng，比如OnInit，对应的钩子函数是ngOnInit()。</p>
</blockquote>

下图列出了constructor和钩子函数，以及它们的执行顺序：

![angular-lifecycle-hooks](https://limeii.github.io/assets/images/posts/angular/angular-lifecycle-hooks.png){:height=“60%" width="60%"}

1. **ngOnChanges：** 当前component/directive的@Input/@Output绑定的值发生变化的时候会触发这个函数，在ngOnInit之前执行。需要注意的是：如果@Input是个对象，对象里面的数据改变但是引用没有变化是不会触发这个函数。

2. **ngOnInit：**在第一个ngOnChanges之后触发，只执行一次。这个函数用来初始化页面内容。

3. **ngDoCheck：**第一种情况是：只要有任何change detection（比如click handlers, http requests, route changes...）都会执行ngDoCheck；第二种情况是：状态发生变化，angular自己又不能捕获这个变化会触发ngDoCheck。在用这个函数的时候要特别小心，里面的代码也尽量的精简，一般建议在开发debug的时候用这个函数。关于ngDoCheck触发时机详细讲解可以参考这篇文章[Angular:ngDoCheck执行时机](https://limeii.github.io/2019/06/angular-ngdocheck-onpush-strategy/)

4. **ngAfterContentInit：**页面有用ng-content进行组件内容投射，在初始化的时候会执行一次这个函数。

5. **ngAfterContentChecked：**在每次检查投射内容的时候执行，ngDoCheck调用之后都会触发这个函数。

6. **ngAfterViewInit：**component的页面或者是它的子页面初始的时候会执行一次这个函数。

7. **ngAfterViewChecked：**在每次检查compoent页面或者它的子页面的时候执行，ngDoCheck调用之后都会触发这个函数。

8. **ngOnDestroy：**在commponet被销毁之前执行。

在使用钩子函数的时候需要注意以下几点：

1. **constructor vs ngOnInit：**在constructor里并不是所有数据都已经存在，比如@ContentChildren/@ContentChild/@ViewChildren/@ViewChild/@Input在执行constructor的时候并不存在，相关代码最好都放在ngOnInit。

2. **ngOnChanges vs ngDoCheck：**ngOnChanges是在@Input的值发生变化时触发；而ngDoCheck在每次change detection的时候都会触发或者是在状态发生变化angular自己又不能捕获时被触发。在用ngDoCheck的时候要非常小心，ngDoCheck被触发的频率非常高，代码尽量精简，避免导致页面性能问题。

3. **ngAfterContentInit vs ngAfterViewInit：**跟ng-content相关的就用ngAfterContentInit，当前component或者它的child componet相关的就用ngAfterViewInit。

4. **ngAfterContentChecked vs ngAfterViewChecked：** 这两个函数也是每次都在change detection会被执行，用的时候也需要小心，避免页面性能问题。

5. **ngOnChanges vs ngAfterViewChecked：** 可以复用的Component考虑ngOnChanges，不可以复用的component考虑ngAfterViewChecked。
