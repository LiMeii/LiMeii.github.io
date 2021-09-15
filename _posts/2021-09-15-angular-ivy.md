---
title: Angular：Ivy
tags: Angular
layout: post
---

最近做了个 Angular 升级项目，从 Angular8 升级到 Angular12，做这个升级的时候，发现 Angular9 默认使用了 Ivy 模板引擎 和 AOT 编译方式。升级项目结束以后，花时间看了下 Ivy 的相关介绍，写篇文章记录下学习成果吧。


在这篇文章中会介绍以下内容：
- 什么是 Ivy
- Angular 引入 Ivy 以后，编译机制有哪些改变
- 什么是 ngcc
- Ivy 对性能的影响

## 什么是 Ivy

我们在写 Angular 项目的时候，比如创建一个 component，一般会有两个文件，一个是 ts 文件，一个是 html 文件，在 html 文件里，我们会用到一些 Angular 的 directive 和 pipe（比如 *ngIf / *ngFor）这些 Angular 语法是没办法直接在浏览器里解析渲染，需要先编译成浏览器可识别的代码，才能正常运行和渲染。在 Angular9 之前的版本，负责这部分的编译叫 View Engine，Angular9 以后的版本就是用 Ivy 把带有 Angular 语法的 html 编译成浏览器可识别的 js 代码 和 html 代码，Ivy 不仅仅是个模板引擎还是个编译器。

## Angular 引入 Ivy 以后，编译机制有哪些改变

### 局部性
局部性指的是，Ivy 只允许读取定义在组件内部的信息，View Engine 需要全局分析整个项目里的代码。

### Tree Shaking
View Engine 编译后的```*.ngfactory.js```（component view）会把整个模板框架的代码都打包到 bundle 文件里；Ivy 编译后的代码，只会把需要用到的框架代码打包到 bundle 文件里，大大减少 bundle 文件的大小。

### Runtime
**View Engine** 在编译过程中会生成一个```*.ngfactory.js```文件， 其实 ngfactory 就是 Component View。在```*.ngfactory.js```生成过程中，ngcc 会把所有可能发生变化的```DOM Nodes/Elements```都找出来，然后给这些```DOM Nodes/Elements```生成```Bindings```，这些```Bindings```里会记录```Element Name/Expression/OldValue```，一旦有异步事件发生（Click 事件或者是 HttpRequest）就会被```ngZone```捕获到，然后触发```Change Detection```，也就是会从```Root Component```开始，从上到下检查所有组件的```Bindings```也就是前面提到的```Component View```，对比```NewVaule```和```OldValue```，如果不一致就会把新值更新到页面，同时把新值更新为旧值（这也就是我们经常提到的脏检查机制```Dirty Checking```）。 整个过程可以用下图理解：
![view-engine](/assets/images/posts/angular/angular-ivy1.png){:height="100%" width="100%"}

**Ivy** 把 component 编译成指令，这些指令可以实例化组件、创建 DOM Tree、执行 Change Detection。需要注意的是这些指令不需要框架的渲染引擎去解析执行，指令本身就是渲染引擎。整个过程如下：
![view-engine](/assets/images/posts/angular/angular-ivy2.png){:height="100%" width="100%"}

比如在 Angular12 的项目里，有一个 component 如下：
```html
<!--app.component.html--> 
<h1>this { { title } } </h1>
```

```ts
//app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'angular12 app';
}
```

用 ngcc 编译以后的```app.component.js```代码如下：
```js
import { Component } from '@angular/core';
import * as i0 from "@angular/core";
export class AppComponent {
    constructor() {
        this.title = 'angular12 app';
    }
}
AppComponent.ɵfac = function AppComponent_Factory(t) { return new (t || AppComponent)(); };
AppComponent.ɵcmp = /*@__PURE__*/ i0.ɵɵdefineComponent({ type: AppComponent, selectors: [["app-root"]], decls: 2, vars: 1, template: function AppComponent_Template(rf, ctx) 
    { if (rf & 1) {
        // create dom instructions
        i0.ɵɵelementStart(0, "h1");
        i0.ɵɵtext(1);
        i0.ɵɵelementEnd();
    } if (rf & 2) {
        // update dom instructions
        i0.ɵɵadvance(1);
        i0.ɵɵtextInterpolate1("this ", ctx.title, "");
    } }, styles: [""] });
(function () { (typeof ngDevMode === "undefined" || ngDevMode) && i0.ɵsetClassMetadata(AppComponent, [{
        type: Component,
        args: [{
                selector: 'app-root',
                templateUrl: './app.component.html',
                styleUrls: ['./app.component.css']
            }]
    }], null, null); })();
//# sourceMappingURL=app.component.js.map
```

Ivy 是基于 Incremental DOM 做的，有点类似于 React 里的 Virtual DOM，跟 Virtual DOM 不同的是：Virtual DOM 是基于整个树做 diff，然后再把diff渲染到页面上（会在内存里生成一个完整的 DOM Tree）；Incremental DOM 是基于节点对比（内存里不会有 DOM Tree），如果节点有变化，直接把变化更新页面，不需要分配内村来保存这个更新，只有增加或删除节点的时候才会分配内存来保存新加/删除的节点。更多关于 Incremental DOM 的介绍可以参考：【[Introducing Incremental DOM](https://medium.com/google-developers/introducing-incremental-dom-e98f79ce2c5f)】


Incremental DOM 有以下两个优势：
- 渲染引擎自己是 Tree Shakable，大大的减小 bundle 文件的大小
- Incremental DOM 只会给新加/删除的节点分配内存，内存分配大大减少了

当我们用 Incremental DOM，指令不需要框架来解析编译，指令本身就是渲染引擎，那么在生成这些指令的时候（也就是编译过程中），用不到的指令不会打包到最终的 bundle 文件里。Virtual DOM 是需要解析编译运行的，在真正运行的时候才知道需要哪些指令，所以打包的时候，会把所有的指令都打包到 bundle 文件里。

Incremental DOM 并不需要内存来生成 DOM Tree，它根本不会去更改 DOM Tree，只是在新加/删除节点的时候分配内存。


Change Detection 还是跟原来的类似，当有异步操作的时候，ngZone 会触发 Change Detection，执行 update dom 的指令来更新渲染页面。

### 编译文件
View Engine 编译以后会有以下四个文件（详细介绍可以参考【[Angular：深入理解Angular编译机制](https://limeii.github.io/2019/08/angular-compiler/)】）：
- ```*.metadata.json```
- ```*.ngfactory.js```
- ```*.js```
- ```*.ngsummary.json```

Ivy 编译之后只有一个文件：
- 不会生成```*.ngfactory.js``` ```*.medadata.json``` ```*.ngsummary.json```文件
- 编译后就只有一个 ```*.js``` 文件


## 什么是 ngcc
在 Anuglar9 之前的模板引擎是 View Engine，Angular9 默认使用 Ivy，Ivy 重写了整个编译器，但是我们实际在写代码的时候，并没有太大的影响，比如：代码语法没有改变；从 Angular8 升级到 Angular12，业务逻辑代码语法不需要改动，可以直接运行。对于开发人员来说，从 View Engine 升级到 Ivy，并没有什么痛苦，可以说是没什么感觉，这是因为有 ngcc 在中间帮我们这部分工作都做好了。


ngcc 是一个 ‘compatibility compiler’，这个 ngcc 编译器就负责一个任务：在编译的过程的中，会去检查```node_modules```，如果有 Angular 的 lib，会去读取这个 lib 的```metadata.json```文件和 JS 代码，会把这些代码编译成 Ivy 可以识别的代码。这个编译过程直接写在了 Angular CLI 里面，并不需要我们手动触发，在第一次跑```ng serve``` ```ng build```，会发现编译的时间要长一些。因为在第一次编译的时候，ngcc 会把基于 View Engine 写的代码编译成兼容 Ivy 的代码，这个编译只是在第一次跑 ```ng serve``` ```ng build```的时候会做，如果你加了新的 Angular lib，会重新触发 ngcc 编译。

## Ivy 对性能的影响
Ivy 完全重写了 Angular 的编译器 和 runtime，Ivy 有以下优势：
- 模板文件也可以 做 Tree Shaking，大大减少了 bundle 文件的大小
- bundle 文件更小，那么整个应用启动时间也会相应的减少
- 内存占用更小


View Engine 的编译机制可以参考文章：【[Angular：深入理解Angular编译机制](https://limeii.github.io/2019/08/angular-compiler/)】
