---
title: webpack code splitting
layout: post
---

# webpack 代码切割
<div class="title-meta">
    <span><img class="title-category-img" src="../../../assets/images/categories/webpack.svg" alt="webpack"></span>
    <span><a class="github-link" href="/2018/09/24/webpack.html">Webpack</a></span>
    <span class="title-bullet">•</span>
    <span>Oct 07, 2018</span>
    <span class="title-bullet">•</span>
    <span>by <a class="github-link" href="http://github.com/limeii" title="http://github.com/limeii">Mei</a></span>
</div>

在介绍webpack代码切割之前，先来介绍一下bundle和chunk文件的区别。

**bundle** 一般来说，有多少个入口文件，就有多少个bundle文件。
```js
    entry: {
        'app': './src/main.ts',
        'vendor': './src/vendor.ts',
        'polyfills': './src/polyfills.ts'
    },
    output: {
        path: path.join(__dirname, './build-dev'),
        filename: 'js/[name].bundle.js',
    }
```
从以上代码来看的话，最后bundle文件有三个，分别是：app.bunlde.js/ vendor.bundle.js/ polyfills.bundle.js。


bundle文件实际上

**chunk**

### 什么是代码切割 - webpack3

### CommonsChunkPlugin - webpack3