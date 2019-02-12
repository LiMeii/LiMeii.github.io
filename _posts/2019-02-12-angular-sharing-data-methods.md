---
title: angular sharing data
layout: post
---

# Angular5：如何在多个组件之间通信

<div class="title-meta">
    <span><img class="title-category-img" src="../../../assets/images/categories/angular.svg" alt="Angular"></span>
    <span><a class="github-link" href="/2018/09/19/angular.html">Angular</a></span>
    <span class="title-bullet">•</span>
    <span>Feb 12, 2019</span>
    <span class="title-bullet">•</span>
    <span>by <a class="github-link" href="http://github.com/limeii" title="http://github.com/limeii">Mei</a></span>
</div>

在Angular组件之间共享数据，有以下四种方式：
#### 父组件至子组件: 通过@Input共享数据
#### 子组件至父组件: 通过@Output EventEmitter共享数据
#### 子组件至父组件: 通过@ViewChild共享数据
#### 不相关组件： 通过service共享数据
