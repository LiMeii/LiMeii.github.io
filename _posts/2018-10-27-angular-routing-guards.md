---
title: angular routing guards
layout: post
---

# Angular Router guards：如何控制页面的访问权限

<div class="title-meta">
    <span><img class="title-category-img" src="../../../assets/images/categories/angular.svg" alt="Angular"></span>
    <span><a class="github-link" href="/2018/09/19/angular.html">Angular</a></span>
    <span class="title-bullet">•</span>
    <span>Oct 27, 2018</span>
    <span class="title-bullet">•</span>
    <span>by <a class="github-link" href="http://github.com/limeii" title="http://github.com/limeii">Mei</a></span>
</div>

路由是整个application中非常重要的一个模块，可以用来加载某个特定的component，同时也可以传参，也可以用来控制某一模块访问权限等等。


这篇文章主要是用来介绍angular中，如何控制页面的访问权限。


源码可以在 [angular-seed-project](https://github.com/LiMeii/angular-seed-project) 查看。


在示例代码中，有五个个模块： AppModule，DashboardModule，ReportsModule，SettingsModule，ProfileModule。


其中AppModule是root module，其他四个是业务模块。假设有两种角色登入系统，分别为 admin 和 user。admin可以看到settings页面，user可以看到dashboard页面。


现在路由设置如下：
```ts
// app-routing.module.ts
const routes: Routes = [
    {
        path: '',
        pathMatch: 'full',
        redirectTo: 'dashboard'
    },
    {
        path: 'dashboard',
        loadChildren: './modules/dashboard/dashboard.module#DashboardModule',
        data: { preload: true }
    },
    {
        path: 'settings',
        loadChildren: './modules/settings/settings.module#SettingsModule'
    },
    {
        path: 'reports',
        loadChildren: './modules/reports/reports.module#ReportsModule'
    },
    {
        path: 'profile',
        loadChildren: './modules/profile/profile.Module#ProfileModule'
    }
    @NgModule({
    // imports: [RouterModule.forRoot(routes, { preloadingStrategy: PreloadAllModules, useHash: true })],
    imports: [RouterModule.forRoot(routes, { preloadingStrategy: AppCustomPreloader, useHash: true })],
    exports: [RouterModule],
    providers: [AppCustomPreloader]
})
export class AppRoutingModule { }

];
```

```ts
//reports-routing.module.ts
const ReportsRoutes: Routes = [
    { path: '', component: ReportsComponent }
];


@NgModule({
    imports: [RouterModule.forChild(ReportsRoutes)],
    declarations: [ReportsComponent],
    exports: [RouterModule]
})

export class ReportsRoutingModule { }
```
其他四个业务模块路由代码类似。


现在是不论是admin还是user角色，在访问页面的时候都不受限制。但是在实际项目中，我们可能需要：

- 在用户没有登入的情况下，只能访问login页面
- 登入成功以后，用户也只可以访问授权的页面


在angular中，可以用Route guards canActivate来实现上面的case。


### 第一步，在现有的模块里面加AuthenticationModule，用来实现login功能。
结构如下：

![angular](https://limeii.github.io/assets/images/posts/angular/angular-routing-permission-authModule.png){:height="100%" width="100%"}

同时在app-routing.module.ts里加上这个模块的路由路径：

```ts
    {
        path: '',
        loadChildren: './modules/authentication/authentication.module#AuthenticationModule',
        data: { preload: true }
    },
```

### 第二步，创建一个auth-guard.module。
结构如下：

![angular](https://limeii.github.io/assets/images/posts/angular/angular-routing-permission-auth-guard.png){:height="100%" width="100%"}

其中CanActivate就是用来控制路由是否有权限访问某个模块的。

### 第三步，把AuthGuardModule import到AppModule
```ts
import { AuthGuardModule } from './core/guards/auth-guard.module';
@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    LocalStorageModule,
    AuthGuardModule,
    LocalStorageModule.withConfig({
      prefix: '',
      storageType: 'localStorage'
    })
  ],
  providers: [LocalStorageService],
  bootstrap: [AppComponent]
})
export class AppModule { }

```

### 第四步，在login成功以后，在local storage保存isLogin为true

在示例代码中用到了localstorage是第三方的library： angular-2-local-storage


login的具体代码如下：

```ts
import { Component } from "@angular/core";
import { LocalStorageService } from 'angular-2-local-storage';
import { Router } from '@angular/router';

import { appStorage, role } from '../../../core/app-constants';

@Component({
    templateUrl: './login.component.html'
})

export class LoginComponent {

    constructor(private localStorageService: LocalStorageService, private router: Router) {

    }

    onAdminLogin() {
        this.localStorageService.set(appStorage.isLogin, true);
        this.localStorageService.set(appStorage.loginType, role.admin);
        this.router.navigate(['settings'])
    }

    onUserLogin() {
        this.localStorageService.set(appStorage.isLogin, true);
        this.localStorageService.set(appStorage.loginType, role.user);
        this.router.navigate(['dashboard'])
    }
}

```

### 第五步，在LoginGuard添加login路由权限代码

这段逻辑是在login之前，所有用户只能访问login页面，其他页面都没有权限访问。

```ts
import { Injectable } from '@angular/core';
import { CanActivate, Router } from '@angular/router';
import { LocalStorageService } from "angular-2-local-storage/dist";

import { appStorage } from '../app-constants';

@Injectable()

export class LoginGuard implements CanActivate {
    constructor(private localStorageService: LocalStorageService, private router: Router) { }
    canActivate(): boolean {
        let isLogin = this.localStorageService.get(appStorage.isLogin);
        if (isLogin) {
            return true;
        }
        else {
            this.router.navigate(['login']);
            return false;
        }

    }
}
```

好了，npm run start把application跑起来以后，login之前，任何业务模块页面都没办法访问：

![angular](https://limeii.github.io/assets/images/posts/angular/angular-routing-permission-login.gif){:height="100%" width="100%"}
