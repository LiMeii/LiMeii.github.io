---
title: cache busting
layout: post
---

# 浏览器缓存问题

浏览器有一个行为就是缓存已经下载过的资源，这个行为可以在缓存有效期内让页面加载更快。
但同时也会有一个弊端，就是服务器资源更新以后，用户可能看不到最新页面。


本文会介绍cache的利与弊，以及怎么解决浏览器缓存问题。

#### 什么是浏览器缓存

如果用户第一次访问某一个网站，所有的资源都是从服务端download下来的，不存在缓存。
以chrome浏览器为例子，F12打开DevTools，可以看到如下图片所示: 加载当前页面有4个requests一共526kb，全是从服务器上下载来。


![第一次访问]( https://limeii.github.io/assets/images/posts/cache-firstload.png){:height="70%" width="70%"}


接着直接刷新页面，如下图所示，在chrome DevTools中可以看到，2requests，并且meii.jpg是从memory cache中拿到的。


![刷新页面]( https://limeii.github.io/assets/images/posts/cache-refreshload.png){:height="70%" width="70%"}


                        