---
title: 浏览器缓存问题
tags: 问题
layout: post
---


浏览器有一个行为就是缓存已经下载过的资源，缓存静态资源，不需要每次从服务器下载文件，这在很大程度提高了性能。
但同时也会有一个弊端，就是服务器资源更新以后，用户可能看不到最新页面。


这篇文章就是介绍cache的利与弊，以及怎么解决浏览器缓存问题。

## 什么是浏览器缓存

如果用户第一次访问某一个网站，所有的资源都是从服务端download下来的，不存在缓存。
以chrome浏览器为例子，F12打开DevTools到network，可以看到如下图片所示: 加载当前页面有4个requests一共526kb，全是从服务器上下载来。


![第一次访问]( https://limeii.github.io/assets/images/posts/issues/cache-firstload.png){:height="100%" width="100%"}


接着直接刷新页面，如下图所示，在chrome DevTools中可以看到，2个requests，并且meii.jpg是从memory cache中拿到的。


![刷新页面]( https://limeii.github.io/assets/images/posts/issues/cache-refreshload.png){:height="100%" width="100%"}

<blockquote><p>
chrome 浏览器要不是从memory cache或disk cache里拿文件，因为没有关浏览器而且时间比较短所示上图中还是从memory cache里拿得
</p></blockquote>

像静态文件，比如JS/图片/CSS文件可以被缓存起来，那么下次到同样的网页的时候，不需要从服务器上下载而是从缓存中拿，大大的提高了访问网页性能。

## 那么浏览器怎么知道要缓存呢？

从服务器返回的http response header中，通常有这几个属性用来标识缓存：
- ETag
- Cache-Control
- Expires
- Last-Modified

### ETag

ETag 通常是服务器生成的一段hash validation token，那么浏览器在后续的request中会把这个token带上，如果ETag一样就返回空的body，response code为304 (not modified)，这时候浏览器请求的response可以直接从cache中拿。

### Cache-Control

Cache-Control本身有好几个属性可以用来设置cache行为

```html
Cache-Control: public
```
public 表示资源可以被缓存在任何可以被缓存的地方（CDN,浏览器等等）

```html
Cache-Control: private
```
private 表示资源只可以被浏览器缓存

```html
Cache-Control: no-store
```
no-store表示浏览器需要每次都从服务器拿资源

```html
Cache-Control: no-cache
```
no-cache并不是不缓存的意思，而是告诉浏览器要缓存文件，但是每次需要跟服务器确认是最新文件以后才能用，一般是看ETag跟服务器是否匹配。

```html
Cache-Control: max-age=60
```
表示资源文件只能被缓存一分钟

```html
Cache-Control: must-revalidate
```
表示只有校验缓存里是最新文件才能用缓存里的版本

### Expires

```html
Expires: Wed, 25 Sep 2018 21:00:00 GMT
```
这个属性是http1.0里的，表示缓存里的文件在这个属性对应时间以后过期。
需要注意的是，如果header有max-age这个属性的时候，Expires这个属性会被忽略。

### Last-Modified

```
Last-Modified: Mon, 12 Sep 2018 14:45:00 GMT
```
这个属性也是http1.0里的，表示当前资源最后更改的时间

这就浏览器怎么知道哪些文件需要被缓存。


## 缓存的弊端


从缓存里拿文件肯定是要比从服务器上拿性能要高，但是也会有弊端。
比如一分钟前一个用户刚访问一个网站，这个时候浏览器缓存了一部分静态文件，这个时候这个网站发布了新版本包含一些新功能，那么在缓存不过期的情况下，这个用户永远都没法看到新版本新功能，除非这个用户强制清除他本地的缓存。
新版本发布以后，每次都需要用户清缓存，显然不合理。

### 那么如何解决这个问题呢？

通常来讲，引用js或者其他静态文件是这样的：
```html
<script src="../js/app.min.js">
```
**第一种方案就是每次手动给这个文件加个版本号**

```html
<script src="../js/app-v2.min.js">
```

**第二种方案就是每次对应静态文件里有内容改动的时候，自动加一段hash到静态文件名里**
```html
<script src="../js/app-ef1d8c670o00b204e9800998ecfere.min.js">
```

**第三种方案是在后面加一段query string**
```html
<script src="../js/app.min.js？cb=3424243234">
```
但是第三种方案不推荐使用，在一些 [proxy serve](https://gtmetrix.com/remove-query-strings-from-static-resources.html) 有一些问题。

通常是用第二种方式来解决缓存问题，可以参考文章：【[webpack：如何解决浏览器缓存问题](/2018/10/webpack-caching)】。