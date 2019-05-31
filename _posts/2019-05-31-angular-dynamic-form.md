---
title: angular dynamic form
layout: post
---

# Angular Dynamic Form：动态创建Form表单

<div class="title-meta">
    <span><img class="title-category-img" src="../../../assets/images/categories/angular.svg" alt="Angular"></span>
    <span><a class="github-link" href="/2018/09/19/angular.html">Angular</a></span>
    <span class="title-bullet">•</span>
    <span>May 31, 2019</span>
    <span class="title-bullet">•</span>
    <span>by <a class="github-link" href="http://github.com/limeii" title="http://github.com/limeii">Mei</a></span>
</div>

在web应用里通常会有这样一种场景：比如支付宝信用卡还款，假设支付宝收费标准如下：

##### 普通用户，2000元以内免费，2000-50000收费10元，50000元以上收费15元。
##### 砖石会员，5000元以内免费，5000-50000收费5元，50000元以上收费10元。

现在需要做一个页面，用来专门用来收集这样的收费标准，以后可能需要增加新的收费标准或者修改现有的收费标准。

一帮页面设计是这样的：