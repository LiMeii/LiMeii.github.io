---
title: Angular Change Detection:变化检测策略
tags: Angular
layout: post
---

在[Angular Change Detection:变化检测机制](https://limeii.github.io/2019/06/angular-changedetection/)这篇文章介绍了angular的变化检测机制，也提到了页面操作（click，submit...）、XHR、Timers（setTimeout，setInterval）这些异步事件都会触发整个angular应用的变化检测。


angular默认的变化检测机制是**ChangeDetectionStrategy.Default**：异步事件callback结束后，NgZone会触发整个组件树至上而下做变化检测，如下所示：

![angular-change-detection](https://limeii.github.io/assets/images/posts/angular/angular-change-detection-strategy01.png){:height="100%" width="100%"}

但是在实际应用里，并不是每个异步操作需要变化检测，某些组件也可以完全不用做变化检测，过多的变化检测实际上也影响了整个应用的性能。angular除了默认的变化检测机制，也提供了**ChangeDetectionStrategy.OnPush**，用OnPush可以跳过某个component或者某个父组件以及它下面所有子组件的变化检测，如下所示：

![angular-change-detection](https://limeii.github.io/assets/images/posts/angular/angular-change-detection-strategy02.png){:height="100%" width="100%"}

我们来看下OnPush具体是怎么用的：


定义一个CDParentComponent如下：

![angular-change-detection](https://limeii.github.io/assets/images/posts/angular/angular-change-detection-strategy03.png){:height="100%" width="100%"}

定义一个CDChildComponent如下：

![angular-change-detection](https://limeii.github.io/assets/images/posts/angular/angular-change-detection-strategy04.png){:height="100%" width="100%"}

在CDChildComponent加了一行代码：**changeDetection: ChangeDetectionStrategy.OnPush**，我们点击Change Info按钮，不会触发CDChildComponent中的变化检测，页面email也不会有变化。


在CDChildComponent加了OnPush表示，在发生异步事件以后触发变化检测，angular会跳过这个组件，不会触发这个组件的变化检测。如果OnPush是加在某个父组件上，那么这个父组件和它下面所有的子组件都不会触发变化检测。


但是在实际应用里，我们并不希望把整个组件的变化检测都禁掉，而是希望部分操作还是可以触发它的变化检测，比如从后端API返回新的数据，虽然加了OnPush，这些数据还是能够更新在页面上。angular也考虑这种情况，在组件里加了OnPush，以下四种情况还是可以触发该组件的变化检测：

- 组件的@Inputs()发生改变，同时@Inputs的引用也发生改变。

- 组件的DOM事件，包括它的子组件的DOM事件，比如click、submit、mouse down触发。

- observable事件，并且设置了Async pipe

- 手动用ChangeDetectorRef.detectChanges()、ChangeDetectorRef.markForCheck()、ApplicationRef.tick()方式触发变化检测




**未完待续**