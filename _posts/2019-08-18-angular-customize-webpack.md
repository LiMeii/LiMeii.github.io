---
title: Angular：如何在 Angular(8.0) 中配置 Webpack
tags: Angular
layout: post
---

在文章【[Angular：如何用Angular(<6.0)和Webpack搭建项目](https://limeii.github.io/2018/09/angular-webpack/)】中介绍了如果在 Angular 项目中想要自己配置 webpack，那么必须用命令‘ng eject’把 Angular 内置的 webpack.config 文件暴露出来，然后根据项目需求自己重写整个 webpack.config 文件。


但是 Angular6.0 以上的版本，Angular 官方去掉‘ng eject’这个命令。那么 Angular6.0+ 项目中，想要根据项目需求添加或者更改 webpack 打包配置要怎么做呢？【[angular-builders](https://github.com/just-jeb/angular-builders)】这个 lib 就是专门用来解决没有'ng eject'后怎么客户化配置项目的 webpack 打包方式。这篇文章会详细介绍在 Angular8.0 中如何用【[angular-builders](https://github.com/just-jeb/angular-builders)】客户化配置 webpack。

本文中用的项目代码在这里：【[angular-performance](https://github.com/LiMeii/angular-performance)】

## 用 angular-cli 创建一个 Angular8 项目
'ng new angular-performance'创建 Angular 项目。

本地开发环境如下：

![angular-customize-webpack](https://limeii.github.io/assets/images/posts/angular/angular-customize-webpack01.png){:height="100%" width="100%"}

## 安装 angular-builders

运行命令：
```
npm install @angular-builders/custom-webpack --save-dev
npm install @angular-devkit/build-angular --save-dev
```
**不需要单独安装 webpack 和 webpack-dev-server**，因为这两个是```@angular-devkit/build-angular```的依赖包，在安装```@angular-devkit/build-angular```会自动安装 webpack 和 webpack-dev-server。

需要注意的是，就算在初始化的```package.json```中已经有```@angular-devkit/build-angular```，还是需要手动再运行一次命令行：
```
npm install @angular-devkit/build-angular --save-dev
```
否则在```ng serve```运行项目的时候，可能会看到类似下面的错误：

```
14% building modules......
Error: No module factory available for dependency type: ContextElementDependency
......
```
该错误表明本地有多个版本的 webpack，把```package.json```文件里的 webpack 去掉，单独再安装```@angular-devkit/build-angular```就可以了。

## 更改 angular.json 中的配置

因为我们想要，在用 angular-cli 的前提下，可以把我们自己新加的客户化 webpack 配置加进去，可以实现效果：


**运行命令行 ng server 或者 ng build 的时候，可以结合已有内置 webpack 配置和新加的客户化 webpack 配置进行打包编译。**


所以需要更改 angular.json 中的配置，比如我们需要把客户化的 webpack 配置，加在运行（serve）和编译（build）命令里，那么关键的配置如下：

```json
   ......
    "architect": {
        ......
        "build": {
            "builder": "@angular-builders/custom-webpack:browser",
            "options": {
                "customWebpackConfig": {
                    "path": "./webpack.config.js"
                }
            }
        },
        "serve": {
            "builder": "@angular-builders/custom-webpack:dev-server",
            "options": {
                "customWebpackConfig": {
                    "path": "./webpack.config.js"
                }
            }
        }
    }
```
<blockquote>
<p>
如果 unit test 也需要加自己的客户化 webpack 配置，可以加在对应 test 那个节点下。
</p>
</blockquote>
customWebpackConfig 节点下的 path 就是指的我们需要新加的客户化 webpack 配置文件，然后再项目根目录下新加一个新的 webpack.config.js 文件，把项目里需要新加的一些 webpack 配置加在这个文件里就可以了。比如在示例项目里加了 copy-webpack-plugin 在编译过程中拷贝文件和 webpack-bundle-analyzer 用来编译打包好以后分析每个 bundle 文件内容，代码如下：

```js
const webpack = require('webpack');
const pkg = require('./package.json');
const path = require('path');
const CopyPlugin = require('copy-webpack-plugin');
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = (config, options) => {
  config.plugins.push(
    new webpack.DefinePlugin({
      'APP_VERSION': JSON.stringify(pkg.version),
    }),
    new BundleAnalyzerPlugin(),
    new CopyPlugin([
      { from: path.join(__dirname, "moackdata"), to: path.join(__dirname, "dist/angular-performance/moackdata") }
    ])
  );

  return config;
};
```

<blockquote>
<p>
新加的客户化 webpack 文件不一定要命名为 webpack.config.js，可以随意命名。而且可以在 serve/build/test 命令里添加各自不同的 webpack 配置文件，满足开发/发布/测试不同的配置需求。
</p>
</blockquote>

在做好以上的配置以后，运行```ng serve```或者```ng build```可以看到运行的结果是合并了 Angular 内置 webpack 和我们新加的客户化 webpack 配置。

## 这么配置的优势是什么？

angular-cli 内置的 webpack 配置，我们通过```ng build --aot=true``` 或者```ng build --prod```可以做 AoT 编译和 tree-shaking，从而可以优化整个 Angular 应用的性能。


因为内置 webpack 配置，开发没办法根据实际项目需求加一些内置 webpack 没有的 loader 或者 plugin，在 Angular6.0 以前的版本都是通过```ng eject```把内置 webpack 暴露出来，大多数开发根据自己项目需求重写整个 webpack 配置，那么同时也需要花额外的精力配合 webpack 去实现 Angular AoT 编译和 tree-shaking 功能。


在 Angular6.0 以上的版本，去掉```ng eject```，保留 angular-cli（AoT 编译和 tree-shaking），并通过【[angular-builders](https://github.com/just-jeb/angular-builders)】满足开发客户化 webpack 配置需求。这么做的优势有：
1. 开发人员可以用最少的时间和精力，同时兼顾性能（AoT/tree-shaking）和项目特殊 loader/plugin 需求。
2. 不需要额外用 awesome-typescript-loader 来编译 typescript
3. 不需要额外用 angular-router-loader 来实现 lazy loading
4. 默认是 Production mode
5. 默认是 Minification
6. 默认是 Uglification
7. 等等.....

<blockquote>
<p>
对于 Angular6 以上的版本， typescript 编译（tsc）、Sass/Less 编译成 CSS、bundle 打包 JavaScript/CSS 、Code splitting、根据路由切割代码到不同的 chunk 文件，这些功能都是在 @ngtools/webpack 里实现的，@ngtools/webpack 的代码一般放在 @angular/cli 里。
</p>
</blockquote>