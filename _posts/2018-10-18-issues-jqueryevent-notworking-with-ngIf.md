---
title: Angular ngIf里所有的Jquery click事件都没有触发
tags: 问题
layout: post
---


### 问题描述

因为公司legacy系统和policy原因，需要在angular项目里引入外部Jquery js文件，angular html页面元素的事件触发是写在这个Jquery文件里。


在项目开发过程中，遇到过这样一个问题：如果页面里用 *ngIf根据条件来显示页面元素，恰好这个元素里需要触发Jquery click事件，这个事件在这种情况下是不会被触发。


代码如下：

```html
    <!-- angular html-->
    <div id="target" class="page-navigation__item js-page-navigation-item" *ngIf="isLogin">
        <button id="lngddl" class="page-navigation__item-link js-page-navigation-item-link">
                language
        </button>
        <div class="page-navigation__secondary js-page-navigation-secondary">
            <div class="page-navigation__secondary-inner js-page-navigation-secondary-inner">
                    <a class="page-navigation__secondary-link tnt tnt-nav-sec-link5">English</a>
                    <a class="page-navigation__secondary-link tnt tnt-nav-sec-link5">español</a>
                </div>
            </div>
    </div>
```

```js
    //jquery click event
  $('.js-page-navigation-item-link').click(function(){
    targetNavItemLink = $(this);
    // 此处省略具体逻辑代码
  });
```

### 问题分析

发现点击button没有期望的效果的时候，我在这个button上面直接加上(click)="test()", 然后点击button发现test方法会被触发，说明html正常没有问题。


尝试把*ngIf="isLogin"去掉以后，这次点击button，这次jquery事件被触发了。


好了，问题就在ngIf，那我再尝试把ngIf换成有类似效果的[ngClass]="{'item-hidde': !isLogin}",其中item-hidde是css，代码如下：
```css
.item-hidde {
    display: none; 
}
```
重新编译代码以后，点击button，jquery click事件能被正常触发。


到这一步的时候，隐约大概知道问题的root cause是什么了，我们来看下ngIf和s正常hidden一段的元素的区别：ngIf是动态的，也就是条件为true的时候，整个div都在显示在页面，如果条件为false的时候，整个div节点根本不会存在。而disply: none不管元素显示还是不显示，整个节点都是存在的。


好了，到这里就知道问题原因了，在Jquery里, 直接绑定的事件对动态生成的元素不生效。


那什么是直接绑定事件？


**Jquery 直接绑定事件和事件delegated**


**直接绑定**
```js
  $('.js-page-navigation-item-link').click(function(){});
```
在代码执行的时候把事件直接绑定到有 js-page-navigation-item-link css的元素上，假如之后有动态加上新的元素也有js-page-navigation-item-link，这个元素的click事件不会被监听到，同理如果移除已有的包含js-page-navigation-item-link css的元素，这个被移除元素的事件还是会被监听。


**事件delegated**

```js
  $(div#target).on('click','.js-page-navigation-item-link',function(){});
```
这个表示div元素并且id为target里面有js-page-navigation-item-link css的所有子元素，一旦有click事件都会被监听到。根据Jquery事件触发的冒泡原理，子元素的click事件都被父元素delegated。

### 解决方案

**第一种方案：** 在html里，不用动态生成元素，也就是不用*ngIf，而是用hidden或者[ngClass]类似根据条件显示/隐藏元素。


**第二种方案：** 在Jquery里，事件不直接绑定而是用delegated。



```
当然在angular中还是尽量避免用外部引用的JQUERY library
```