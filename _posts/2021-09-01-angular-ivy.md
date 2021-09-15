---
title: Angular：Ivy模板引擎
tags: Angular
layout: post
---

最近做了个 Angular 升级项目，从 Angular8 升级到 Angular12，做这个升级的时候，发现 Angular9 默认使用了 Ivy 模板引擎 和 AOT 编译方式。升级项目结束以后，花时间看了下 Ivy 的相关介绍，写篇文章记录下学习成果吧。


在这篇文章中会介绍以下内容：
- 什么是 Ivy
- 为什么要用 Ivy
- 什么是 ngc
- Angular 引入 Ivy 以后，编译机制有哪些改变
- Ivy 对性能的影响

## 什么是 Ivy

我们在写 Angular 项目的时候，比如创建一个 component，一般会有两个文件，一个是 ts 文件，一个是 html 文件，在 html 文件里，我们会用到一些 Angular 的 directive 和 pipe（比如 *ngIf / *ngFor）这些 Angular 语法是没办法直接在浏览器里解析渲染，需要先编译成浏览器可识别的代码，才能正常运行和渲染。在 Angular9 之前的版本，负责这部分的编译叫 View Engine，Angular9 以后的版本就是用 Ivy 模板引擎把带有 Angular 语法的 html 编译成浏览器可识别的 js 代码 和 html 代码。

## 为什么要用 Ivy

Ivy 是完全重写了 Angular 的编译器 和 runtime，Ivy 有以下优势：
- 大大减少了编译时间
- 大大减少了编译以后 bundle 文件的大小
- html 文件也可以 做 tree shaking

在【[Angular：深入理解Angular编译机制](https://limeii.github.io/2019/08/angular-compiler/)】介绍了 Angular 的编译机制（基于 View Engine），可以先来看看 View Engine 和 Ivy 编译机制有哪些区别。

### 编译文件
View Engine 编译以后会有以下四个文件（详细介绍可以参考【[Angular：深入理解Angular编译机制](https://limeii.github.io/2019/08/angular-compiler/)】）：
- ```*.metadata.json```：把 .ts(component/NgModule) 文件里的 decorator 信息和 constructor 的依赖注入信息用 json 的形式记录下来，下次在二次编译的时候不需要再从 .ts 文件里拿了。二次编译是指在自己项目中引用第三方库，在编译自己项目的时候需要对第三方库进行二次编译打包。如果我们自己项目要用 AoT 编译，那么第三方库必须要提供 .metadata.json 文件。

- ```*.ngfactory.js```：里面包含了创建组件、渲染组件(涉及 DOM 操作)、执行变化检测(获取 oldValue 和 newValue 对比)、销毁组件的代码，也就是我们说的 component view。

- ```*.js```：是 .ts(component/NgModule) 文件里除 decorator 和 constructor 之外的内容，编译成了 es6 代码。

- ```*.ngsummary.json```：包含了 .metadata.json 中所有的信息，因此如果使用了 ngsummary.json，就不需要 .metadata.json了。

Ivy 编译之后只有一个文件：
- 不会生成```*.ngfactory.js``` ```*.medadata.json``` ```*.ngsummary.json```文件
- 编译后就只有一个 ```*.js``` 文件

比如在 Angular12 的项目里，有一个 component 如下：
```html
<!--app.component.html--> 
<<<<<<< Updated upstream
<h1>this {{title}}</h1>
=======
<h1>this { { title } } </h1>
>>>>>>> Stashed changes
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

用 ngc 编译以后的```app.component.js```代码如下：
```js
import { Component } from '@angular/core';
import * as i0 from "@angular/core";
export class AppComponent {
    constructor() {
        this.title = 'angular12 app';
    }
}
AppComponent.ɵfac = function AppComponent_Factory(t) { return new (t || AppComponent)(); };
AppComponent.ɵcmp = /*@__PURE__*/ i0.ɵɵdefineComponent({ type: AppComponent, selectors: [["app-root"]], decls: 2, vars: 1, template: function AppComponent_Template(rf, ctx) { if (rf & 1) {
        i0.ɵɵelementStart(0, "h1");
        i0.ɵɵtext(1);
        i0.ɵɵelementEnd();
    } if (rf & 2) {
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



## 什么是 ngc
在 Anuglar9 之前的模板引擎是 View Engine，Angular9 默认使用 Ivy，Ivy 重写了整个编译器，但是我们实际在写代码的时候，并没有太大的影响，比如：代码语法没有改变；从 Angular8 升级到 Angular12，业务逻辑代码语法不需要改动，可以直接运行。对于开发人员来说，从 View Engine 升级到 Ivy，并没有什么痛苦，可以说是没什么感觉，这是因为有 ngc 在中间帮我们这部分工作都做好了。


<<<<<<< Updated upstream
ngc 是一个 ‘compatibility compiler’，这个 ngc 编译器就负责一个任务：在编译的过程的中，会去检查```node_modules```，如果有 Angular 的 lib，会去读取这个 lib 的```metadata.json```文件和 JS 代码，会把这些代码编译成 Ivy 可以识别的代码。这个编译过程直接写在了 Angular CLI 里面，并不需要我们手动触发，在第一次跑```ng serve``` ```ng build```，会发现编译的时间要长一些。因为在第一次编译的时候，ngc 会把基于 View Engine 写的代码编译成兼容 Ivy 的代码，这个编译只是在第一次跑 ```ng serve``` ```ng build```的时候会做，但是如果你加了新的 Angular lib，会触发 ngc 编译。
=======
ngc 是一个 ‘compatibility compiler’，这个 ngc 编译器就负责一个任务：在编译的过程的中，会去检查```node_modules```，如果有 Angular 的 lib，会去读取这个 lib 的```metadata.json```文件和 JS 代码，会把这些代码编译成 Ivy 可以识别的代码。这个编译过程直接写在了 Angular CLI 里面，并不需要我们手动触发，在第一次跑```ng serve``` ```ng build```，会发现编译的时间要长一些。因为在第一次编译的时候，ngc 会把基于 View Engine 写的代码编译成兼容 Ivy 的代码，这个编译只是在第一次跑 ```ng serve``` ```ng build```的时候会做，如果你加了新的 Angular lib，会重新触发 ngc 编译。
>>>>>>> Stashed changes

