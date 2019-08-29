---
title: Angular：性能优化清单
tags: Angular
layout: post
---

在深入理解和学习Angular过程中，发现在搭建Angular项目、实际项目应用、打包、合理使用变化检测策略、缓存，各个方面都可以优化项目性能。因为都写在不同的文章里，比较分散，为了方便查阅，在这篇文章集中把性能优化相关的文章都列出来了。

## 推荐用Angular自带的编译方式打包

Angular内置了webpack打包方式，很多人在搭建Angular(< 6.0)项目的时候，发现Angular内置的webpack并不能满足实际项目打包的一些需求，所以通过```ng eject```把内置的webpack.config文件暴露出来，然后根据自己项目需求重写整个webpack.config配置，可以参考这篇文章【[Angular：如何用Angular(<6.0)和Webpack搭建项目](https://limeii.github.io/2018/09/angular-webpack/)】


但是Angular(>= 6.0)，去掉了```ng eject```命令，鼓励大家用Angular内置的打包方式开发Angular项目，因为```AoT``` ```Tree-Shaking``` ```ngc``` ```tsc``` ```minification``` ```Production mode```等等这些在Angular内置的打包方式中都是默认配置，不需要我们自己再去找第三方的loader或者plugin来实现类似的功能；而且从性能上说内置的打包方式要比我们自己搭的要好。


如何结合内置的打包方式和我们自己的webpack配置文件编译打包整个项目，可以参考文章：
- 【[Angular：如何在Angular(8.0)中配置Webpack](https://limeii.github.io/2019/08/angular-customize-webpack/)】

## 使用AoT编译

在打包过程中使用AoT编译，指的是把所有HTML component NgModule CSS都编译成浏览器可以识别的es5代码。从而用户访问Angular应用的时候，只需要下载相应的budnle文件后直接渲染。不需要先下载```@angular/compiler```编译器编译模板文件，然后再渲染，大大提高了页面渲染性能。


在模板文件中Angular有自己的directive component pipe，如果不先编译成es6代码，就没办法做Tree Shaking。AoT(ngc)编译，可以把模板文件先打包成es6(或者TypeScript)文件，这样就可以做Tree Shaking，把一些没用的代码去掉，从而减少bundle文件的体积，也可以提高性能。


关于深入理解Angular编译，可以参考这篇文章：
- 【[Angular：深入理解Angular编译机制](https://limeii.github.io/2019/08/angular-compiler/)】

## Tree Shaking

如果项目中有一些代码(方法、文件)，在项目里完全没有被用到，这种代码称为Dead Code。大量Dead Code如果编译打包进bundle文件，会导致bundle文件过大，页面渲染下载bundle文件的时候会浪费带宽，也会影响性能。


Tree Shaking就是用来解决这种问题，它是指在编译打包过程中把Dead Code去掉，不把这些没用到的代码打包到最后的bundle文件里，从而可以减小bundle文件的体积，提高应用性能。可以把整个应用想象成一棵树，function/component/service/lib好比是树叶，而那些定义但又没有被调用的function/component/service/lib好比是枯树叶，Tree Shaking就是把那些枯树叶从树上摇下去。


关于更多Tree Shaking的理解和应用可以参考这篇文章：
- [Angular性能优化：Tree Shaking](https://limeii.github.io/2019/08/angular-tree-shaking/)

## lazy loading和preloading bundle文件
如果把一个大项目所有文件都打包进一个bundle文件，用户打开浏览器访问这个网站的时候，首先要从服务器下载这个超大的bundle文件，加上还需要时间做一些解析，会导致网站响应过慢。在Angular项目中可以通过配置lazy loading和preloading来提高性能。关于lazy loading和preloading在Angular项目中的理解和应用可以参考这篇文章：
- 【[Angular：lazy loading和preloading](https://limeii.github.io/2018/09/angular-lazy-loading/)】

## webpack Code Splitting
在项目中经常会有一些公用代码，被多个module或者component引用，如果在打包的时候直接把公用代码重复打包进不同的bundle文件，会造成代码冗余，也会影响应用性能。我们可以通过webpack Code Splitting做代码切割，把公用的代码单独提取出来放在chunk文件里，用户访问页面的时候只需要下载一次这个chunk文件就可以了。具体可以参考这篇文章：
- [webpack(3)：代码切割](https://limeii.github.io/2018/10/webpack-code-splitting/)

## 使用ChangeDetectionStrategy.OnPush策略
Angular默认的变化检测机制是：异步事件callback结束后，NgZone会触发整个组件树至上而下做变化检测，也就是说页面一个小小的Click事件就会触发所有组件的变化检测。虽然Angular变化检测本身性能已经很好了，在毫秒内可以做成百上千次变化检测。但是随着项目越来越大，其实很多不必要的变化检测还是会在一定程度上影响性能。在Angular中可以通过OnPush来跳过一些不必要的变化检测，从而优化整个应用的性能。更多关于OnPush策略的理解和应用可以参考这篇文章：
- 【 [Angular Change Detection：变化检测策略](https://limeii.github.io/2019/06/angular-changeDetectionStrategy-OnPush/)】


## 缓存
### 用RxJS实现缓存效果
Angular中通过HttpClient执行Http Request返回的Observables是Cold Observable，这导致每次调用API，都会生成一个新的Observable实例，有订阅之后才开始发送值，这也符合现在前端开发要求。但是实际开发过程中，有时候后端会有提供一些公用的常量API，不同页面都需要用这些常量，按现在的调用API的方式，会导致常量API在不同的页面重复多次被调用，这种方式显然性能不好。

可以通过ReplaySubject实现缓存效果，第一次调用常量API之后把这些常量缓存起来，之后调用同样的API就可以直接在ReplaySubject拿到值，不用每次都调用后端API。具体实现可以参考文章：
- 【[RxJS：如何通过RxJS实现缓存](https://limeii.github.io/2019/08/rxjs-caching/)】

### 通过Service Worker缓存

**未完待续**
