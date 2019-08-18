---
title: Angular：如何用Angular(<6.0)和Webpack搭建项目
tags: Angular
layout: post
---

## angular-cli
按照[Angular官方教程](https://angular.io/tutorial)搭建一个Angular项目，默认都是先安装angula-cli，再用命令行'ng new angular-project-name'会自动生成一个符合angular best practice的项目， 如下所示：


![angular seed project]( https://limeii.github.io/assets/images/posts/angular/angular-seed-project.png){:height="100%" width="100%"}

源码可以在 [angular-seed-project](https://github.com/LiMeii/angular-seed-project) 查看。

<blockquote>
<p>
本文中的示例代码用的angular-cli的版本是1.6.0，angular是5.0.0
</p>
</blockquote>
然后直接跑'ng serve'就把项目跑起来了，用angluar-cli是不是很方便直接，对初学者也很友好，这个过程中都不需要配置webpack。

其实angular-cli打包这一块的代码，里面源码用的也是webpack，为了降低初学者的学习曲线，angular把webpack内置了，默认用cli创建的项目里是没有把webpack.config文件暴露给开发者，不让开发改webpack配置代码，也就是说不能加我们想要用的plugin loader，也不能更改bunlde文件命名和优化bundle文件大小等等。

如果需要写自己的webpack，官方提供了ng eject这个命令，把webpack.config文件暴露出来，一旦eject就不能再用angular-cli，编译打包就完全让开发自己控制。


## ng eject

首先在terminal里执行 'ng eject'

![angular eject]( https://limeii.github.io/assets/images/posts/angular/angular-seedproject-eject.png){:height="100%" width="100%"}

<blockquote>
<p>
需要注意的是 Angular6 以上版本目前不支持ng eject。
'ng eject'以后就不能再用ng的命令了，需要在package.json中的script配置你自己的命令。
</p>
</blockquote>

执行eject命令以后，会在根目录下新加 webpack.config.js 文件，这个是angular-cli默认的一个webpack配置，大致的内容如下：

![angular default webpack]( https://limeii.github.io/assets/images/posts/angular/angular-default-webpack.png){:height="100%" width="100%"}


接下来就教你如何在angular中搭建你自己的webpack打包方式。

## 如何在angular中配置webpack

**第一步，在src目录下新增一个vendor.ts文件，这个文件主要是用来引用第三方library，比如node_modules下面的library**


在vendor文件里improt第三方library默认是从node_modules目录下找，所以如果是node_modules里面的library，那么node_modules之前的路径都可以省略

```ts
//vendor.ts
// Angular 2
import "@angular/platform-browser";
import "@angular/platform-browser-dynamic";
import "@angular/core";
import "@angular/common";
import "@angular/http";
import "@angular/router";
import "@angular/forms";

// RxJS
import "rxjs/Rx";
```

**第二步，更改webpack.config.js**

```js
module.exports = function(env){
  console.log(env);
  return require(`./webpack.${env}.js`);
}
```

**第三步，创建本地开发用的打包方式，在根目录下新增一个webpack.dev.js文件**


关于webpack.dev完整代码可以查看 [webpack.dev.js](https://github.com/LiMeii/angular-seed-project/blob/master/webpack/webpack.dev.js)


**第四步，在package.json文件script节点里增加命令**

```js
 "scripts": {
    "start": "webpack-dev-server --env=dev --hot --inline --port 3000 --open\"",
    "build:dev": "webpack --env=dev --progress --profile --colors"
  }
```
然后在terminal里输入 'rum run start' 能在本地把整个项目run起来，每次改动 ts css html 文件会自动编译刷新页面，即时就能看到页面变化。

执行'npm run build:dev'这个命令行，会在根目录下生成build-dev文件目录，在这个目录下是最终开发环境下编译打包后的bundle文件。

![angular build dev]( https://limeii.github.io/assets/images/posts/angular/angular-build-dev-file.png){:height="100%" width="100%"}


**第五步，创建Production的打包方式，在根目标下新增一个webpack.prod.js文件**


关于webpack.prod完整代码可以查看 [webpack.prod.js](https://github.com/LiMeii/angular-seed-project/blob/master/webpack/webpack.prod.js)

需要在package.json文件script节点下加以下命令：

```js
 "build:prod": "webpack --env=prod --progress --profile --colors"
```
然后在terminal中执行'npm run build:prod'这个命令行，在根目录节点下生成dist文件目录，这个目录包含最终发布到PROD的bundle文件。

![angular build prod]( https://limeii.github.io/assets/images/posts/angular/angular-build-prod-file.png){:height="100%" width="100%"}

好啦，现在就把 Angular+Webpack 项目搭起来啦。
