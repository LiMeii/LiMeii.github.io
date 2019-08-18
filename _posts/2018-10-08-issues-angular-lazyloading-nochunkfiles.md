---
title: Angular lazy loading没有生成业务模块对应的chunk文件
tags: 问题
layout: post
---


### 问题描述

开发环境：

- 基于angular5.0 和 webpack3.10.0

在angular2以上的版本，提供了lazy loading / preloading 的功能，防止大型网站在加载首页时间过长，降低用户体验。


在【[angular lazy loading 和 preloading](/2018/09/angular-lazy-loading)】这篇文章解释了lazy loading 和 preloading原理和实现方式。



但是在项目打包后，没有生成业务模块的chunk文件，没有chunk文件也就没办法实现lazy loading。源码：[angular-seed-project](https://github.com/LiMeii/angular-seed-project) 


用[webpack bundle analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer)插件发现，所有的业务模块代码：DashboardModule/SettingModule/ReportsModule还是一起打包在app.bundle.js文件中，没有分别生成三个单独的chunk文件：0.chunk.js 1.chunk.js 2.chunk.js


### 问题分析

**业务代码打包进app.bundle.js, 而不是生成独立的chunk文件，说明angular router设置不对？**

回头看了下路由文件，没发现什么问题，完全是按照官方best practice写的路由。

**为什么会打包进app.bundle.js, 而不其他的比如vendor.bundle.js？**

回头看了下app.module.ts文件，找到问题了，我在这个文件里import了这三个业务模块。而且app-routing.module.ts又设置了lazy loading。


源码如下：
```ts
    //app.module.ts
    import { DashboardModule } from './modules/dashboard/dashboard.module';
    import { ReportsModule } from './modules/reports/reports.module';
    import { SettingsModule } from './modules/settings/settings.module';
```

```ts
  //app-routing.module.ts

    import { NgModule } from '@angular/core';
    //import { PreloadAllModules, RouterModule, Routes } from '@angular/router';
    import { RouterModule, Routes } from '@angular/router';
    import { AppCustomPreloader } from './app.custom.preloader';

    const routes: Routes = [
        { path: '', pathMatch: 'full', redirectTo: 'dashboard' },
        {
            path: 'dashboard',
            loadChildren: './modules/dashboard/dashboard.module#DashboardModule',
            data: { preload: true }
        },
        { path: 'settings', loadChildren: './modules/settings/settings.module#SettingsModule' },
        { path: 'reports', loadChildren: './modules/reports/reports.module#SettingsModule' },

    ];

    @NgModule({
        // imports: [RouterModule.forRoot(routes, { preloadingStrategy: PreloadAllModules, useHash: true })],
        imports: [RouterModule.forRoot(routes, { preloadingStrategy: AppCustomPreloader, useHash: true })],
        exports: [RouterModule],
        providers: [AppCustomPreloader]
    })
    export class AppRoutingModule { }
```
angular在编译打包过程中，在root module中引用了业务模块，就会直接把业务模块一起打包进app.bundle.js, 忽略lazy loading的设置，从而不会生成chunk文件。

### 解决方案
把业务模块引用从root module文件中删掉就可以了。