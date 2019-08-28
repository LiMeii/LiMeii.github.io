---
title: Angular：深入理解Angular编译机制
tags: Angular
layout: post
---

在这篇文章中会介绍以下内容：

- 为什么Angular需要编译。

- Angular编译机制：JiT vs AoT。

- AoT的工作原理。

- AoT对性能的影响。


## 为什么Angular需要编译

Angular是基于TypeScript，编译打包的时候会用tsc将TypeScript编译成es5文件，这样在浏览器JavaScript Virtual Machines (VM)可以直接运行es5代码。那么Angular为什么还需要编译呢？在Angular中除了TypeScript之外，还有HTML模板文件，在这些模板文件里有Angular自带的组件、指令、管道等等，为了让浏览器可以识别和运行这些东西，那么就需要用Angular编译器把这些编译成浏览器可识别和运行的JS代码。

## Angular编译机制：JiT vs AoT

Angular的编译器有两种执行机制：JiT和AoT，默认运行```ng build```和```ng serve```的时候是JiT的编译方式，```ng build --prod``` ```ng build --aot``` 或者```ng serve --aot```都是AoT的编译方式。一句话概括两者的区别：Angular编译器（ngc）执行的时机不一样，JiT是浏览器在渲染页面的时候先把Angular编译器下载到本地，然后把HTML模板编译成浏览器可识别运行的JS代码；AoT是项目在打包的时候就把HTML模板编译成浏览器可识别运行的JS代码，在浏览器渲染时不需要下载Angular编译器也不需要编译，直接运行这些代码就可以了。整个流程如下：

### JiT
```
- 基于TypeScript开发Angular项目
- 用tsc编译Angular项目
- 打包
- Minification
- 部署
```

当用户访问这个网站的时候：
```
- 下载相关的静态资源文件，包括Angular编译器（ngc）
- 启动Angular
- Angular编译器执行编译
- 页面渲染
```
### AoT
```
- 基于TypeScript开发Angular项目
- 用ngc编译Angular项目
  - 先把模板文件编译成TypeScript（tsc）
  - 再把TypeScript编译成JS
- 打包
- Minification
- 部署
```

当用户访问网站的时候：
```
- 下载相关的静态资源文件，不需要下载Angular编译器（ngc）
- 启动Angular
- 页面渲染
```

从上面的流程可以看出，用AoT编译方式打包，在页面渲染的时候不需要下载Angular编译器代码，也不需要执行编译，而是直接渲染页面。从性能上来说，AoT要胜过JiT。除了编译时机不同，我们还可以通过代码来看看两种编译方式最后的文件有什么区别。创建一个Angular项目，然后运行```ng build``` ```ng build --prod```来看看最后编译文件有什么不同。


源码在这里：【[angular-performance](https://github.com/LiMeii/angular-performance)】

代码结构如下：
```
--src
 -- app
   -- modules
     -- dashboard
       - dashboard.component.html
       - dashboard.component.ts
       - dashboard.module.ts
     -- deep-understanding-aot
       - deep-understanding-aot.component.html
       - deep-understanding-aot.component.ts
       - deep-understanding-aot.module.ts
   -- shared
 - index.html
 - main.ts
 - styles.css
```

**运行ng build（JiT）**


编译之后的文件如下，
![angular-compiler](https://limeii.github.io/assets/images/posts/angular/angular-compiler01.png){:height="100%" width="100%"}

我们可以看到Vendor文件最大，vendor-es5.js有3882KB，vendor-es2015.js有3856KB。我们可以通过工具【[source-map-explorer](https://github.com/danvk/source-map-explorer)】来看看vendor文件里到底有什么。

```
//先安装source-map-explorer
npm install -g source-map-explorer

// 运行如下命令，查看vendor-es5.js
npx source-map-explorer dist/angular-performance/vendor-es5.js
```
vendor-es5.js文件结构如下;
![angular-compiler](https://limeii.github.io/assets/images/posts/angular/angular-compiler02.png){:height="100%" width="100%"}

我们可以看到因为JiT是在页面渲染的时候编译模板文件，所以需要打包compiler源码，compiler源码就有1.32M！


**运行ng build:prod（AoT）**


默认ng build:prod不会产生source map文件，但是我们需要用source-map-explorer工具分析bundle文件结构，所以需要在运行ng build:prod也产生source map文件。需要改动angular.json文件配置，改动如下：

```
      "architect": {
        "build": {
            ......
            ......
          "configurations": {
            "production": {
              ......
              ......
              "sourceMap": true, // 把sourceMap设置为True
              ......
              ......
            }
          }
        }
      }
```

![angular-compiler](https://limeii.github.io/assets/images/posts/angular/angular-compiler03.png){:height="100%" width="100%"}

我们看到main文件最大，其中main-es5.241e3ea4617330a70446.js有275KB。

```
// 运行如下命令，查看main-es5.241e3ea4617330a70446.js
npx source-map-explorer dist/angular-performance/main-es5.241e3ea4617330a70446.js
```
main的文件结构如下：
![angular-compiler](https://limeii.github.io/assets/images/posts/angular/angular-compiler04.png){:height="100%" width="100%"}

我们可以看到，AoT是在打包的时候编译模板文件，所以不需要把打包compiler源码，bundle文件明显减小了很多！