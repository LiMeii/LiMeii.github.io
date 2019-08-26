---
title: Angular性能优化：Tree Shaking
tags: Angular
layout: post
---

在这篇文章中会介绍以下内容：

- 什么是Tree Shaking，以及它对性能的影响。

- 如何让angular(6.0+)的Service实现Tree Shaking。

- 在webpack4的项目中，怎么实现Tree Shaking。


在文章【[Angular：如何在Angular(8.0)中配置Webpack](https://limeii.github.io/2019/08/angular-customize-webpack/)】提到了Angular内置的编译打包方式会执行Tree Shaking，可以提高angular应用的性能。那我们先看下什么是Tree Shaking。

## Tree Shaking

Tree Shaking是指在编译打包过程中，会把那些定义好的代码但是又没有被调用的代码去掉，不把这些没用到的代码打包到最后的bundle文件里，从而可以减小bundle文件的体积，提高应用性能。可以把整个应用想象成一棵树，function/component/service/lib好比是树叶，而那些定义但又没有被调用的function/component/service/lib好比是枯树叶，Tree Shaking就是把那些枯树叶从树上摇下去。


angular内置的打包方式在```ng build --prod```会启用Tree Shaking，在```ng build```开发模式下编译不会用Tree Shaking。我们可以对比angular项目的开发和生产编译结果，来看看Tree Shaking的效果，具体代码可以在【[angular-performance](https://github.com/LiMeii/angular-performance)】查看。


首先定义一个typescript的module，export两个function：square，cube；具体代码如下：

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

然后在DashboardComponent中只引用cube方法：

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
用```ng serve```把项目跑起来以后，在console里会有以下的输出：

```
this is cube function in commonutility
dashboard.component.ts:12 here is the cube result 27
```

之前说过在```ng build```开发模式下编译不会用Tree Shaking，所以编译以后的bundle文件里会有```square```方法的源码，我们```ng build```编译代码看下结果：


在编译后的bunlde文件：modules-dashboard-dashboard-module-es5.js里，我们能看到如下代码，没有Tree Shaking，```square(x)```方法在整个项目里都没有被调用，但是编译打包的时候，还是把整个方法放进了最后的bundle文件里，在bundle文件的相关代码如下：

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

我们再来试一下```ng build --prod```启用Tree Shaking，因为```ng build --prod```不仅会做Tree Shaking还会做Minification和Uglification，所以最后的编译代码都是被处理过的。不能直观的看代码，但是在```square```和```cube```里都有console log，我们可以通过在最后编译bundle文件：3-es5.55d98a7c1ad4d4f5ce12.js里用关键字‘commonutility’来搜索查看。结果如下：

![angular-tree-shaking](https://limeii.github.io/assets/images/posts/angular/angular-tree-shaking01.png){:height="100%" width="100%"}

"this is square function in commonutility"并没有在最后的bundle文件里，说明启用Tree Shaking之后不会把square打包进最后的bundle文件里。

## 如何让angular(6.0+)的Service实现Tree Shaking

angular6发布了一项新功能：Tree Shakeable Providers，这个新的功能是用来让angular中的service也可以Tree Shaking。在此之前，angular中的service都是依赖注入的，service都是通过@Injectable 装饰器定义，比如定义一个AService如下：

```ts
import { Injectable } from "@angular/core";

@Injectable()
export class AService {
    constructor() {
        console.log("this is Aservice, no matter using or not, Aservice will awalys be bundled.");
    }
}
```
在component/module里引用service都需要先import然后再providers元数据里加上对应的service，比如在AppModule引入AService的写法如下：

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

angular在编译的时候，因为已经有```import { AService }```语句，那么不管这个AService有没有被调用，对angular来说并不好区分AService是否被真正调用，所以不管是否调用，最后AService都会被打包进bundle文件，没办法做Tree Shaking。


angular6.0发布的新功能：Tree Shakeable Providers，就是用来解决这个问题，通过```providedIn ```元数据选项定义该service是用在哪个module里，然后在对应的module里就不需要显示```import { ServiceName }```，没有import语句以后，那么angular在编译的时候就很很容易判断service到底有没有被调用，从而可以很容易实现对service做Tree Shaking。我们具体来看看代码示例：


定义一个BService，```providedIn: "root"```表示BService用在了root module（AppModule），然后在AppComponent调用BService，代码如下：

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

为了对比，我们在定义一个CService，```providedIn: "root"```表示CService用在了root module（AppModule），但是不再任何地方调用CService，代码如下：

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

再用```ng build --prod```编译打包，service文件会打包到```main-es5.396328bc020fc5aa2168.js```bundle文件里，同样通过关键字搜索的方式来看下结果：


搜索：AService，可以看到AService没有用```providedIn: "root"```，而且没有被调用，有Tree Shaking，但是因为在AppModule有```import { AService }```，不管是否被调用都会打包进最后的bundle文件。
![angular-tree-shaking](https://limeii.github.io/assets/images/posts/angular/angular-tree-shaking02.png){:height="100%" width="100%"}

搜索：BService，可以看到BService用```providedIn: "root"```，在AppModule不需要```import { BService }```，并且在AppComponent里调用，有Tree Shaking，会被打包进最后的bundle文件。
![angular-tree-shaking](https://limeii.github.io/assets/images/posts/angular/angular-tree-shaking03.png){:height="100%" width="100%"}

搜索：CService，可以看到CService用```providedIn: "root"```，在AppModule不需要```import { CService }```，但是没有被调用，有Tree Shaking，不会被打包进最后的bundle文件。
![angular-tree-shaking](https://limeii.github.io/assets/images/posts/angular/angular-tree-shaking04.png){:height="100%" width="100%"}

## 在webpack4的项目中，怎么实现Tree Shaking

如果不用angular内置的打包方式做Tree Shaking，那么在webpack4怎么实现Tree Shaking？在webpack官方文档：【[Webpack Tree Shaking](https://webpack.js.org/guides/tree-shaking/)】有介绍结合 ```"sideEffects": false```和 ```mode: "production"```实现Tree Shaking，我尝试用当前最新的webpack版本：```webpack@4.39.2```搭了一个项目，尝试在webpack4中使用Tree Shaking，发现跟官方文档介绍还是有点出入。


首先用```npm init```初始化了一个项目，源码在这里：【[webpack4-practice](https://github.com/LiMeii/webpack4-practice)】，然后安装webpack/webpack-cli等等包。配置好webpack config文件。具体如下：

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

在main.js中调用cube方法，具体代码如下：

```js
import { cube } from "./shared/common-utility";

console.log("the result for cube(5) is " + cube(5));
```
<blockquote>
<p>
此时并没有按官方文档介绍的那样在package.json文件中设置"sideEffects": false。只是对dev和prod的mode分别设置mode:"development"和mode:"production"。
</p>
</blockquote>

运行```npm run build:dev```，在打包出来的bundle 文件：main.07bdac8643ee4eaf7a5d.js中可以看到```square```方法虽然没有调用，但还是被打包进来了，说明```mode:"development"```模式下不会Tree Shaking，


运行```npm run build:prod```，在打包出来的bundle文件：main.a60a09c27b85b5bcc81f.js中只能看到```cube```方法被打包，说明```mode:"production"```模式下，webpack4默认做Tree Shaking。


接着按官方文档的介绍，在package.json文件中设置sideEffects：

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

最后编译的结果跟上面不用sideEffects的效果一模一样，说明至少在```webpack@4.39.2```版本中，只要设置了```mode:"production"```模式就可以执行Tree Shaking，跟```sideEffects```没什么关系。

<blockquote>
<p>
需要注意的是，Webpack内置的Tree Shaking只对ES6 module语法有用，对那些用Babel把ES6 modules编译成CommonJS modules不起作用。
</p>
</blockquote>
