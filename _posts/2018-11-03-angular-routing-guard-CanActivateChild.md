---
title: Angular Router guards：如何控制页面的访问权限 (CanActivateChild)
tags: Angular
layout: post
---


前面两篇文章介绍了路由权限CanActivate和Canload的用法，这里接着介绍路由权限CanActivateChild的用法。


源码可以在 [angular-seed-project](https://github.com/LiMeii/angular-seed-project) 查看。


在介绍用法之前，先来讲下这三个的区别：

- 1：CanActivate可以用在component级别，也可以用在module级别。

- 2：CanLoad只能和lazy loading一起用，也就是一定要和loadChildren一起用。

- 3：CanActivateChild 是用在child router上。

- 4：如有多层级路由中有CanActivate，CanActivateChild，Canload，最先检查CanActivateChild，从最下往上检查路由权限，一旦有返回值false整个校验终止；然后检查CanActivate，从上往下检查路由权限，也就是从顶层到最深的route，一旦有返回值为false的，整个检查终止；而Canload是整个module 加载之前就校验，也就是三个里面最先检查。

- 5：CanLoad的路由检查在整个module load之前就进行了，并且优先级要比preloading要高，也就是如果用到了PreloadAllModules，有用CanLoad的module在项目启动的时候不会


好了，接下俩开始介绍CanActivateChild的具体用法。


**第一步，在ReportsModule里加上两个新的component作为child router对应的component**


结构如下：

![angular](https://limeii.github.io/assets/images/posts/angular/angular-router-canactivatedchild-1.png){:height="100%" width="100%"}

并import到ReportsModule里面
```ts
import { NgModule } from '@angular/core';
import { RouterModule } from '@angular/router';

import { ReportsRoutingModule } from './reports-routing.module';
import { ReportsComponent } from './reports.component';
import { DailyReportCompoent } from './daily-reports/daily-reports-component';
import { WeeklyReportsComponent } from './weekly-reports/weekly-reports-component';


@NgModule({
    imports: [
        RouterModule,
        ReportsRoutingModule
    ],
    declarations: [
        ReportsComponent,
        DailyReportCompoent,
        WeeklyReportsComponent
    ]
})

export class ReportsModule { }
```


**第二步，在reports-routing.module.ts里面设置child router**

```ts
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';

import { ReportsComponent } from './reports.component';
import { DailyReportCompoent } from './daily-reports/daily-reports-component';
import { WeeklyReportsComponent } from './weekly-reports/weekly-reports-component';
import { AdminGuard } from '../../core/guards/admin-guard';
import { permissionSets } from '../../core/app-constants';

const ReportsRoutes: Routes = [
    {
        path: '',
        component: ReportsComponent,
        data: { privileges: [permissionSets.APP_ManageReports] },
        canActivate: [AdminGuard],
        children: [{
            path: '',
            children: [
                { path: 'dailyreports', component: DailyReportCompoent },
                { path: 'weeklyreports', component: WeeklyReportsComponent }
            ]
        }]
    }
];


@NgModule({
    imports: [RouterModule.forChild(ReportsRoutes)],
    exports: [RouterModule]
})

export class ReportsRoutingModule { }
```


**第三步，在reports.component.html页面给child componet加上router-outlet**
```html
<div class="content">
    <h3>Reports Page</h3>
    <nav>
        <a routerLink="./dailyreports" routerLinkActive="active">Daily Reports</a>
        <a routerLink="./weeklyreports" routerLinkActive="active">Weekly Reports</a>
    </nav>
</div>
<router-outlet></router-outlet>
```


**第四步，在admin-guard.ts实现canActivateChild接口**

```ts
    canActivateChild(activatedRouteSnapshot: ActivatedRouteSnapshot): boolean {
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
```

**第五步，在在reports-routing.module.ts里面设置canActivateChild**

```ts
const ReportsRoutes: Routes = [
    {
        path: '',
        component: ReportsComponent,
        data: { privileges: [permissionSets.APP_ManageReports] },
        canActivate: [AdminGuard],
        children: [{
            path: '',
            canActivateChild: [AdminGuard],
            children: [
                { path: 'dailyreports', component: DailyReportCompoent },
                { path: 'weeklyreports', component: WeeklyReportsComponent }
            ]
        }]
    }
];
```

整个项目跑起来以后，在canActivateChild为false的情况下，是没有办法访问dailyreports weeklyreports这两个component。
