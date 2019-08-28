---
title: Angular：性能优化清单
tags: Angular
layout: post
---

在我深入理解和学习Angular过程中，发现在搭建Angular项目、实际项目应用、打包、合理使用变化检测策略，各个方面都可以优化项目性能。由于都写在了不同的文章里，也比较分散，为了方便查阅，在这篇文章集中把跟性能相关的文章都列出来了：

## 推荐用Angular自带的编译方式打包

Angular内置了webpack打包方式，很多人在搭建Angular(< 6.0)项目的时候，发现Angular内置的webpack并不能满足实际项目打包的一些需求，所以通过```ng eject```把内置的webpack.config文件暴露出来，然后根据自己项目需求重写整个webpack.config配置，可以参考这篇文章【[Angular：如何用Angular(<6.0)和Webpack搭建项目](https://limeii.github.io/2018/09/angular-webpack/)】


但是Angular(>= 6.0)，去掉了```ng eject```鼓励大家用Angular内置的打包方式开发Angular项目，因为```AoT``` ```Tree-Shaking``` ```ngc``` ```tsc``` ```minification``` ```Production mode```等等这些在Angular内置的打包方式中都涵盖了，不需要我们自己再去找第三方的一些loader或者plugin来实现类似的功能；而且从性能上说内置的打包方式要比我们自己搭的要好。


如何结合内置的打包方式和我们自己的webpack配置文件编译打包整个项目，可以参考文章【[Angular：如何在Angular(8.0)中配置Webpack](https://limeii.github.io/2019/08/angular-customize-webpack/)】

## 使用AoT编译

在打包过程中，就把所有的HTML component NgModule CSS都编译成浏览器可以识别的es5代码，在用户访问Angular应用的时候，只需要下载相应的budnle文件，不需要下载```@angular/compiler```编译器来编译模板文件，大大提高了页面渲染性能。


因为在模板文件中有Angular自己的directive component pipe，没办法做Tree Shaking。用AoT(ngc)编译，可以把模板文件先打包成es6(或者TypeScript)文件，这样也可以做Tree Shaking，把一些没用的代码去掉，从而减少bundle文件的体积，也会提高性能。


关于深入理解Angular编译，可以参考这篇文章【[Angular：深入理解Angular编译机制](https://limeii.github.io/2019/08/angular-compiler/)】

## Tree Shaking
 [Angular性能优化：Tree Shaking](https://limeii.github.io/2019/08/angular-tree-shaking/)

## lazy loading和preloading bundle文件
[Angular：lazy loading和preloading](https://limeii.github.io/2018/09/angular-lazy-loading/)

## 使用ChangeDetectionStrategy.OnPush策略
 [Angular Change Detection：变化检测策略](https://limeii.github.io/2019/06/angular-changeDetectionStrategy-OnPush/)
