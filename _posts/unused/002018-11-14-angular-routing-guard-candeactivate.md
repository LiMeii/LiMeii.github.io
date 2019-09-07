---
title: Angular Router guards：如何提醒用户离开页面时保存数据 (CanDeactivate)
tags: Angular
layout: post
---


在前面几篇文章分别介绍了angular router guard中 CanLoad CanActivate 和 CanActivateChild的用法和区别。


这篇文章介绍angular router guard CanDeactivate的用法。


假设我们在一个注册网页上填写个人信息的时候，填到一半需要回到首页，不管是直接改变url或者是点击浏览器左上角的后退按钮，会直接去首页，填到一半的信息不会做任何处理，也不会保存。

这种行为不是很友好，做得好一点会弹出一个对话框提示用户有未保存的数据，CanDeactivate就是用来处理这种情况的。


源码可以在 [angular-seed-project](https://github.com/LiMeii/angular-seed-project) 查看。


**第一步，新加CanDeactivateGuard，并把它import到AuthGuardModule中，代码逻辑如下**


![angular](https://limeii.github.io/assets/images/posts/angular/angular-router-guard-candeactivate.png){:height="100%" width="100%"}

**第二步，在SettingsComponent中实现canDeactivate方法**


先按最简单的逻辑，只要离开settings页面就弹出一个对话框问用户是否要离开，confirm要离开返回值是true，否则就为false，代码逻辑如下：

![angular](https://limeii.github.io/assets/images/posts/angular/angular-router-guard-candeactivate-1.png){:height="100%" width="100%"}

**第三步，在SettingsComponent对应的路由上加上canDeactivate guard**

![angular](https://limeii.github.io/assets/images/posts/angular/angular-router-guard-candeactivate-2.png){:height="100%" width="100%"}


npm run start 以后，到settings页面，然后尝试到其他页面的时候，会有一个对话框跳出来，点击cancel后SettingsComponent中的canDeactivate返回值为false会留在当前页面，

点击confirm后SettingsComponent中的canDeactivate返回值为true会直接到reports页面，动图效果如下：

![angular](https://limeii.github.io/assets/images/posts/angular/angular-routing-guard-candeactivate-3.gif){:height="100%" width="100%"}


如果你想要实现，当页面数据有改动时，尝试离开当前页面弹出对话框，那么就把SettingsComponent中的canDeactivate方法的逻辑改一改就可以了。


以上的这个CanDeactivateGuard 适用于所有的component，也就是把这个CanDeactivateGuard用在ReportsComponent也会有同样的效果， 如果想要为某一特定的compoent实现对应的CanDeactivateGuard，可以参考如下代码：

```ts
import { Injectable }           from '@angular/core';
import { Observable }           from 'rxjs';
import { CanDeactivate,
         ActivatedRouteSnapshot,
         RouterStateSnapshot }  from '@angular/router';

import { CrisisDetailComponent } from './crisis-center/crisis-detail/crisis-detail.component';

@Injectable({
  providedIn: 'root',
})
export class CanDeactivateGuard implements CanDeactivate<CrisisDetailComponent> {

  canDeactivate(
    component: CrisisDetailComponent,
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): Observable<boolean> | boolean {
    return window.confirm('Do you want leave?');
  }
}
```

**总结**


假如项目中不同层级的router中包含 CanLoad CanActivate CanActivateChild CanDeactivate，那么router会从最里面的child router开始检查CanDeactivate 和 CanActivateChild，然后再从最外面的路由开始检查CanActivate，中间一旦有为返回值为false的，整个router校验就终止。如果有CanLoad，这个是在整个module load之前就校验了。