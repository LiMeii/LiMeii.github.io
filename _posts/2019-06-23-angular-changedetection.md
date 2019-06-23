---
title: Angular Change Detection:变化检测机制
tags: Angular
layout: post
---

angular中的变化检测机制是当component状态有变化的时候，angular都能检测到这些变化，并且能够将这些变化反应到页面上。


比如有这样一个component，代码如下：

![angular-change-detection](https://limeii.github.io/assets/images/posts/angular/angular-change-detection01.png){:height="100%" width="100%"}

当把cdParentComponent中data.name的值改成limeii，页面会直接把meii更新成limeii，看似很简单的一个改动，其实在angular内部涉及到很多复杂的操作，包括变化检测、脏检测机制、数据绑定、单向数据流、更新DOM、NgZone等等。


在上一篇文章[Angular：单向数据流](https://limeii.github.io/2019/06/angular-unidirectional-data-flow/)中有介绍单向数据流是怎么回事，也提到angular application其实就是组件树，变化检测都是沿着组件树从上到下执行的。我们都知道在angular里，每个component都有一个html 模板，在angular内部，编译器在component和模板之间会生成一个component view。这个component view把component和模板关联起来，双向数据绑定、脏数据检测和更新DOM都是由这个component view实现的。变化检测机制也可以说就是沿着component view的树状结构从上到下执行的。


**component view到底是什么？**


把data.name值改成limeii，会触发变化检测，同时angular会做数据脏检查，也就是对比当前值（limeii）和之前的值（meii）是否一样，如果发现两者不一致，会把当前的值（limeii）更新到页面上。


为了实现上述流程，angular需要component view保存每个DOM节点的引用，同时也需要保存component的数据引用、这个数据之前的值和取值表达式。如下所示：

![angular-change-detection](https://limeii.github.io/assets/images/posts/angular/angular-change-detection02.png){:height="100%" width="100%"}

当编译器编分析组件模板的时候，知道每次变化检测有可能需要更新页面上DOM元素属性的值，对于这些属性，编译器都会给它创建绑定，绑定里至少有这个属性名称和取值的表达式。


在上面的代码中，属性data.name是component的值，textContent是对应页面span元素的属性，编译器会通过绑定把这两者关联起来。

![angular-change-detection](https://limeii.github.io/assets/images/posts/angular/angular-change-detection03.png){:height="100%" width="100%"}

有了component view、绑定、脏数据检查，组件内部数据的变化就可以触发变化检测从而更新页面DOM属性的值。


**什么会触发组件数据的变化？**


最常见的一种方式，在页面按钮的click事件更新data.name的值，代码如下：

![angular-change-detection](https://limeii.github.io/assets/images/posts/angular/angular-change-detection04.png){:height="100%" width="100%"}

还有一种常见的方式是，通过http request拿到data的值，如下：

```ts
  ngOnInit() {
    this.http.get('/contacts')
      .map(res => res.json())
      .subscribe(contacts => this.data.name = contacts.name;);
  }

```

angular中会检测onMicrotaskEmpty，当onMicrotaskEmpty没有异步事件以后，就不会触发变化检测。也就是通常有如下三种方式会导致组件数据变化：

- 事件：页面 click、submit、mouse down......
- XHR：从后端服务器拿到数据
- Timers：setTimeout()、setInterval()


**angular又怎么知道要做变化检测？**


前面那三种会导致数据变化从而angular状态变化，那又是谁通知angular要做变化检测更新页面DOM呢？NgZone（zone.js）充当了这个角色。


NgZone的主要工作是处理angular中所有的异步操作（由前面三种方式触发的），每当有异步操作的时候，NgZone会触发变化检测。



**未完待续**