---
title: webpack：打包后的bundle文件里到底有什么
tags: Webpack
layout: post
---


webpack把 ts js html css imgs等文件都统统打包成bundle.js文件，只知道所有静态文件都在bundle文件里，但是具体bundle里面有什么呢？

## 一个文件：a.js，一个入口文件，生成一个bundle文件

源码在这里：[angular-seed-project](https://github.com/LiMeii/angular-seed-project).

``` js
// a.js

console.log('this is a.js file');
```
```js
// webpack.bundle.js

module.exports = {
    entry: {
        'a': './src/app/bundle/a.js',
    },
    output: {
        path: path.join(__dirname, '../build-bundle'),
        filename: 'js/[name].bundle.js'
    },
    plugins: [
        new CleanWebpackPlugin(['./build-bundle'], { root: path.join(process.cwd(), '') })
    ]
};
```
最终生成的 a.bundle.js 文件如下：
```js
//a.bundle.js

/******/ (function (modules) { // webpackBootstrap
/******/ 	// The module cache
/******/ 	var installedModules = {};
/******/
/******/ 	// The require function
/******/ 	function __webpack_require__(moduleId) {
/******/
/******/ 		// Check if module is in cache
/******/ 		if (installedModules[moduleId]) {
/******/ 			return installedModules[moduleId].exports;
            /******/
}
/******/ 		// Create a new module (and put it into the cache)
/******/ 		var module = installedModules[moduleId] = {
/******/ 			i: moduleId,
/******/ 			l: false,
/******/ 			exports: {}
            /******/
};
/******/
/******/ 		// Execute the module function
/******/ 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
/******/
/******/ 		// Flag the module as loaded
/******/ 		module.l = true;
/******/
/******/ 		// Return the exports of the module
/******/ 		return module.exports;
        /******/
}
/******/
/******/
/******/ 	// expose the modules object (__webpack_modules__)
/******/ 	__webpack_require__.m = modules;
/******/
/******/ 	// expose the module cache
/******/ 	__webpack_require__.c = installedModules;
/******/
/******/ 	// define getter function for harmony exports
/******/ 	__webpack_require__.d = function (exports, name, getter) {
/******/ 		if (!__webpack_require__.o(exports, name)) {
/******/ 			Object.defineProperty(exports, name, {
/******/ 				configurable: false,
/******/ 				enumerable: true,
/******/ 				get: getter
    /******/
});
            /******/
}
        /******/
};
/******/
/******/ 	// getDefaultExport function for compatibility with non-harmony modules
/******/ 	__webpack_require__.n = function (module) {
/******/ 		var getter = module && module.__esModule ?
/******/ 			function getDefault() { return module['default']; } :
/******/ 			function getModuleExports() { return module; };
/******/ 		__webpack_require__.d(getter, 'a', getter);
/******/ 		return getter;
        /******/
};
/******/
/******/ 	// Object.prototype.hasOwnProperty.call
/******/ 	__webpack_require__.o = function (object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
/******/
/******/ 	// __webpack_public_path__
/******/ 	__webpack_require__.p = "";
/******/
/******/ 	// Load entry module and return exports
/******/ 	return __webpack_require__(__webpack_require__.s = 0);
    /******/
})
/************************************************************************/
/******/([
/* 0 */
/***/ (function (module, exports) {

        // this is use to analyse what's in bundle file

        console.log('this is a.js file');

        /***/
})
/******/]);

```

<blockquote>
<p>
1. 从最后的bundle文件可以看出来，整个bundle文件是一个自执行表达式，传入参数是一个数组，数组里有一个funciton，这个function里面包含了a.js里面的内容。
</p>
<p>
2. webpack打包以后，每个模块都有一个独一无二的id，0 1 2 3......其实就是自执行表达式传入参数数组的索引。
</p>
<p>
3. 在这个自执行表达式中有一个闭包，__webpack_require__是模块加载函数，通过模块id找到对应模块，加载过的模块会放入cache中，下次就直接从cache调用不用重复加载。
</p>
</blockquote>

是不是很简单，所有模块都以参数数组的形式传入到自执行表达式中。

## 两个文件：a.js和b.js，两个入口文件，打包成两个bundle文件

``` js
//a.js
console.log('this is a.js file');
```
``` js
//b.js
console.log('this is b.js file');
```

```js
//webpack.bundle.js
module.exports = {
    entry: {
        'a': './src/app/bundle/a.js',
        'b': './src/app/bundle/b.js'
    },

    output: {
        path: path.join(__dirname, '../build-bundle'),
        filename: 'js/[name].bundle.js'
    }
};
```
最后编译的bundle文件有两个，a.budnle.js和b.bundle.js，从下面代码可以看出来moduleid是唯一不会重复的。

```js
//a.bundle.js
/******/ (function(modules) { // webpackBootstrap
/******/ 	// 省略重复代码
/******/ 	// Load entry module and return exports
/******/ 	return __webpack_require__(__webpack_require__.s = 0);
/******/ })
/************************************************************************/
/******/ ([
/* 0 */
/***/ (function(module, exports) {

// this is use to analyse what's in bundle file

console.log('this is a.js file');

/***/ })
/******/ ]);

```

```js
//b.bundle.js
/******/ (function (modules) { // webpackBootstrap
/******/ 	// 省略重复代码
/******/ 	// Load entry module and return exports
/******/ 	return __webpack_require__(__webpack_require__.s = 1);
    /******/
})
/************************************************************************/
/******/([
/* 0 */,
/* 1 */
/***/ (function (module, exports) {

        // this is use to analyse what's in bundle file

        console.log('this is b.js file');

        /***/
})
/******/]);
```

## 两个文件：a.js和b.js，a引用b文件，一个入口，生成一个bundle文件

``` js
//a.js
var b = require('./b.js');

console.log('this is a.js file');

b.b();
```
``` js
//b.js
exports.b = function () {
    console.log('this is b.js file')
};
```
```js
//webpack.bundle.js
module.exports = {
    entry: {
        'a': './src/app/bundle/a.js'
    },
    output: {
        path: path.join(__dirname, '../build-bundle'),
        filename: 'js/[name].bundle.js'
    }
};
```

最终生成的一个a.bundle.js，从以下代码中可以看出，自执行函数的传入参数数组变成了两个，分别对应a.js b.js的内容，不同的是a对b的引用变成了：var b = __webpack_require__(1);

```js
//a.bundle.js
/******/ (function (modules) { // webpackBootstrap
/******/ 	// 省略重复代码
/******/
/******/ 	// Load entry module and return exports
/******/ 	return __webpack_require__(__webpack_require__.s = 0);
    /******/
})
/************************************************************************/
/******/([
/* 0 */
/***/ (function (module, exports, __webpack_require__) {

        // this is use to analyse what's in bundle file

        var b = __webpack_require__(1);

        console.log('this is a.js file');

        b.b();

        /***/
}),
/* 1 */
/***/ (function (module, exports) {

        // this is use to analyse what's in bundle file
        exports.b = function () {
            console.log('this is b.js file')
        };

        /***/
})
/******/]);
```

以上就是最后bundle文件里的具体内容，但是这样这样打包会有以下问题：
<blockquote>
<p>
如果多个文件a.js、b.js 、c.js、d.js，
a、b、c 文件同时引用了d文件，
入口文件为a、b、c, 那么d就会被重复打包到a.bundle.js、b.bundle.js、c.bundle.js ，会有代码冗余。
</p>
</blockquote>
【[webpack 代码切割](/2018/10/webpack-code-splitting)】这篇文章介绍了如何解决上述问题。