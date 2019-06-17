---
title: Angular Router guards：如何控制页面的访问权限 (CanActivate)
tags: Angular
layout: post
---


路由是整个application中非常重要的一个模块，可以用来加载某个特定的component，同时也可以传参，也可以用来控制某一模块访问权限等等。


这篇文章主要是用来介绍angular中，如何用CanActivate控制页面的访问权限。


源码可以在 [angular-seed-project](https://github.com/LiMeii/angular-seed-project) 查看。


在示例代码中，有五个模块：AppModule，DashboardModule，ReportsModule，SettingsModule，ProfileModule。


其中AppModule是root module，其他四个是业务模块。假设有两种角色登入系统，分别为admin和user。admin可以看settings和reports页面，user可以看dashboard和profile页面。


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


**第一步，在现有的模块里面加AuthenticationModule，用来实现login功能**


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

**第二步，创建一个auth-guard.module，用来实现login guard**


没有登入成功的用户除了login页面，其他模块都没办法访问。


结构如下：

![angular](https://limeii.github.io/assets/images/posts/angular/angular-routing-permission-auth-guard.png){:height="100%" width="100%"}

其中CanActivate就是用来控制路由是否有权限访问某个模块的。


**第三步，把AuthGuardModule import到AppModule**


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


**第四步，在login成功以后，在local storage保存isLogin为true**


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

**第五步，在LoginGuard添加login路由权限代码**


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

**第六步，在app-routing.module.ts的路由上加上canActivate**

```ts
const routes: Routes = [
    {
        path: '',
        loadChildren: './modules/authentication/authentication.module#AuthenticationModule',
        data: { preload: true }
    },
    {
        path: 'dashboard',
        loadChildren: './modules/dashboard/dashboard.module#DashboardModule',
        data: { preload: true },
        canActivate: [LoginGuard]
    },
    {
        path: 'settings',
        loadChildren: './modules/settings/settings.module#SettingsModule',
        canActivate: [LoginGuard]
    },
    {
        path: 'reports',
        loadChildren: './modules/reports/reports.module#ReportsModule',
        canActivate: [LoginGuard]
    },
    {
        path: 'profile',
        loadChildren: './modules/profile/profile.Module#ProfileModule',
        canActivate: [LoginGuard]
    }

];

```

好了，到现在为止，login这块的权限都已经加好了，npm run start把application跑起来以后，login之前，任何业务模块页面都没办法访问：

![angular](https://limeii.github.io/assets/images/posts/angular/angular-routing-permission-login.gif){:height="100%" width="100%"}

**第七步，限制addmin只可以访问settings和reports页面，user只可以访问dashboard和profile页面**


访问没有权限的页面会返回login页面


现在app-constants.ts文件中，加上permissionset和admin/user 访问权限

```ts
//app-constants.ts
export const permissionSets = {
    APP_ManageSettings: ' APP_ManageSettings',
    APP_ManageReports: 'APP_ManageReports',
    APP_ManageDashboard: 'APP_ManageDashboard',
    APP_ManageProfile: 'APP_ManageProfile'
}

export const userPermission = {
    adminFeatures: {
        privilege: [permissionSets.APP_ManageSettings, permissionSets.APP_ManageReports],
        menus: [
            {
                title: 'settings',
                routerLink: 'settings',
                privilege: [permissionSets.APP_ManageSettings]
            },
            {
                title: 'reports',
                routerLink: 'reports',
                privilege: [permissionSets.APP_ManageReports]
            }
        ]
    },
    userFeatures: {
        privilege: [permissionSets.APP_ManageDashboard, permissionSets.APP_ManageProfile],
        menus: [
            {
                title: 'dashboard',
                routerLink: 'dashboard',
                privilege: [permissionSets.APP_ManageDashboard]
            },
            {
                title: 'profile',
                routerLink: 'profile',
                privilege: [permissionSets.APP_ManageProfile]
            }
        ]
    }
}
```

新加admin-guard.ts 用来控制admin访问权限，并且import到auth-guard.module.ts

```ts
// admin-guard.ts
import { Injectable } from "@angular/core";
import { Router, CanActivate, ActivatedRouteSnapshot } from '@angular/router';
import { LocalStorageService } from "angular-2-local-storage/dist";

import { role, userPermission, appStorage } from '../app-constants';


@Injectable()

export class AdminGuard implements CanActivate {
    constructor(private router: Router,
        private localStorageService: LocalStorageService) { }

    canActivate(activatedRouteSnapshot: ActivatedRouteSnapshot): boolean {
        let isLogin = this.localStorageService.get(appStorage.isLogin);
        let roleType = this.localStorageService.get(appStorage.loginType);
        let canRoute = false;
        if (isLogin && roleType === role.admin) {
            let privilegesToRoute = activatedRouteSnapshot.data.privileges;
            if (privilegesToRoute) {
                privilegesToRoute.forEach(element => {
                    if (userPermission.adminFeatures.privilege.indexOf(element) !== -1) {
                        canRoute = true;
                    }
                });
            }
        } else {
            this.router.navigate(['login']);
            canRoute = false;
        }
        return canRoute;
    }
}
```

在setting-routing.module.ts中引入AdminGuard，并且在路由上加上可以访问的权限数据

```ts
//setting-routing.module.ts
import { permissionSets } from '../../core/app-constants';
import { AdminGuard } from '../../core/guards/admin-guard';

const settingsRoutes: Routes = [
    {
        path: '',
        component: SettingsComponent,
        data: { privileges: [permissionSets.APP_ManageSettings] },
        canActivate: [AdminGuard]
    }
];
```

在reports-routing.module.ts中引入AdminGuard，并且在路由上加上可以访问的权限数据

```ts
//setting-routing.module.ts
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';

import { ReportsComponent } from './reports.component';
import { AdminGuard } from '../../core/guards/admin-guard';
import { permissionSets } from '../../core/app-constants';

const ReportsRoutes: Routes = [
    {
        path: '',
        component: ReportsComponent,
        data: { privileges: [permissionSets.APP_ManageReports] },
        canActivate: [AdminGuard]
    }
];

```

user用户的权限代码与admin权限代码类似，具体代码可以在 [angular-seed-project](https://github.com/LiMeii/angular-seed-project) 查看。

好了，现在整个路由权限都加好了，效果如下：

![angular](https://limeii.github.io/assets/images/posts/angular/angular-routing-permission-for-role.gif){:height="100%" width="100%"}

**总结**


这篇文章中主要是介绍了CanActivate对路由权限的控制，这个用法有一个缺陷：用户虽然没有权限访问某个module或是component，但是通过按钮或者url hit到对应的路由的时候，module/component还是会被加载。

接下来一篇文章介绍 [Angular Router guards：如何控制页面的访问权限 (Canload)](/2018/11/angular-routing-guard-canload) 会介绍canload对路由权限控制的用法，canload和canactive的一个主要区别就是canload在没有权限访问的时候不会去加载模块。
