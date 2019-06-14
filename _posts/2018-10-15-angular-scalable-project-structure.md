---
title: 如何定义一个可扩展的Angular项目结构
tags: Angular
layout: post
---


最开始搭angular项目，曾经对项目结构，哪些文件放在哪个folder纠结了好久。


这篇文章主要是用来总结，我自己在搭angular项目过程中对项目结构理解，希望能帮到有同样困扰的小伙伴。

### angular 项目结构

用angular-cli快速建一个angular项目，基本结构如下：

![angular-base-structure.png](https://limeii.github.io/assets/images/posts/angular/angular-base-structure.png){:height="100%" width="100%"}

如果新加的文件都放在app文件夹下，随着项目功能越来越多，新加的 moldue / componet/ service/ 单元测试等文件越来越多，module 和 componet混在一起，整个项目结构会非常乱，也不利于后期代码维护和管理。


下面这张图是我用到的项目结构：

![angular-scalable-structure.png](https://limeii.github.io/assets/images/posts/angular/angular-scalable-structure.png){:height="100%" width="100%"}

### core 文件夹

core文件夹展开以后如下：
![angular-structure-core.png](https://limeii.github.io/assets/images/posts/angular/angular-structure-core.png){:height="100%" width="100%"}

项目启动还是默认的AppModule，core这个文件夹主要是用来放一些，项目里公用的一些 coponent/ service/ feature 等等，只需要在项目AppModule里引用一次，项目启动以后实例化一次以后全局都能用。


authentication这个目录下放的是整个项目的login，logout，forget password，结构如下：

```ts
|-- authentication
     |-- login
          |-- login.component.ts|html|spec.ts|service.ts
     |-- authentication-routing.module.ts
     |-- authentication.module.ts
```


header footer是全局component，结构如下：


```ts
|-- header
     |-- header.component.ts|html|spec.ts|service.ts
|-- footer
     |-- footer.component.ts|html|spec.ts|service.ts
```

http文件夹下面包含了整个项目公用的http方法和后台返回错误的处理方法。


```ts
|-- http
     |-- http-error-handler.service.ts|spec.ts
     |-- http.api.service.ts|spec.ts
```


constants这个文件夹里包含了全局一些常量，结构如下：


```ts
|-- constants
     |-- api-url-constants.ts
     |-- app-constants.ts
```


guards这个文件夹主要是放angular route guards，主要是一些interfaces用来告诉路由是否可以访问模块，可以简单的理解为页面权限管理。


shared文件夹主要是用来放公用的一些service，比如页面一些validation service。


mocks文件夹是用来放前端开发的mock data。


### modules 文件夹

这个文件用来放业务模块代码，结构如下：


```ts
|-- modules
     |-- dashboard
          |-- dashboard.component.ts|html|spec.ts|service.ts
          |-- dashboard-routing.module.ts
          |-- dashboard.module.ts
     |-- settings
          |-- settings.component.ts|html|spec.ts|service.ts
          |-- settings-routing.module.ts
          |-- settings.module.ts
```


### shared 文件夹

这个文件用来放一些公用的 pipes/filters/directive/components，结构如下：


```ts
|-- shared
     |-- components
          |-- button.component.ts|spec.ts
     |-- directives
          |-- onlynumber.directive.ts|spec.ts
     |-- pipes
          |-- capitalize.pipe.ts
```


### assets 文件夹

这个文件文件夹里就放css imgs


当然不同的项目需求，也会导致项目结构的差异，以上的项目结构是我自己基于现有项目需求定义的。