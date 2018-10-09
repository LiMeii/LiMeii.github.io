---
title: webpack css extract
layout: post
---

# 如何利用 webpack 优化打包 CSS 文件
<div class="title-meta">
    <span><img class="title-category-img" src="../../../assets/images/categories/webpack.svg" alt="webpack"></span>
    <span><a class="github-link" href="/2018/09/24/webpack.html">Webpack</a></span>
    <span class="title-bullet">•</span>
    <span>Oct 10, 2018</span>
    <span class="title-bullet">•</span>
    <span>by <a class="github-link" href="http://github.com/limeii" title="http://github.com/limeii">Mei</a></span>
</div>

在 [如何利用 webpack 解决浏览器缓存问题](/2018/10/09/webpack-caching.html) 文章介绍了用chunkhash解决浏览器缓存问题，这篇文章默认是把CSS文件一起打包进 JS bundle文件中。


一般项目里面CSS的改动比较少，如果打包成 JS bundle 文件，再结合chunkhash，每次发布以后，虽然CSS文件没有改动，但是客户端还是需要重新下载这些文件。如果CSS文件过大的话，在一定程度上会影响性能。


接下来就介绍，在打包过程不把CSS inline 到 JS bundle文件，而是直接提取生成单独的CSS文件，如果项目中CSS有更改，提出生成的CSS文件名也会带上不同的hashcode。

### extract-text-webpack-plugin


