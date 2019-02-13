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


**1. 父组件至子组件: 通过@Input共享数据**


**2. 子组件至父组件: 通过@Output EventEmitter共享数据**


**3. 子组件至父组件: 通过@ViewChild共享数据**


**4. 不相关组件： 通过service共享数据**


在介绍这几种方式之前，先来看下父子组件和不相关组件是什么，在下图中可以看出，左边是描述父子组件关系，左右两个是描述不相关组件关系。
![angular sharing data]( https://limeii.github.io/assets/images/posts/angular/angular-sharingdata.png){:height="50%" width="50%"}


## 第一种方式，父组件至子组件: 通过@Input共享数据
这个例子是在子组件中直接列出出父组件中所有的颜色，具体代码如下：

![angular sharing data]( https://limeii.github.io/assets/images/posts/angular/angular-sharing-data1.png){:height="100%" width="100%"}
![angular sharing data]( https://limeii.github.io/assets/images/posts/angular/angular-sharing-data2.png){:height="100%" width="100%"}

## 第二种方式，子组件至父组件: 通过@Output EventEmitter共享数据
这个例子是子组件提供两个按钮进行投票，而父组件中需要实时显示投票结果，具体代码如下：

![angular sharing data]( https://limeii.github.io/assets/images/posts/angular/angular-sharing-data3.png){:height="100%" width="100%"}
![angular sharing data]( https://limeii.github.io/assets/images/posts/angular/angular-sharing-data4.png){:height="100%" width="100%"}

## 第三种方式，子组件至父组件: 通过@ViewChild共享数据

在第二种方式中，通过@Output EventEmitter共享数据，父组件只能访问子组件的某几个属性值，但是没办法去调用子组件中的方法。但是通过@ViewChild可是实现父组件同时可以访问子组件中的属性和方法。


这个例子是一个简单的计时器，所有的计时逻辑都在子组件中，父组件负责显示时间。这个方式需要注意以下两点：


**子组件中方法或者变量为public的时候才能被父组件方法**
**在父组件中，只有等它本是AfterViewInit之后timerChildComponent才存在**

![angular sharing data]( https://limeii.github.io/assets/images/posts/angular/angular-sharing-data5.png){:height="100%" width="100%"}

![angular sharing data]( https://limeii.github.io/assets/images/posts/angular/angular-sharing-data6.png){:height="100%" width="100%"}