---
title: webpack：如何解决浏览器缓存问题
tags: 问题
layout: post
---

在这篇文章里会介绍怎么在webpack中解决浏览器缓存问题。

<blockquote>
<p>
这篇文章是基于 webpack3.10.0
</p>
<p>
默认打包的时候是直接把CSS inline到bundle文件里面
</p>
</blockquote>

## 缓存
webpack打包以后的文件一般都是：app.bundle.js、vendor.bundle.js，每次发布以后，如果用户浏览器缓存没有过期，加上文件名字相同，浏览器不会从服务器下载最新文件，导致用户不能看到新发布的功能。

**这就是我们所说的浏览器缓存问题。**

解决方案就是，每次如果文件有改动，那么在发布的时候，就让对应bundle文件名不一样。这样，不管浏览器缓存过没过期，都会强制浏览器从服务器拿最新代码。


按照这个思路，一旦有文件改动，不管是ts，js，img还是css文件有改动，打包的时候，对应的bundle文件名也要有变化。才能强制浏览器去服务器拿最新文件，从而解决缓存问题。

## chunkhash

在webpack中，build生成bundle文件的时候，在bundle文件名中加上hashcode，一旦有改动，那么bundle文件名中的hashcode也不一样，这样就保证了每次发布，bundle文件名不一样。

<blockquote>
<p>
加上hashcode以后的bundle文件如下：
</p>
<p>
app.f68357fc8e99ece57afe.bundle.js
</p>
<p>
vendor.0d5cad4949ab26faaa39.bundle.js
</p>
</blockquote>

具体用法是在webpack配置文件的output filename中加上 ```[chunkhash]```：
```js
  //webpack.prod.js
  output: {
    path: path.join(process.cwd(), 'dist'),
    publicPath: "/dist/",
    filename: 'js/[name].[chunkhash].bundle.js',
  }
```
完整代码可参考：[angular-seed-project](https://github.com/LiMeii/angular-seed-project)


在terminal 跑命令行```npm run build:prod```之后生成文件如下：

![webpack-caching-chunkhash](https://limeii.github.io/assets/images/posts/webpack/webpack-caching-chunkhash.png){:height="100%" width="100%"}

<blockquote>
<p>
里面有如下三个文件，是因为用到angular lazy loading所产生的chunk文件：
</p>
<p>0.068716ea07094f619891.bundle.js</p>
<p>1.3392ab62515e716550b2.bundle.js</p>
<p>2.f69ff655a0890c6a5b91.bundle.js</p>
</blockquote>

除了用chunkhash之外，还可以用hash

## hash

把output filename中的```chunkhash```改为```hash```：
```js
  //webpack.prod.js
  output: {
    path: path.join(process.cwd(), 'dist'),
    publicPath: "/dist/",
    filename: 'js/[name].[hash].bundle.js',
  }
```
在terminal跑命令行```npm run build:prod```之后生成文件如下：

![webpack-caching-hash](https://limeii.github.io/assets/images/posts/webpack/webpack-caching-hash.png){:height="100%" width="100%"}

从最终bundle文件名可以看出，所有bundle文件hashcode都是一样的。


我们来测试下：改动app.component.html文件，使用```hash```的方式，重新build以后，bundle文件如下：

<blockquote>
   <p> 0.64de26c53b0cd5ce3331.bundle.js  </p> 
   <p> 1.64de26c53b0cd5ce3331.bundle.js  </p>  
   <p> 2.64de26c53b0cd5ce3331.bundle.js   </p> 
   <p> vendor.64de26c53b0cd5ce3331.bundle.js  </p> 
   <p> app.64de26c53b0cd5ce3331.bundle.js  </p> 
   <p> polyfills.64de26c53b0cd5ce3331.bundle.js  </p> 
</blockquote>
所有的bundle文件名都改了，但hashcode都一样。



使用```chunkhash```方式，重新build以后，bundle文件如下：

<blockquote>
<p>0.068716ea07094f619891.bundle.js</p> 
<p>1.3392ab62515e716550b2.bundle.js</p> 
<p>2.f69ff655a0890c6a5b91.bundle.js</p> 
<p>vendor.0d5cad4949ab26faaa39.bundle.js </p> 
<p>app.c33736260b829570e8ba.bundle.js </p> 
<p>polyfills.9d98b7f13680b7f15ee4.bundle.js </p> 
</blockquote>

从最终bundle文件名可以看出，只有app.bundle文件的```hashcode```有改动，其他的都跟上一次```contenthash```方式编译结果一致。

## hash与chunkhash的区别：

1. hash 是用来给本次build计算hashcode，所有的编译结果文件中的hashcode都会一样。

2. chunkhash 是用来给每个entry file计算hashcode，每个编译结果文件中的hashcode都是独一无二的。而且entry file任一文件改动，对应的bundle文件hashcode也会改动，否则就保持不变。

<blockquote>
<p><font color="#BF1827">
需要注意的是：hash 或 chunkhash最好只用在生产环境下，如果在开发环境下用，会导致编译变慢。
</font></p>
</blockquote>

## 总结

这篇文章主要是介绍了，hash 和 chunkhash的基本用法，默认打包的时候是直接把CSS inline到bundle文件里面。


我们知道CSS在项目里面改动的次数不多，每次都inline到bundle文件里，会有以下问题：

1.  如果本身CSS文件就很大，会导致bundle文件过大

2. 任何一次非CSS代码改动，发布以后客户端会重新下载文件，虽然CSS没有变化，但是需要每次跟budnle文件一起下载


那么如何解决这个问题呢？


在【[webpack：优化打包CSS文件](/2018/10/webpack-css-extract)】文章中会结合ExtractTextPlugin和CSS介绍更全面的解决缓存问题方案。