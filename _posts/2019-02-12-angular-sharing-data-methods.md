---
title: Angular：如何在多个组件之间通信
tags: Angular
layout: post
---


在Angular组件之间共享数据，有以下四种方式：

##### 1. 父组件至子组件: 通过 @Input 共享数据

##### 2. 子组件至父组件: 通过 @Output EventEmitter 共享数据

##### 3. 子组件至父组件: 通过 @ViewChild 共享数据

##### 4. 不相关组件： 通过 service 共享数据


在介绍这几种方式之前，先来看下父子组件和不相关组件是什么，在下图中可以看出，左边是描述父子组件关系，左右两个是描述不相关组件关系。
![angular sharing data]( https://limeii.github.io/assets/images/posts/angular/angular-sharingdata.png){:height="50%" width="50%"}


**第一种方式，父组件至子组件: 通过 @Input 共享数据**


这个例子是在子组件中直接列出出父组件中所有的颜色，具体代码如下：

![angular sharing data]( https://limeii.github.io/assets/images/posts/angular/angular-sharing-data1.png){:height="100%" width="100%"}

![angular sharing data]( https://limeii.github.io/assets/images/posts/angular/angular-sharing-data2.png){:height="100%" width="100%"}

效果如下：
![angular sharing data]( https://limeii.github.io/assets/images/posts/angular/angular-sharing-data7.png){:height="50%" width="50%"}

**第二种方式，子组件至父组件: 通过 @Output EventEmitter 共享数据**


这个例子是子组件提供两个按钮进行投票，而父组件中需要实时显示投票结果，具体代码如下：

![angular sharing data]( https://limeii.github.io/assets/images/posts/angular/angular-sharing-data3.png){:height="100%" width="100%"}

![angular sharing data]( https://limeii.github.io/assets/images/posts/angular/angular-sharing-data4.png){:height="100%" width="100%"}

效果如下：
![angular sharing data]( https://limeii.github.io/assets/images/posts/angular/angular-sharing-data8.png){:height="50%" width="50%"}

**第三种方式，子组件至父组件: 通过 @ViewChild 共享数据**


在第二种方式中，通过 @Output EventEmitter 共享数据，父组件只能访问子组件的某几个属性值，但是没办法去调用子组件中的方法。但是通过 @ViewChild 可是实现父组件同时可以访问子组件中的属性和方法。


这个例子是一个简单的计时器，所有的计时逻辑都在子组件中，父组件负责显示时间。这个方式需要注意以下两点：


- **1.子组件中方法或者变量为 public 的时候才能被父组件访问**


- **2.在父组件中，只有等它 AfterViewInit 之后 timerChildComponent 才存在**

![angular sharing data]( https://limeii.github.io/assets/images/posts/angular/angular-sharing-data5.png){:height="100%" width="100%"}

![angular sharing data]( https://limeii.github.io/assets/images/posts/angular/angular-sharing-data6.png){:height="100%" width="100%"}

效果如下：
![angular gif]( https://limeii.github.io/assets/images/posts/angular/angular-sharing-data9.gif){:height="50%" width="50%"}


**第四种方式，不相关组件： 通过 service 共享数据**


如果两个组件直接没有直接联系，那么就可以通过 service 来通信，在下面的例子中，两个不相关的组件：SiblingOneComponent 和 SiblingTwoComponent，在 SiblingOneComponent 写消息，实时在 SiblingTwoComponent 显示出来。代码如下：

![angular sharing data]( https://limeii.github.io/assets/images/posts/angular/angular-sharing-data10.png){:height="100%" width="100%"}

![angular sharing data]( https://limeii.github.io/assets/images/posts/angular/angular-sharing-data11.png){:height="100%" width="100%"}

![angular sharing data]( https://limeii.github.io/assets/images/posts/angular/angular-sharing-data12.png){:height="100%" width="100%"}

![angular sharing data]( https://limeii.github.io/assets/images/posts/angular/angular-sharing-data13.png){:height="100%" width="100%"}

效果如下：

![angular gif]( https://limeii.github.io/assets/images/posts/angular/angular-sharing-data14.gif){:height="80%" width="80%"}


示例代码可以在这查看：[angular-seed-project](https://github.com/LiMeii/angular-seed-project)