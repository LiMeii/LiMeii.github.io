---
title: why need node proxy for front-end Dev
layout: post
---

# 用node代理解决前后端联调跨域问题
<div class="title-meta">
    <span><img class="title-category-img" style="margin-top: 0.2em;" src="../../../assets/images/categories/js.svg" alt="js"></span>
    <span><a class="github-link" href="/2018/09/19/js.html">CSS</a></span>
    <span class="title-bullet">•</span>
    <span>Apr 21, 2019</span>
    <span class="title-bullet">•</span>
    <span>by <a class="github-link" href="http://github.com/limeii" title="http://github.com/limeii">Mei</a></span>
</div>

最近做的项目，后端API是用的.NET， 前端开发用的是Angular4, 是一个前后端分离的项目。在本地开发的时候，我用node express搭了一个proxy用来解决前后端跨域联调的问题。


最近项目来了新人，每次带新人开发的时候，都会问到为什么要用这个node proxy，本地开发的时候直接调用后端DEV-INT服务器上的API不就好了吗？ 每次都解释这个是用来解决联调跨域的，刚工作的同事基本都是懵比状态，有经验的一般会问，不是在服务器端设置好CROS就可以了，但是对于代理实现跨域并不是很了解。在这篇文章里就会解释为什么用要跨域以及如果用node proxy解决联调跨域问题。



