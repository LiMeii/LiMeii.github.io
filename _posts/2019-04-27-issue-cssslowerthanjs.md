---
title: js
layout: post
---

# 生产环境里CSS渲染比JS要慢导致的问题
<div class="title-meta">
    <span><img class="title-category-img" style="margin-top: 0.2em;" src="../../../assets/images/categories/css3.svg" alt="css"></span>
    <span><a class="github-link" href="/2018/09/19/CSS.html">CSS</a></span>
    <span class="title-bullet">•</span>
    <span>Apr 27, 2019</span>
    <span class="title-bullet">•</span>
    <span>by <a class="github-link" href="http://github.com/limeii" title="http://github.com/limeii">Mei</a></span>
</div>


### 问题描述

公司项目是一个基于Angular4.2.4的项目，CSS文件是由美国UX team提供的，页面也有一些动画效果，是UX team基于JQUERY写的，所有业务代买都是基于Angular typescript实现的，这个项目框架是在index页面引用UX team提供的CSS和 JS文件。然后在写具体业务页面的时候直接用UX team的CSS并且自带动画效果。


在此基础上印度同事这样写了一段代码: 用户点击一按钮的时候，会执行美国team的JQUERY方法将这个按钮的样式从CSS1到CSS2， 然后在他的业务typescript代码里在根据这个CSS是css1还是css2把状态传给后端API更新数据库里的用户状态。


在开发环境里一切正常，上QA测了好几轮也没什么问题。relase night上生产后，QA在测试这个功能的时候发现，点击这个按钮用户状态有时候能正常更新，有时候根本不跟新，页面CSS的样式也不对。


出生产事故啦，QA紧张得不得了，我跑过去试了几次，确实有问题，发现用户点击按钮，首先按钮样式根本不会从css1更新到css2，导致API传给数据库的状态永远是css1对应的值。


### 问题分析

**QA两个环境五个QA测了这么多次都没发现问题，先到QA环境里看看能不能重新？**

登入QA环境以后，试了好多次没问题。好奇怪，开发环境呢？把开发环境RUN起来，发现开发环境也没有办法重新？找到印度同事写的那段代码，看了下逻辑好像没问题，唯一奇怪的就是，他的业务代码需要依赖css，而控制css改变代码是在美国UX team提供的JQUERY代码里。看到这里就明白问题的根源是什么啦。JQUERY那段操作css从css1改到css2那段代码根本没执行，或者是执行得比业务调用API的那段JS慢导致的。在JQUERY代码里直接把对这个样式操作的方法直接注释掉，然后发现开发环境里百分百重现了这个问题。

一开始还挺奇怪的，JS不是单线程的吗？按道理这段代码应该有执行顺序的呀，不可能会慢的呀。不过css好像是属于DOM操作，是页面渲染？我记得页面渲染浏览器有单独的页面渲染线程？是不是页面渲染线程比JS线程慢导致的？