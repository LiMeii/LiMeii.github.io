---
title: Angular lazy loading和preloading
tags: Angular
layout: post
---


### 什么是lazy loading 和 preloading


在打开浏览器访问某一个网站的时候，要从服务器下载 bundle/chunk 等文件。如果文件过大或过多，会导致网站响应过慢。

**lazy loading**，翻译到中文就是懒加载，其实就是按需加载。

最开始访问某一个网站index页面的时候，只下载跟index/核心代码相关的 bundle/chunk 等文件。之后路由到新的页面，再按需下载新页面需要的文件，大大的提高了首页访问的性能，但是访问新页面的时候还是需要时间下载新文件。

这样就防止在访问大型网站首页的时候，下载文件过大或过多，导致网站首页响应时间过长，降低用户体验。

可以参考如下动图：



![angular lazy loading](https://limeii.github.io/assets/images/posts/angular/angular-lazy-loading.gif){:height="100%" width="100%"}

**preloading**，翻译到中文是预加载，这种方式的意思，一旦网站被访问，会去下载所有的文件。

不同的是，用户访问网站的时候，最先加载网站所需要的初始化的文件，然后再后台下载其他JS文件。

preloading这种方式在用户切换模块也就是访问新页面之前就把所需的文件下载好了，相比lazy loading方式大大提高了页面切换时的性能。

但是如果网站过大，一次性全部预加载所有的文件，也会导致首页加载之后后台下载过多不必要的文件。

页面加载最优的方案是，初始化网页加载的时间越少越好，然后再按需加载新页面。也就是初始化（核心）bundle文件要越小越好，其他的业务模块应该要用户点了导航之后再按需加载。

### 如何保证核心bundle文件越小越好

**通过代码切割**


什么是代码切割？在代码打包过程中，可以把最终bundle文件中，独立的或者公用的代码抽取出来，放在chunk文件中，这就是代码切割。有了抽取出来的chunk文件，就可以实现lazy loading了。


怎么实现代码切割？什么时候做代码切割？


可以参考文章 [webpack 代码切割](/2018/10/webpack-code-splitting) 

### 如何在angular中实现lazy loading
在angular中，lazy loading是跟路由一起实现的。也就是在代码打包过程中，把每个路由对应的模块都打包成独立的chunk文件，最终就可以实现用户点击导航到新页面的时候按需加载对应的chunk文件。


**第一步，在webpack中配置代码切割**


为了实现按照路由切割代码，我们需要用到 [angular-router-loader](https://www.npmjs.com/package/angular-router-loader) 或者是 [ng-router-loader](https://www.npmjs.com/package/ng-router-loader)。 以 angular-router-loader 为例，需要在webpack config文件中加如下代码：

```js
    {
        test: /\.ts$/,
        use: [
          'awesome-typescript-loader',
          'angular-router-loader',
          'angular2-template-loader',
        ]
      }
```
然后再webpack output中加如下代码：
```js
  output: {
    path: path.join(__dirname, './build-dev'),
    filename: 'js/[name].bundle.js',
    chunkFilename: 'js/[id].chunk.js'
  },
```
这种配置，chunk最后的名字会是 0.chunk.js 1.chunk.js 2.chunk.js ......

**第二步，配置路由**
```
比如项目启动模块是 AppModule
业务模块分别是：DashboardModule SettingsModule ReportsModule
```
如下是 ReportsModule的代码：
```ts
    // reports.module.ts
    import { NgModule } from '@angular/core';
    import { Routes, RouterModule } from '@angular/router';

    // containers
    import { ReportsComponent } from './reports.component';

    // routes
    export const ROUTES: Routes = [{ path: '', component: ReportsComponent }];

    @NgModule({
    imports: [RouterModule.forChild(ROUTES)],
    declarations: [ReportsComponent],
    })
    export class ReportsModule {}
```
在AppModule中，加入如下代码：
```ts
    // app.module.ts
    export const ROUTES: Routes = [
    { path: 'reports', loadChildren: '../reports/reports.module#ReportsModule' },
    ];
```

这就实现了当url为 /reports的时候，就会下载report这个模块的业务代码，需要注意的是ReportsModule中，path是 ' '。


其他模块也类似这样实现，最后AppModule的路由代码如下：
```ts
    export const ROUTES: Routes = [
    { path: '', pathMatch: 'full', redirectTo: 'dashboard' },
    {
        path: 'dashboard',
        loadChildren: '../dashboard/dashboard.module#DashboardModule',
    },
    {
        path: 'settings',
        loadChildren: '../settings/settings.module#SettingsModule',
    },
    { path: 'reports', loadChildren: '../reports/reports.module#ReportsModule' },
    ];
```

这样在build结束以后，除了bundle文件以外，会有三个chunk文件分别是 0.chunk.js，1.chunk.js，2.chunk.js。这样就实现了按需加载。
- 需要注意的是，在app.module.ts文件中不要再import业务代码module，否则不会生成chunk文件**


完整代码可以在 [angular-seed-project](https://github.com/LiMeii/angular-seed-project) 中查看，需要注意的是在源码中把路由代码单独提取到app-routing.module.ts文件中了。

### 如何在angular中实现preloading
在angular中，[PreloadAllModules](https://angular.io/api/router/PreloadAllModules)这个功能可以用来实现预加载所有的文件，一旦访问网页就会下载所有的文件。
实现代码如下：
```ts
    import { RouterModule, Routes, PreloadAllModules } from @angular/router;

    export const ROUTES: Routes = [
    { path: '', pathMatch: 'full', redirectTo: 'dashboard' },
    { path: 'dashboard', loadChildren: '../dashboard/dashboard.module#DashboardModule' },
    { path: 'settings', loadChildren: '../settings/settings.module#SettingsModule' },
    { path: 'reports', loadChildren: '../reports/reports.module#ReportsModule' }
    ];

    @NgModule({
    // ...
    imports: [
        RouteModule.forRoot(ROUTES, { preloadingStrategy: PreloadAllModules })
    ],
    // ...
    })
    export class AppModule {}

```
在文章最开始就提到，一般我们不需要所有文件都预加载，而是最核心的文件预加载以后，在按需加载业务模块代码。

### 如何在angular中结合使用preloading和lazy loading
angular中默认只有PreloadAllModules这个选择，如果需要预加载一部分代码，就需要自己写代码来实现。


我们写一个自己的 AppCustomPreloader 如下：
```ts
    import { PreloadingStrategy, Route } from '@angular/router';

    import { Observable, of } from 'rxjs';

    export class AppCustomPreloader implements PreloadingStrategy {
    preload(route: Route, load: Function): Observable<any> {
        return route.data && route.data.preload ? load() : of(null);
    }
    }
```
然后把AppModule改为：
```ts
    import { AppCustomPreloader } from './app-custom-preloader';

    export const ROUTES: Routes = [
    { path: '', pathMatch: 'full', redirectTo: 'dashboard' },
    { 
        path: 'dashboard', 
        loadChildren: '../dashboard/dashboard.module#DashboardModule',
        data:{preload:true} 
    },
    { path: 'settings', loadChildren: '../settings/settings.module#SettingsModule' },
    { path: 'reports', loadChildren: '../reports/reports.module#ReportsModule' }
    ];

    @NgModule({
    // ...
    imports: [
        RouteModule.forRoot(ROUTES, { preloadingStrategy: AppCustomPreloader })
    ],
    // ...
    })
    export class AppModule {}
```
这就表示dashboard会预加载，而其他两个模块会按需加载。