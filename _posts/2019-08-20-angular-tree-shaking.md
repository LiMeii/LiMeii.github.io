---
title: Angular：Tree Shaking
tags: Angular
layout: post
---

在这篇文章中会介绍以下内容：

- 什么是Tree Shaking，以及它对性能的影响。

- 如何让angular(6.0+)的Service实现Tree Shaking

- 在不用angular内置打包编译方式的时候，怎么结合webpack实现Tree Shaking。


在文章【[Angular：如何在Angular(8.0)中配置Webpack](https://limeii.github.io/2019/08/angular-customize-webpack/)】提到了Angular内置的编译打包方式会执行Tree Shaking，可以提高angular应用的性能。那我们先看下什么是Tree Shaking。

# Tree Shaking

Tree Shaking是指在编译打包过程中，会把那些定义好的代码但是又没有被调用的代码去掉，不把这些没用到的代码打包进最后的bundle文件里，从而可以减小bundle文件的体积。可以把整个应用想象成一棵树，源码/component/service/lib好比是树叶，而那些定义但又没有被调用的function/component/service好比是枯树叶，Tree Shaking就是把那些枯树叶从树上摇下去。


angular内置的打包方式在```ng build --prod```会启用Tree Shaking，在```ng build```开发模式下编译不会用Tree Shaking。我们可以用angular的项目来看看Tree Shaking的效果，具体代码可以在【[angular-performance](https://github.com/LiMeii/angular-performance)】查看。


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

```js
this is cube function in commonutility
dashboard.component.ts:12 here is the cube result 27
```

之前说过在```ng build```开发模式下编译不会用Tree Shaking，所以编译以后的bundle文件里会有```square```方法的源码，我们```ng build```编译代码看下结果：


在编译后的bunlde文件：modules-dashboard-dashboard-module-es5.js里，我们能看到如下代码，没有Tree Shaking，```square(x)```方法在整个项目里都没有被调用，但是编译打包的时候，还是把整个方法放在了最后的bundle文件里。

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

**未完待续**