---
title: Angular:ngDoCheck执行时机
tags: Angular
layout: post
---

在[angular官方文档](https://angular.io/guide/lifecycle-hooks)对ngDoCheck执行时机是这么描述的：

![angular-onpush-ngdocheck](https://limeii.github.io/assets/images/posts/angular/angular-ngdocheck-onpush-strategy04.png){:height="100%" width="100%"}

翻译过来就是：

 - 1：NgDoCheck会在状态发生变化，angular自己又不能捕获时被触发。

 - 2：每次变化检测以后，都会触发ngDoCheck钩子函数，紧跟在ngOnChanges和ngOnInit之后运行。

第二点很好理解，对于第一点：**NgDoCheck会在状态发生变化，angular自己又不能捕获时被触发。**这到底是什么意思呢？

在[Angular Change Detection:变化检测策略](https://limeii.github.io/2019/06/angular-changeDetectionStrategy-OnPush/)这篇文章中解释过：某一个组件中设置angular变化检测策略为OnPush，如果没有以下四种情况，angular是不会为这个组件或者它的子组件执行变化检测。

- 组件的@Input()引用发生变化。

- 组件的DOM事件，包括它子组件的DOM事件，比如click、submit、mouse down等事件。

- Observable订阅事件，并且设置了Async pipe。

- ChangeDetectorRef.detectChanges()、ChangeDetectorRef.markForCheck()、ApplicationRef.tick()，手动调用这三种方式触发变化检测。


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

**第一种情况：这三个组件都是默认的检测策略，**没有@Input绑定、没有DOM事件、没有Observable、没有手动触发变化检测，页面运行起来以后，效果如下：

![angular-onpush-ngdocheck](https://limeii.github.io/assets/images/posts/angular/angular-ngdocheck-onpush-strategy02.png){:height="100%" width="100%"}

在这种情况下没有@Input绑定，所以ngOnChanges不会被触发，那为什么组件B和C的ngDoCheck分别执行了两遍？


其实在[Angular：单向数据流](https://limeii.github.io/2019/06/angular-unidirectional-data-flow/)这篇文章中解释过：整个变化检测和组件生命周期钩子函数的执行顺序之间的关系。那么实际整个变化检测流程为：

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
在父组件执行变化检测，会更新子组件的绑定，从而触发一次子组件的ngDoCheck；第二遍是子组件它本身的变化检测触发的。


**这种情况就解释了NgDoCheck执行的第二种情景：每次变化检测以后，都会触发ngDoCheck钩子函数，紧跟在ngOnChanges和ngOnInit之后运行**


**第二章情况：把Component B的变化检测策略设置为OnPush，**没有@Input绑定、没有DOM事件、没有Observable、没有手动触发变化检测，那么angular只会对组件A执行变化检测，会跳过组件B和C的变化检测，代码如下：

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

很奇怪对不对！！本来说好的组件B和C不会执行变化检测，怎么NgDoCheck还是触发了？组件B中的ngDoCheck执行了两遍！！


在组件B中设置了OnPush检测策略，整个变化检测流程如下：

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

组件B执行第一个NgDoCheck很好理解：子组件里的NgDoCheck每次都是在父组件执行变化检测的时候执行，组件B虽然设置了OnPush策略，但是父组件A的在执行变化检测时会触发一次B的NgDoCheck。


组件B执行第二个NgDoCheck是因为：组件C在它自己的NgDoCheck之后会**update DOM interpolations for component B**，组件B状态发生改变，因为此时组件B设置了OnPush，不会执行变化检测，angular捕获不到这个状态改变。NgDoCheck会在状态发生变化，angular自己又不能捕获时被触发。


组件C执行NgDoCheck触发的原因和组件B的第二个NgOnCheck类似：**update DOM interpolations for component C**组件C本身自己会有页面渲染，状态发生改变，因为组件B被设置为OnPush策略，子组件C也是OnPush策略，angular捕获不到这个状态改变。NgDoCheck会在状态发生变化，angular自己又不能捕获时被触发。


**这种情况就解释了NgDoCheck执行的第一种情景：NgDoCheck会在状态发生变化，angular自己又不能捕获时被触发**


**总结**

ngDoCheck不只是在变化检测后触发，angular本身不能捕获的变化，会在NgDoCheck里被捕获。


本文中用到到的示例代码在这里：[angular-change-detection](https://github.com/LiMeii/angular-change-detection)