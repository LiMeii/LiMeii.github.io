---
title: Angular:在OnPush策略下，@Input引用没有变化，ngDoCheck还是会被触发
tags: Angular
layout: post
---

在[Angular Change Detection:变化检测策略](https://limeii.github.io/2019/06/angular-changeDetectionStrategy-OnPush/)这篇文章中解释过：某一个组件中设置angular变化检测策略为OnPush，如果没有以下四种情况，angular是不会为这个组件或者它的子组件执行变化检测。

- 组件的@Input()引用发生变化。

- 组件的DOM事件，包括它子组件的DOM事件，比如click、submit、mouse down等事件。

- Observable订阅事件，并且设置了Async pipe。

- ChangeDetectorRef.detectChanges()、ChangeDetectorRef.markForCheck()、ApplicationRef.tick()，手动调用这三种方式触发变化检测。


我们来看一个例子，有一个组件树结构如下：

![angular-change-detection](https://limeii.github.io/assets/images/posts/angular/angular-ngdocheck-onpush-strategy01.png){:height="100%" width="100%"}

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

现在这三个组件都是默认的检测策略，没有@Input绑定、没有DOM事件、没有Observable、没有手动触发变化检测，页面运行起来以后，效果如下：

![angular-change-detection](https://limeii.github.io/assets/images/posts/angular/angular-ngdocheck-onpush-strategy02.png){:height="100%" width="100%"}

现在把Component B的变化检测策略设置为OnPush，没有@Input绑定、没有DOM事件、没有Observable、没有手动触发变化检测，那么angular只会对组件A执行变化检测，会跳过组件B和C的变化检测，代码如下：

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

![angular-change-detection](https://limeii.github.io/assets/images/posts/angular/angular-ngdocheck-onpush-strategy03.png){:height="100%" width="100%"}

很奇怪对不对！本来说好的组件B不会执行变化检测，怎么ngDoCheck还是触发了？而且组件C也触发了？


其实在[Angular：单向数据流](https://limeii.github.io/2019/06/angular-unidirectional-data-flow/)这篇文章中解释过：整个变化检测和组件生命周期钩子函数的执行顺序之间的关系。那么实际整个检测变化流程为：

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
从上述流程中可以看到，子组件里的NgDoCheck每次都是在父组件执行变化检测的时候执行。那就不难理解为什么组件B设置为OnPush策略，但是依然会执行组件B的NgDoCheck事件。


但在组件B中设置了OnPush检测策略，整个变化检测流程如下：

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

**未完待续**