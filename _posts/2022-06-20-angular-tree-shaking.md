---
title: Angular性能优化：Tree Shaking
tags: Angular
layout: post
---

在这篇文章中会介绍以下内容：

- 什么是 Tree Shaking，以及它对性能的影响。

- 如何让 Angular(6.0+) 的 Service 实现 Tree Shaking。

- 在 webpack4 的项目中，怎么实现 Tree Shaking。


在文章【[Angular：如何在Angular(8.0)中配置Webpack](https://limeii.github.io/2019/08/angular-customize-webpack/)】提到了 Angular 内置的编译打包方式会执行 Tree Shaking，可以提高 Angular 应用的性能。那我们先看下什么是 Tree Shaking。

## Tree Shaking

Tree Shaking 是指在编译打包过程中，会把那些定义好的代码但是又没有被调用的代码去掉，不把这些没用到的代码打包到最后的 bundle 文件里，从而可以减小 bundle 文件的体积，提高应用性能。可以把整个应用想象成一棵树，function/component/service/lib 好比是树叶，而那些定义但又没有被调用的 function/component/service/lib 好比是枯树叶，Tree Shaking 就是把那些枯树叶从树上摇下去。


Angular 内置的打包方式在```ng build --prod```会启用 Tree Shaking，在```ng build```开发模式下编译不会用 Tree Shaking。我们可以对比 Angular 项目的开发和生产编译结果，来看看 Tree Shaking 的效果，具体代码可以在【[angular-performance](https://github.com/LiMeii/angular-performance)】查看。


首先定义一个 typescript 的 module，export 两个 function：square，cube；具体代码如下：

```ts
// common-utility.ts
export function square(x) {
    console.log("this is square function in commonutility");
    return x * x;
}
export function cube(x) {
    console.log("this is cube function in commonutility");
    return x * x * x;
}
```

然后在 DashboardComponent 中只引用 cube 方法：

```ts
//dashboard.module.ts
import { Component, OnInit } from "@angular/core";
import { cube } from "../../shared/common-utility";

@Component({
    templateUrl: "./dashboard.component.html"
})

export class DashboardComponent implements OnInit {
    ngOnInit() {
        console.log("here is the cube result " + cube(3));
    }
}
```
用```ng serve```把项目跑起来以后，在 console 里会有以下的输出：

```
this is cube function in commonutility
dashboard.component.ts:12 here is the cube result 27
```

之前说过在```ng build```开发模式下编译不会用 Tree Shaking，所以编译以后的 bundle 文件里会有```square```方法的源码，我们```ng build```编译代码看下结果：


在编译后的 bunlde 文件：modules-dashboard-dashboard-module-es5.js 里，我们能看到如下代码，没有 Tree Shaking，```square(x)```方法在整个项目里都没有被调用，但是编译打包的时候，还是把整个方法放进了最后的 bundle 文件里，在 bundle 文件的相关代码如下：

```js
/***/ "./src/app/shared/common-utility.ts":
/*!******************************************!*\
  !*** ./src/app/shared/common-utility.ts ***!
  \******************************************/
/*! exports provided: square, cube */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, "square", function() { return square; });
/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, "cube", function() { return cube; });
function square(x) {
    console.log("this is square function in commonutility");
    return x * x;
}
function cube(x) {
    console.log("this is cube function in commonutility");
    return x * x * x;
}

/***/ })
```

我们再来试一下```ng build --prod```启用 Tree Shaking，因为```ng build --prod```不仅会做 Tree Shaking 还会做 Minification 和 Uglification，所以最后的编译代码都是被处理过的。不能直观的看代码，但是在```square```和```cube```里都有 console log，我们可以通过在最后编译 bundle 文件：3-es5.55d98a7c1ad4d4f5ce12.js 里用关键字‘commonutility’来搜索查看。结果如下：

![angular-tree-shaking](https://limeii.github.io/assets/images/posts/angular/angular-tree-shaking01.png){:height="100%" width="100%"}

"this is square function in commonutility"并没有在最后的 bundle 文件里，说明启用 Tree Shaking 之后不会把 square 打包进最后的 bundle 文件里。

## 如何让 Angular(6.0+) 的 Service 实现 Tree Shaking

Angular6 发布了一项新功能：Tree Shakeable Providers，这个新的功能是用来让 Angular 中的 service 也可以 Tree Shaking。在此之前，Angular 中的 service 都是依赖注入的，service 都是通过 @Injectable 装饰器定义，比如定义一个 AService 如下：

```ts
import { Injectable } from "@angular/core";

@Injectable()
export class AService {
    constructor() {
        console.log("this is Aservice, no matter using or not, Aservice will awalys be bundled.");
    }
}
```
在 component/module 里引用 service 都需要先 import 然后再 providers 元数据里加上对应的 service，比如在 AppModule 引入 AService 的写法如下：

```ts
......
import { AService } from "./shared/service/a.service"

@NgModule({
  .....
  providers: [AService],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

Angular 在编译的时候，因为已经有```import { AService }```语句，那么不管这个 AService 有没有被调用，对 Angular 来说并不好区分 AService 是否被真正调用，所以不管是否调用，最后 AService 都会被打包进 bundle 文件，没办法做 Tree Shaking。


Angular6.0 发布的新功能：Tree Shakeable Providers，就是用来解决这个问题，通过```providedIn ```元数据选项定义该 service 是用在哪个 module 里，然后在对应的 module 里就不需要显示```import { ServiceName }```，没有 import 语句以后，那么 Angular 在编译的时候就很很容易判断 service 到底有没有被调用，从而可以很容易实现对 service 做 Tree Shaking。我们具体来看看代码示例：


定义一个 BService，```providedIn: "root"```表示 BService 用在了 root module（AppModule），然后在 AppComponent 调用 BService，代码如下：

```ts
//BService
import { Injectable } from "@angular/core";

@Injectable({
    providedIn: "root"
})

export class BService {
    constructor() {
        console.log("this is Bservice, there has one component using this, so Bservice will be bundled");
    }
}
```

```ts
//AppComponent
import { Component } from "@angular/core";

import { BService } from "./shared/service/b.service";

@Component({
  selector: "app-root",
  templateUrl: "./app.component.html",
  styleUrls: ["./app.component.css"]
})
export class AppComponent {
  title = "angular-performance";
  constructor(private bService: BService) { }
}
```

为了对比，我们在定义一个 CService，```providedIn: "root"```表示 CService 用在了 root module（AppModule），但是不再任何地方调用 CService，代码如下：

```ts
import { Injectable } from "@angular/core";

@Injectable({
    providedIn: "root"
})
export class CService {
    constructor() {
        console.log("this is Cservice, there has no component using this, so Cservice won't be bundled.");
    }
}
```

再用```ng build --prod```编译打包，service 文件会打包到```main-es5.396328bc020fc5aa2168.js```bundle 文件里，同样通过关键字搜索的方式来看下结果：


搜索：AService，可以看到 AService 没有用```providedIn: "root"```，而且没有被调用，有 Tree Shaking，但是因为在 AppModule 有```import { AService }```，不管是否被调用都会打包进最后的 bundle 文件。
![angular-tree-shaking](https://limeii.github.io/assets/images/posts/angular/angular-tree-shaking02.png){:height="100%" width="100%"}

搜索：BService，可以看到 BService 用```providedIn: "root"```，在 AppModule 不需要```import { BService }```，并且在 AppComponent 里调用，有 Tree Shaking，会被打包进最后的 bundle 文件。
![angular-tree-shaking](https://limeii.github.io/assets/images/posts/angular/angular-tree-shaking03.png){:height="100%" width="100%"}

搜索：CService，可以看到 CService 用```providedIn: "root"```，在 AppModule 不需要```import { CService }```，但是没有被调用，有 Tree Shaking，不会被打包进最后的 bundle 文件。 
![angular-tree-shaking](https://limeii.github.io/assets/images/posts/angular/angular-tree-shaking04.png){:height="100%" width="100%"}

## 在 webpack4 的项目中，怎么实现 Tree Shaking

如果不用 Angular 内置的打包方式做 Tree Shaking，那么在 webpack4 怎么实现 Tree Shaking？在 webpack 官方文档：【[Webpack Tree Shaking](https://webpack.js.org/guides/tree-shaking/)】有介绍结合 ```"sideEffects": false```和 ```mode: "production"```实现 Tree Shaking，我尝试用当前最新的 webpack 版本：```webpack@4.39.2```搭了一个项目，尝试在 webpack4 中使用 Tree Shaking，发现跟官方文档介绍还是有点出入。


首先用```npm init```初始化了一个项目，源码在这里：【[webpack4-practice](https://github.com/LiMeii/webpack4-practice)】，然后安装 webpack/webpack-cli 等等包。配置好 webpack config 文件。具体如下：

【[webpack4-practice/config/webpack.common.config.js](https://github.com/LiMeii/webpack4-practice/blob/master/config/webpack.common.config.js)】


【[webpack4-practice/config/webpack.dev.config.js](https://github.com/LiMeii/webpack4-practice/blob/master/config/webpack.dev.config.js)】


【[webpack4-practice/config/webpack.prod.config.js](https://github.com/LiMeii/webpack4-practice/blob/master/config/webpack.prod.config.js)】

在根目录下，创建如下文件：

```
|----scr
      |----index.html
      |----main.js
      |----shared
           |----common-utility.js

```

common-utility.js 代码如下：

```js
export function square(x) {
    console.log("this is square function in commonutility");
    return x * x;
}

export function cube(x) {
    console.log("this is cube function in commonutility");
    return x * x * x;
}
```

在 main.js 中调用 cube 方法，具体代码如下：

```js
import { cube } from "./shared/common-utility";

console.log("the result for cube(5) is " + cube(5));
```
<blockquote>
<p>
此时并没有按官方文档介绍的那样在 package.json 文件中设置"sideEffects": false。只是对 dev 和 prod 的 mode 分别设置 mode:"development" 和 mode:"production"。
</p>
</blockquote>

运行```npm run build:dev```，在打包出来的 bundle 文件：main.07bdac8643ee4eaf7a5d.js 中可以看到```square```方法虽然没有调用，但还是被打包进来了，说明```mode:"development"```模式下不会 Tree Shaking，


运行```npm run build:prod```，在打包出来的 bundle 文件：main.a60a09c27b85b5bcc81f.js 中只能看到```cube```方法被打包，说明```mode:"production"```模式下，webpack4 默认做 Tree Shaking。


接着按官方文档的介绍，在 package.json 文件中设置 sideEffects：

```json
"name": "webpack4-practice",
"sideEffects": false
```
或者是：

```json
"name": "webpack4-practice",
  "sideEffects": [
    "./src/shared/common-utility.js"
  ]
```

最后编译的结果跟上面不用 sideEffects 的效果一模一样，说明至少在```webpack@4.39.2```版本中，只要设置了```mode:"production"```模式就可以执行 Tree Shaking，跟```sideEffects```没什么关系。

<blockquote>
<p>
需要注意的是，Webpack 内置的 Tree Shaking 只对 ES6 module 语法有用，对那些用 Babel 把 ES6 modules 编译成 CommonJS modules 不起作用。
</p>
</blockquote>


## 其他 Tree Shaking 工具

- [Rollup](https://github.com/rollup/rollup)
- [Google Closure Compiler](https://github.com/google/closure-compiler)