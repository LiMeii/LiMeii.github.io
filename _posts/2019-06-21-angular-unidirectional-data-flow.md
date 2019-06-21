---
title: Angular：单向数据流
tags: Angular
layout: post
---

在介绍angular单向数据流之前我们先来看下angular中component之间关系树状结构图：

![angular component tree](https://limeii.github.io/assets/images/posts/angular/angular-components-tree.png){:height="50%" width="50%"}

比如在child A component从http response拿到最新model的值，并且需要把变化后的值渲染到页面，这个过程会触发child A component的change detection。


这个change detection不仅仅会在child A component中执行，它会沿着component关系树结构从上到下执行，也就是说在GrandChild component中也会触发执行change detection。如果GrandChild component中还有它自己的child component，会继续触发这个它的child component的chang detection。一旦child A component中的change detection已经完成了，任何在GrandChild component或者更低层级的component都不允许去改变child A component中的属性。


这个过程就是angular的单向数据流。


现在我们来看看，如果在GrandChild component的钩子函数里去改child A component的值，会发生什么？


**1. 在GrandChild component中，如果在ngOnChanges/ngOnInit/ngDoCheck函数中去改child A component中的属性会被立即更新。**


**2. 如果在ngAfterViewChecked和ngAfterViewInit中去改child A component的属性** child A component不会立马更新而是要等到下一次child A component的change detection才会更新。如果是在开发环境里，也就是没有enableProdMode的情况下，还会报错：ExpressionChangedAfterItHasBeenCheckedError。

