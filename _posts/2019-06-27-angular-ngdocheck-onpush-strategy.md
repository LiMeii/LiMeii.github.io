---
title: Angular：ngDoCheck执行时机
tags: Angular
layout: post
---

在 [Angular官方文档](https://angular.io/guide/lifecycle-hooks) 对```ngDoCheck```执行时机是这么描述的：

<blockquote>
<p>Detect and act upon changes that Angular can't or won't detect on its own.</p>
<p>Called during every change detection run, immediately after ngOnChanges() and ngOnInit().</p>
</blockquote>

直译过来就是：

 1. 在状态发生变化，Angular 自己本身不能捕获这个变化时会触发```NgDoCheck```。

 2. 每次变化检测以后，都会触发```ngDoCheck```钩子函数，紧跟在```ngOnChanges```和```ngOnInit```之后运行。

第二点很好理解，第一点到底是什么意思呢？

在【[Angular Change Detection:变化检测策略](https://limeii.github.io/2019/06/angular-changeDetectionStrategy-OnPush/)】这篇文章中解释过：某一个组件中设置 Angular 变化检测策略为 OnPush，如果没有以下四种情况，Angular 是不会为这个组件或者它的子组件执行变化检测。

1. 组件的 @Input() 引用发生变化。
2. 组件的 DOM 事件，包括它子组件的 DOM 事件，比如 click、submit、mouse down 等事件。
3. Observable 订阅事件，同时设置 Async pipe。
4. ChangeDetectorRef.detectChanges()、ChangeDetectorRef.markForCheck()、ApplicationRef.tick()，手动调用这三种方式触发变化检测。


我们来看一个例子，有一个组件树结构如下：

![angular-onpush-ngdocheck](https://limeii.github.io/assets/images/posts/angular/angular-ngdocheck-onpush-strategy01.png){:height="100%" width="100%"}

这三个组件代码如下：

```ts
//Component A
import { Component } from "@angular/core";
@Component({
    template: `<h1>this is component a</h1>
               <app-componentb></app-componentb>`
})
export class AComponent {

}
```

```ts
//Component B
import { Component, OnChanges, DoCheck } from "@angular/core";
@Component({
    selector: 'app-componentb',
    template: `<h3>this is component b</h3>
               <app-componentc></app-componentc>`
})
export class BComponent implements OnChanges, DoCheck {

    ngOnChanges() {
        console.log('ngOnChanges triggered in component B');
    }

    ngDoCheck() {
        console.log('ngDoCheck triggered in component B');
    }

}
```

```ts
//Component C
import { Component, OnChanges, DoCheck } from "@angular/core";
@Component({
    selector: 'app-componentc',
    template: `<h5>this is component C</h5>`
})
export class CComponent implements OnChanges, DoCheck {
    ngOnChanges() {
        console.log('ngOnChanges triggered in component C');
    }
    ngDoCheck() {
        console.log('ngDoCheck triggered in component C');
    }
}
```

## 第一种情况：三个组件都是默认的检测策略.

没有```@Input```绑定、没有 DOM 事件、没有```Observable```、没有手动触发变化检测，页面运行起来以后，效果如下：

![angular-onpush-ngdocheck](https://limeii.github.io/assets/images/posts/angular/angular-ngdocheck-onpush-strategy02.png){:height="100%" width="100%"}

在这种情况下没有```@Input```绑定，所以```ngOnChanges```不会被触发，那为什么组件B和C的```ngDoCheck```分别执行了两遍？


其实在【[Angular：单向数据流](https://limeii.github.io/2019/06/angular-unidirectional-data-flow/)】这篇文章中解释过：整个变化检测和组件生命周期钩子函数的执行顺序之间的关系。对于组件A B C实际整个变化检测流程为：

```
Checking A component:
  - update B input bindings
  - call NgDoCheck on the B component
  - update DOM interpolations for component A
 
 Checking B component:
    - update C input bindings
    - call NgDoCheck on the C component
    - update DOM interpolations for component B
 
   Checking C component:
      - update DOM interpolations for component C
```
在父组件执行变化检测，会更新子组件的绑定，从而触发一次子组件的```ngDoCheck```；第二遍是子组件它本身的变化检测触发的。

<blockquote>
<p>这种情况就解释了 NgDoCheck 执行的第二种情景：每次变化检测以后，都会触发 ngDoCheck 钩子函数，紧跟在 ngOnChanges 和 ngOnInit 之后运行</p>
</blockquote>

## 第二种情况：把 ComponentB 的变化检测策略设置为 OnPush

没有```@Input```绑定、没有 DOM 事件、没有```Observable```、没有手动触发变化检测，那么 Angular 只会对组件A执行变化检测，跳过组件 B 和 C 的变化检测，代码如下：

```ts
// Component B with ChangeDetectionStrategy.OnPush
import { Component, OnChanges, DoCheck, ChangeDetectionStrategy } from "@angular/core";
@Component({
    selector: 'app-componentb',
    template: `<h3>this is component b</h3>
               <app-componentc></app-componentc>`,
    changeDetection: ChangeDetectionStrategy.OnPush
})
export class BComponent implements OnChanges, DoCheck {
    ngOnChanges() {
        console.log('ngOnChanges triggered in component B');
    }
    ngDoCheck() {
        console.log('ngDoCheck triggered in component B');
    }
}
```

页面运行起来后效果如下：

![angular-onpush-ngdocheck](https://limeii.github.io/assets/images/posts/angular/angular-ngdocheck-onpush-strategy03.png){:height="100%" width="100%"}

很奇怪对不对！！本来说好的组件 B 和 C 不会执行变化检测，怎么```NgDoCheck```还是触发了？组件 B 中的```ngDoCheck```执行了两遍！！


我们来看下在组件 B 中设置了```OnPush```检测策略，整个变化检测流程：

```
Checking A component:
  - update B input bindings
  - call NgDoCheck on the B component
  - update DOM interpolations for component A
 if (bindings reference changed 
    || DOM event 
    || Observable Async pipe
    || ChangeDetectorRef.detectChanges()
    || ChangeDetectorRef.markForCheck()
    || ApplicationRef.tick()) -> checking B component:
    - update C input bindings
    - call NgDoCheck on the C component
    - update DOM interpolations for component B
 
   Checking C component:
      - update DOM interpolations for component C
```

组件 B 执行第一个```NgDoCheck```很好理解：子组件里的```NgDoCheck```每次都是在父组件执行变化检测的时候执行，组件 B 虽然设置了```OnPush```策略，但是父组件 A 的在执行变化检测时会触发一次 B 的```NgDoCheck```。


组件 B 执行第二个```NgDoCheck```是因为：组件 C 在它自己的```NgDoCheck```之后会**update DOM interpolations for component B**，组件 B 状态发生改变，因为此时组件 B 设置了```OnPush```，不会执行变化检测，Angular 捕获不到这个状态改变。```NgDoCheck```会在状态发生变化，Angular 自己又不能捕获时被触发。


组件 C 执行```NgDoCheck```触发的原因和组件 B 的第二个```NgOnCheck```类似：**update DOM interpolations for component C**组件 C 本身自己页面渲染时状态发生改变，因为组件 B 被设置为```OnPush```策略，子组件 C 也是```OnPush```策略，Angular 捕获不到这个状态改变。```NgDoCheck```会在状态发生变化，Angular 自己又不能捕获时被触发。

<blockquote>
<p>这种情况就解释了 NgDoCheck 执行的第一种情景：在状态发生变化，Angular 自己本身不能捕获这个变化时会触发 NgDoCheck</p>
</blockquote>

## 总结

ngDoCheck 不只是在变化检测后触发，在组件里设置 OnPush 策略，ngDoCheck 依旧可以被触发。


本文中用到到的示例代码在这里：[angular-change-detection](https://github.com/LiMeii/angular-change-detection)