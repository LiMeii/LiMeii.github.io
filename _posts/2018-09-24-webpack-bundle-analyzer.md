---
title: webpack：分析和优化webpack bundle文件
tags: Webpack
layout: post
---


在用 webpack 打包的过程当中，曾经把 webpack 官方文档看了不下十遍，对于最终 bundle 文件里具体到底有什么或者某个特定的 module 到底被放到哪一个 bundle 文件里，始终都是一头雾水。
特别是代码切割这一块，看了那么多遍，并且 google 了各类大神对这一块的解释，还是一头雾水。直到我发现 webpack-bundle-analyzer 这个小插件，把它 install 到本地项目里跑起来，一下子就彻底明白了。


今天这篇文章就是给大家介绍 webpack-bundle-analyzer。这个插件是 build 生成项目文件时同时生成一张所有 bundle 文件的动态 treemap 图片。
如下所示：


![treemap img]( https://limeii.github.io/assets/images/posts/webpack/webpack-bundle-analyzer.gif){:height="100%" width="100%"}


在这个树形图片里，会有包含下面的内容：


1. 每个打包以后的 bundle 文件里面，真正包含哪些内容，项目里的 module、js、component、html、css、img 最后都被放到哪个对应的 bunlde 文件里了。

2. 每个 bundle 文件里，列出了每一个的 module、componet、js 具体 size，同时会列出 start size、parsed size、gzip size 这三种不同的形式下到底多大，方便优化。

<blockquote>
<p>start size：原始没有经过 minify 处理的文件大小</p>
<p>parse size：比如 webpack plugin 里用了 uglify，就是 minified 以后的文件大小</p>
<p>gzip size：被压缩以后的文件大小</p>
</blockquote>


基于以上给出的信息，
你就能比较直观的在图片里看到，哪些公用 library 被重复打包到不同的 bundle 文件里，或者是说哪一个过大影响性等等；从而你就可以对你的 webpack 打包方式进行优化。

## 用法

 先在项目里面安装这个 plugin:

```js
npm install --save-dev webpack-bundle-analyzer
```

然后在 webpack config 文件里面加上以下代码:

 ```js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = { 
  plugins: [

     new BundleAnalyzerPlugin()
  ]
};
 ```


 最后 run webpack build 命令，比如```npm run build```，在 build 结束以后，默认会直接在浏览器里把最终的动态 treemap 图片展示出来


 关于更多的用法，可以参照官方文档 [webpack bundle analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer)