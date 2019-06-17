---
title: Angular Router guards：如何控制页面的访问权限 (Canload)
tags: Angular
layout: post
---


在[Angular Router guards：如何控制页面的访问权限 (CanActivate)](/2018/10/angular-routing-guards)介绍了CanActivate对路由权限的控制。

在这篇文章中会介绍CanLoad对路由权限的控制和用法。


在开始介绍CanLoad的用法之前，先来讲下CanLoad和CanActivate的区别：

- 1：CanActivate可以用在component级别，也可以用在module级别；CanLoad只能和lazy loading一起用，也就是一定要和loadChildren一起用。

- 2：如有多层级路由中有CanActivate，是从上往下检查路由权限，也就是从顶层到最深的child route，一旦有返回值为false的，整个检查终止。

- 3：CanLoad的路由检查在整个module load之前就进行了，并且优先级要比preloading要高，也就是如果用到了PreloadAllModules，有用CanLoad的module在项目启动的时候不会preloading。


好了，接下来就开始介绍Canload的用法。

**第一步，在admin-guard.ts和user-guards.ts实现CanLoad接口**

```ts
// admin-guard.ts
export class AdminGuard implements CanActivate, CanLoad{
    //省略重复代码
    canLoad(): boolean {
        let isLogin = this.localStorageService.get(appStorage.isLogin);
        let roleType = this.localStorageService.get(appStorage.loginType);
        let canRoute = false;
        if (isLogin && roleType === role.admin) {
            canRoute = true;
        }
        return canRoute;
    }
}
```
```ts
//user-guards.ts
export class UserGuard implements CanActivate, CanLoad {
    constructor(private router: Router, private localStorageService: LocalStorageService) { }
    //省略重复代码
    canLoad(): boolean {
        let isLogin = this.localStorageService.get(appStorage.isLogin);
        let roleType = this.localStorageService.get(appStorage.loginType);
        let canRoute = false;
        if (isLogin && roleType === role.user) {
            canRoute = true;
        }
        return canRoute;
    }
}
```

**第二步，在app-routing.module.ts中给对应的loadChildren加上canLoad**


dashboard settings reports profile对应的module都是需要lazyloading，也就是有访问到这些模块并且是有权限的才会被加载。

```ts
//app-routing.module.ts
import { UserGuard } from './core/guards/user-guards';
import { AdminGuard } from './core/guards/admin-guard';

const routes: Routes = [
    {
        path: '',
        loadChildren: './modules/authentication/authentication.module#AuthenticationModule',
        data: { preload: true }
    },
    {
        path: 'dashboard',
        loadChildren: './modules/dashboard/dashboard.module#DashboardModule',
        canLoad: [UserGuard]
    },
    {
        path: 'settings',
        loadChildren: './modules/settings/settings.module#SettingsModule',
        canLoad: [AdminGuard]
    },
    {
        path: 'reports',
        loadChildren: './modules/reports/reports.module#ReportsModule',
        canLoad: [AdminGuard]
    },
    {
        path: 'profile',
        loadChildren: './modules/profile/profile.Module#ProfileModule',
        canLoad: [UserGuard]
    }

];
```

到这一步，代码都写好了，我们npm run start看下登入之前，canload对路由权限的控制：

![angular](https://limeii.github.io/assets/images/posts/angular/angular-routing-canload.gif){:height="100%" width="100%"}

从上面的动图可以看出，在登入之前，除了加载login页面对应的4.chunk.js文件以外，其他模块都不能访问。


而且虽然在url输入正确的url，对应的模块（chunk.js）也不会被加载


我们来看下，admin登入以后，模块的加载情况：
![angular](https://limeii.github.io/assets/images/posts/angular/angular-routing-canload-canactivate.gif){:height="100%" width="100%"}

从上面的动图可以看出，登入之前只有4.chunk.js，admin登入以后有权限看settings/reports，settings对应加载0.chunk.js，reports对应加载1.chunk.js。

url里输入profile/dashboard的时候，无权限访问对应页面，也不会加载对应的chunk文件。


user登入以后也是类似的效果。