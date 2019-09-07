---
title: 环境升级后，rxjs编译失败
tags: 问题
layout: post
---


## 问题描述

最近换电脑了，整个环境都是重新安装的，刚才跑了一个原来的老项目，发现npm install成功，但是npm run start以后会有下面的错误：

![issues-rxjs-compiled-failed03]( https://limeii.github.io/assets/images/posts/issues/rxjs-compiled-failed03.png){:height="100%" width="100%"}

#### 电脑升级后的开发环境如下：

![issues-rxjs-compiled-failed01]( https://limeii.github.io/assets/images/posts/issues/rxjs-compiled-failed.png){:height="100%" width="100%"}

#### 老项目各个包的版本如下：

![issues-rxjs-compiled-failed01]( https://limeii.github.io/assets/images/posts/issues/rxjs-compiled-failed02.png){:height="100%" width="100%"}

## 问题分析

Google了一下，发现有人在github rxjs的Issues里提过类似的bug。


bug地址：[https://github.com/ReactiveX/rxjs/issues/4512](https://github.com/ReactiveX/rxjs/issues/4512)


而且给出了解释和解决方案：

![issues-rxjs-compiled-failed04]( https://limeii.github.io/assets/images/posts/issues/rxjs-compiled-failed04.png){:height="100%" width="100%"}

<blockquote>
<p>
总结来说，用angular-cli 6.1.5默认创建的angular项目，angular版本是6.1.10，rsjx版本是6.0.0，typescript版本是2.7.2。在rxjs@6中支持conditional type：ObservedValueOf，但是typescript@2.7.2中并不支持conditional type。所以才会出现上面的编译失败问题。
</p>
</blockquote>

### 解决方案

**第一种方案：把Typescript版本从2.7.2升级到2.8。**


**第二种方案：在package.json中把"rxjs": "^6.0.0"改成"rxjs": "6.0.0"，然后重新npm install**




