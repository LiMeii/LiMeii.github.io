---
title: Angular：单向数据流
tags: Angular
layout: post
---

在介绍angular单向数据流之前我们先来看下angular中component之间关系树状结构图：

![angular component tree](https://limeii.github.io/assets/images/posts/angular/angular-components-tree.png){:height="50%" width="50%"}

比如在child A component从http response拿到最新model的值，并且需要把变化后的值渲染到页面，这个过程会触发child A component的变化检测（change detection）。

<blockquote>
<p>
这个变化检测不仅仅会在child A component中执行，它会从root component开始沿着component关系树结构从上到下执行，直到最后一个child component完成变化检测达到稳定状态。也就是说在GrandChild component中也会触发执行变化检测，如果GrandChild component中还有它自己的child component，会继续触发它的child component的变化检测。在这个从上到下的变化检测流中，一旦child A component中的变化检测已经完成了，任何在GrandChild component或者更低层级的component都不允许去改变child A component中的属性。
</p>
</blockquote>

**<font color="#BF1827">这个过程就是angular的单向数据流。</font>**


但是在GrandChild component里，有些钩子函数通过```@Output```去改变child A component的数据值，是允许的，而有些又不允许。


现在我们来看看，如果在GrandChild component的钩子函数里通过```@Output```去改child A component的值，会发生什么？

**定义一个ChildAComponent，在这里会显示从GrandChildComponent发过来的message，代码如下：**

```ts
@Component({
    template:`<h3 style="color: black">this is child A page</h3>
            <h4 style="color: black">message from grand child: { { msgFromGrandChild } } </h4>
            <app-grand-child (sendMsgToParent)="getMsgFromGrandChild($event)">
                <span style="color: coral">here is the content projection</span>            
            </app-grand-child>`
})

export class ChildAComponent {
    msgFromGrandChild: any = '';
    getMsgFromGrandChild(value: any) {
        this.msgFromGrandChild = value;
    }
}
```

**定义一个GrandChildComponent，通过@Output向ChildAComponent发送message，代码如下：**

```ts
@Component({
    selector: "app-grand-child",
    template: `<h5 style="color: crimson">this is grand child</h5>
                <ng-content></ng-content>`
})

export class GrandChildComponent implements  OnInit {
    @Output() sendMsgToParent: EventEmitter<any> = new EventEmitter<any>();
    msg = "hello, change this to parent component"
    ngOnInit() {
        this.sendMsgToParent.emit(this.msg);
    }
}
```

**1. GrandChild组件，在ngOnInit函数中改变了ChildAComponent的msgFromGrandChild属性的值**

效果如下：message能正常显示也没有error：
![angular-unidirectional-data-flow](https://limeii.github.io/assets/images/posts/angular/angular-unidirectional-data-flow3.png){:height="100%" width="100%"}

<blockquote>
<p>
把ngOnInit换成ngDoCheck、ngAfterContentInit、ngAfterContentChecked、ngOnChanges，效果是一样的，在ChildAComponent中message都能正常显示也不会报错。
</p>
</blockquote>

**2. GrandChild组件，在ngAfterViewInit中去改childA组件msgFromGrandChild属性的值** 

代码如下：

```ts
export class GrandChildComponent implements AfterViewInit {
    @Output() sendMsgToParent: EventEmitter<any> = new EventEmitter<any>();
    msg = "hello, change this to parent component"
    ngAfterViewInit() {
        this.sendMsgToParent.emit(this.msg);
    }
}
```

这个时候会发现在console里会有```ExpressionChangedAfterItHasBeenCheckedError```，具体如下：

![angular-unidirectional-data-flow](https://limeii.github.io/assets/images/posts/angular/angular-unidirectional-data-flow5.png){:height="100%" width="100%"}

<blockquote>
<p>
如果把ngAfterViewInit换成ngAfterViewChecked，效果也是一样的，会有同样的错误。
</p>
</blockquote>

**出现这种错误的原因是：**
在angular中强制了单向数据流，当有变化的时候，变化检测机制是沿着component关系树结构从上到下执行，直到最后一个child component变化检测完成，这个完整的变化检测才算结束。在这个过程中，parent component的变化检测完成以后，任何更低一层级去改上一层级的属性，都不允许。如果是在生产环境里，也就是启用了enableProdMode()会直接忽略这样的操作，页面也不会显示变化以后的值，也不会报错。但是在开发模式下，在每一次变换检测（change detection）以后，angular会从上到下再多跑一个变化检测，确保每次改动之后所有的状态是stable的，这个时候发现有低层级改动上一层级的值，就会出现上面那个错误。

**那为什么在ngAfterViewInit和ngAfterViewChecked会报错，而且其他几个钩子函数里不报错呢？**

我们来调试一下他的core.js源代码，具体调试方法如下：

![angular-unidirectional-data-flow](https://limeii.github.io/assets/images/posts/angular/angular-unidirectional-data-flow06.gif){:height="100%" width="100%"}

把```checkAndUpdateView```方法简化一下：

```js
function checkAndUpdateView(view, ...) {
    ...       
    // update input bindings on child views (components) & directives,
    // call NgOnInit, NgDoCheck, ngOnChanges,ngAfterContentInit, ngAfterContentChecked hooks if needed
    Services.updateDirectives(view, CheckType.CheckAndUpdate);
    
    // DOM updates, perform rendering for the current view (component)
    Services.updateRenderer(view, CheckType.CheckAndUpdate);
    
    // run change detection on child views (components)
    execComponentViewsAction(view, ViewAction.CheckAndUpdate);
    
    // call AfterViewChecked and AfterViewInit hooks
    callLifecycleHooksChildrenFirst(…, NodeFlags.AfterViewChecked…);
    ...
}
```
从上面的代码可以看出来，在GrandChild component中，```AfterViewChecked```和```AfterViewInit```是在它自己的变化检测（change detetion）之后再执行的，也就是它状态stable之后再执行的，这时候在去触发它上一层级属性的改动，被认为是违反angular的单向数据流。


从上面的分析可以看到：angular在变化检测（change detection）过程中也会去触发生命周期钩子函数。比较有意思的是，有些钩子函数是在DOM Rending/change detection之前触发，有些是在之后触发。


最后来总结一下，在angular整个页面渲染的过程：

![angular-unidirectional-data-flow](https://limeii.github.io/assets/images/posts/angular/angular-unidirectional-data-flow7.png){:height="100%" width="100%"}

1. 更新child component的input bindings，然后会触发child component中OnInit、DoCheck、OnChanges函数，如果页面有ng-content，相应也会触发ngAfterContentInit和ngAfterContentChecked。

2. angular会继续渲染当前component也就是parent component页面。

3. 触发child component中的变化检测（change detection）。

4. 触发child component中的AfterViewInit和theAfterViewChecked。


在【[Angular Change Detection:变化检测机制](https://limeii.github.io/2019/06/angular-changedetection/)】这篇文章里介绍了变化检测机制。

本文中用到的示例代码在这里：[angular-change-detection](https://github.com/LiMeii/angular-change-detection)