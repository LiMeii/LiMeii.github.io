---
title: webpack(3)：代码切割
tags: Webpack
layout: post
---


在上一篇文章【[webpack：打包后的bundle文件里到底有什么](/2018/10/webpack-what-in-bundle) 】这篇文章解释了什么是bundle文件以及bundle文件里的具体内容。
也提到过单单只是有多少个入口文件，打包出多少个bundle文件这种方式，会导致代码冗余，同一模块代码会被重复打包到不同的bundle文件中。


为了解决这个问题，我们需要把重复共用代码提取出来，放在单独的文件中，其他bundle有引用公用代码只需要加载这个单独文件就可以了。

**提取入口文件中公用的代码，这种行为就是代码切割，生成的独立文件就是chunk文件。**

下面就来介绍如何在 **webpack3** 中进行代码切割。

## CommonsChunkPlugin

在webpack3中是通过CommonsChunkPlugin实现公用代码切割，具体用法如下：

源码在这里：[angular-seed-project](https://github.com/LiMeii/angular-seed-project).


a 和 c 都引用了b文件代码
```js
//a.js
var b = require('./b.js');
console.log('this is a.js file');
b.b();
```
```js
//b.js
exports.b = function () {
    console.log('this is b.js file')
};
```

```js
//c.js
var b = require('./b.js');
console.log('this is c.js file');
b.b();
```

**第一种方式，```new webpack.optimize.CommonsChunkPlugin({name: 'commons'})```**

```js
//webpack.bundle.js
module.exports = {
    entry: {
        'a': './src/app/bundle/a.js',
        'c': './src/app/bundle/c.js'
    },
    output: {
        path: path.join(__dirname, '../build-bundle'),
        filename: 'js/[name].bundle.js'
    },
    plugins: [
        new CleanWebpackPlugin(['./build-bundle'], { root: path.join(process.cwd(), '') }),
        new webpack.optimize.CommonsChunkPlugin({ name: 'commons' }),
    ]
};
```
最后生成三个bundle文件：a.bundle.js、c.bundle.js、commons.bundle.js，具体代码如下：

```js
//a.bundle.js
webpackJsonp([1], [
        /* 0 */,
        /* 1 */
        (function (module, exports, __webpack_require__) {

        // this is use to analyse what's in bundle file

        var b = __webpack_require__(0);

        console.log('this is a.js file');

        b.b();
})
], [1]);
```
```js
//c.bundle.js
webpackJsonp([2], {
        /*2*/
        (function (module, exports, __webpack_require__) {

            var b = __webpack_require__(0);

            console.log('this is c.js file');

            b.b();
})

}, [2]);
```
```js
//commons.bundle.js
(function (modules) { // webpackBootstrap
    //在这里省略了webpack生成的代码
})
([
    /* 0 */
    (function (module, exports) {
            // this is use to analyse what's in bundle file
            exports.b = function () {
                console.log('this is b.js file')
            };
        })
]);
```

从上面代码可以看出，common.bundle.js里面包含了b.js的代码并且module id 为0，a.bunlde.js c.bundle.js文件分别通过module id对b文件实现了加载。

<blockquote>
<p>
CommonsChunkPlugin定义了公用代码需要放到commons.bundle.js文件中，在webpack打包过程中，发现没有commons这个bundle文件，会新创建这个文件，并且把入口文件a.js和c.j这个文件中共用代码（b.js）抽取出来放到commons.bundle.js文件中。a 和 c budnle文件中只保留自己的代码。
</p>
</blockquote>

**第二种方式，```new webpack.optimize.CommonsChunkPlugin({ name: 'c' })```**

我们把commons换成c，最后生成两个bundle文件 a.bunlde.js、c.bundle.js

```js
//a.bundle.js
webpackJsonp([1], [
        /* 0 */
        (function (module, exports, __webpack_require__) {

        // this is use to analyse what's in bundle file

        var b = __webpack_require__(0);

        console.log('this is a.js file');

        b.b();
})
], [1]);
```

```js
//c.bundle.js
(function (modules) { // webpackBootstrap
    // 这里省略webpack生成的代码
})
    ([
        /* 0 */
        (function (module, exports) {
            // this is use to analyse what's in bundle file
            exports.b = function () {
                console.log('this is b.js file')
            };
        }),
        /* 2 */
        (function (module, exports, __webpack_require__) {

            var b = __webpack_require__(0);

            console.log('this is c.js file');

            b.b();

        })
    ]);
```
从上面的bundle文件代码可以看出，公用代码模块（b.js）被放到 c.bundle 文件中了。

<blockquote>
<p>
CommonsChunkPlugin定义了公用代码需要放到c.bundle.js文件中，在webpack打包过程中，发现已经有c.bundle.js文件，会把入口文件a.js和c.j这个文件中共用代码（b.js）抽取出来放到c.bundle.js文件中。a.budnle文件中只保留自己的代码。
</p>
</blockquote>

关于CommonsChunkPlugin其他属性的应用可以参考 [webpack 代码切割官方文档](https://webpack.js.org/plugins/commons-chunk-plugin/)