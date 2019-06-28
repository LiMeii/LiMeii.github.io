---
title: Angular Change Detection：变化检测机制
tags: Angular
layout: post
---

<blockquote>
<p>
angular中的变化检测机制是当component状态有变化的时候，angular都能检测到这些变化，并且能够将这些变化反应到页面上。
</p>
</blockquote>

比如有这样一个component，代码如下：

```ts
@Component({
  template:`<h1>I am <span [textContent]="data.name"></span><h1>`
})
export class CDParentComponent {
  data : any = { name:'meii', address: 'ShangHai'};
}
```

当把cdParentComponent中```data.name```的值改成limeii，页面会直接把meii更新成limeii，看似很简单的一个改动，其实在angular内部涉及到很多复杂的操作，包括```变化检测```、```脏数据检查```、```数据绑定```、```单向数据流```、```更新DOM```、```NgZone```等等。


单向数据流在这篇[Angular：单向数据流](https://limeii.github.io/2019/06/angular-unidirectional-data-flow/)文章有详细介绍，也提到angular 应用其实就是组件树，变化检测都是沿着组件树从root component开始至上而下执行的。我们都知道在angular里，每个component都有一个html模板，在angular内部，编译器在component和模板之间会生成一个component view。数据绑定、脏数据检查和更新DOM都是由这个component view实现的。变化检测机制也可以说就是沿着component view的树状结构从上到下执行的。


## component view到底是什么？


把```data.name```值改成limeii，会触发变化检测，同时angular会做数据脏检查，也就是对比当前值（limeii）和之前的值（oldvalue：meii）是否一样，如果发现两者不一致，会把当前的值（limeii）更新到页面上。同时也会把当前的值保持为oldvalue。


为了实现上述流程，angular需要component view保存每个DOM节点引用，同时也要保存component数据引用、数据之前的值和取值表达式。如下所示：

![angular-change-detection](https://limeii.github.io/assets/images/posts/angular/angular-change-detection02.png){:height="100%" width="100%"}

当编译器分析组件模板的时候，知道每次变化检测有可能需要更新页面上DOM元素属性的值，对于这些属性，编译器都会给它创建绑定，绑定里至少有这个属性名称和取值的表达式。


如上代码，属性```data.name```是component的值，```textContent```是对应页面span元素的属性，编译器会通过绑定把这两者关联起来。

![angular-change-detection](https://limeii.github.io/assets/images/posts/angular/angular-change-detection03.png){:height="100%" width="100%"}

有了component view、绑定、脏数据检查，组件状态变化就可以触发变化检测从而更新页面DOM属性的值。


## 那什么会触发组件状态的变化？


最常见的一种方式，在页面按钮的```click```事件更新```data.name```的值，代码如下：

```ts
@Component({
  template:`<h1>I am <span [textContent]="data.name"></span><h1>
            <button (click)="changeName()">Change Name</button>`
})
export class CDParentComponent {
  data : any = { name:'meii', address: 'ShangHai'};
  changeName(){
    this.data.name = "limeii";
  }
}
```

还有一种常见的方式是，通过http request拿到data的值，如下：

```ts
  ngOnInit() {
    this.http.get('/contacts')
      .map(res => res.json())
      .subscribe(contacts => this.data.name = contacts.name;);
  }

```

**<font color="#BF1827">angular通常有如下三种方式会导致组件数据变化：</font>**

1. 事件：页面 click、submit、mouse down......
2. XHR：从后端服务器拿到数据
3. Timers：setTimeout()、setInterval()


## angular又怎么通知各个组件做变化检测？


前面那三种方式会导致angular状态变化，那又是谁知道状态已经发生改变，需要通知angular触发变化检测从而更新页面DOM呢？```NgZone（zone.js）```充当了这个角色。


NgZone可以简单的理解为是一个异步事件拦截器，它能够hook到异步任务的执行上下文，然后就可以来处理一些操作，比如每个异步任务callback以后就会去通知angular做变化检测。


angular源码中有一个```ApplicationRef```，可以监听NgZones ```onTurnDone```事件，每当```onTurnDone```被触发后，它会立马执行```tick()```方法，```tick()```会从上到下沿着组件树触发变化检测。```ApplicationRef```简洁版代码如下：

```ts
// very simplified version of actual source
class ApplicationRef {
  changeDetectorRefs:ChangeDetectorRef[] = [];

  constructor(private zone: NgZone) {
    this.zone.onTurnDone
      .subscribe(() => this.zone.run(() => this.tick());
  }

  tick() {
    this.changeDetectorRefs
      .forEach((ref) => ref.detectChanges());
  }
}
```


每个component都有自己的变化检测器，负责检查它们各自的绑定，结构如下：

![angular-change-detection](https://limeii.github.io/assets/images/posts/angular/angular-change-detection05.png){:height="100%" width="100%"}


有了NgZone上述三种异步事件都会导致整个angular应用发生变化检测，虽然angular变化检测本身性能已经很好了，在毫秒内可以做成百上千次变化检测。但是随着项目越来越大，其实很多不必要的变化检测还是会在一定程度上影响性能。


在这篇文章[Angular Change Detection:变化检测策略](https://limeii.github.io/2019/06/angular-changeDetectionStrategy-OnPush/)介绍了如何通过OnPush来跳过一些不必要的变化检测，从而优化整个应用的性能。