---
title: Angular：如何在Angular(8.0)中配置Webpack
tags: Angular
layout: post
---

在文章【[Angular：如何用Angular(<6.0)和Webpack搭建项目](https://limeii.github.io/2018/09/angular-webpack/)】中介绍了如果在angular项目中想要自己配置webpack，那么必须用命令‘ng eject’把angular内置的webpack.config文件暴露出来，然后根据项目需求自己重写整个webpack.config文件。


但是angular6.0以上的版本，angular官方去掉‘ng eject’这个命令。那么angular6.0+项目中，想要根据项目需求添加或者更改webpack打包配置要怎么做呢？【[angular-builders](https://github.com/just-jeb/angular-builders)】这个lib就是专门用来解决没有'ng eject'后怎么客户化配置项目的webpack打包方式。这篇文章会详细介绍在angular8.0中如何用【[angular-builders](https://github.com/just-jeb/angular-builders)】客户化配置webpack。

本文中用的项目代码在这里：【[angular-performance](https://github.com/LiMeii/angular-performance)】

## 用angular-cli创建一个angular8项目
'ng new angular-performance'创建angular项目。

本地开发环境如下：

![angular-customize-webpack](https://limeii.github.io/assets/images/posts/angular/angular-customize-webpack01.png){:height="100%" width="100%"}

## 安装angular-builders

运行命令：
```
npm install @angular-builders/custom-webpack --save-dev
npm install @angular-devkit/build-angular --save-dev
```
**不需要单独安装webpack和webpack-dev-server**，因为这两个是```@angular-devkit/build-angular```的依赖包，在安装```@angular-devkit/build-angular```会自动安装webpack和webpack-dev-server。

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
该错误表明本地有多个版本的webpack，把```package.json```文件里的webpack去掉，单独再安装```@angular-devkit/build-angular```就可以了。

## 更改angular.json中的配置

因为我们想要，在用angular-cli的前提下，可以把我们自己新加的客户化webpack配置加进去，可以实现效果：


**运行命令行ng server或者ng build的时候，可以结合已有内置webpack配置和新加的客户化webpack配置进行打包编译。**


所以需要更改angular.json中的配置，比如我们需要把客户化的webpack配置，加在运行（serve）和编译（build）命令里，那么关键的配置如下：

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
如果unit test也需要加自己的客户化webpack配置，可以加在对应test那个节点下。
</p>
</blockquote>
customWebpackConfig节点下的path就是指的我们需要新加的客户化webpack配置文件，然后再项目根目录下新加一个新的webpack.config.js文件，把项目里需要新加的一些webpack配置加在这个文件里就可以了。比如在示例项目里加了copy-webpack-plugin在编译过程中拷贝文件和webpack-bundle-analyzer用来编译打包好以后分析每个bundle文件内容，代码如下：

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
新加的客户化webpack文件不一定要命名为webpack.config.js，可以随意命名。而且可以在serve/build/test命令里添加各自不同的webpack配置文件，满足开发/发布/测试不同的配置需求。
</p>
</blockquote>

在做好以上的配置以后，运行```ng serve```或者```ng build```可以看到运行的结果是合并了angular内置webpack和我们新加的客户化webpack配置。

## 这么配置的优势是什么？

angular-cli内置的webpack配置，我们通过```ng build --aot=true``` 或者```ng build --prod```可以做AoT编译和tree-shaking，从而可以优化整个angular应用的性能。


因为内置webpack配置，开发没办法根据实际项目需求加一些内置webpack没有的loader或者plugin，在angular6.0以前的版本都是通过```ng eject```把内置webpack暴露出来，大多数开发根据自己项目需求重写整个webpack配置，那么同时也需要花额外的精力配合webpack去实现angular AoT编译和tree-shaking功能。


在angular6.0以上的版本，去掉```ng eject```，保留angular-cli（AoT编译和tree-shaking），并通过【[angular-builders](https://github.com/just-jeb/angular-builders)】满足开发客户化webpack配置需求。这么做的优势有：
1. 开发人员可以用最少的时间和精力，同时兼顾性能（AoT/tree-shaking）和项目特殊loader/plugin需求。
2. 不需要额外用awesome-typescript-loader来编译typescript
3. 不需要额外用angular-router-loader来实现lazy loading
4. 默认是Production mode
5. 默认是Minification
6. 默认是Uglification
7. 等等.....

<blockquote>
<p>
对于Angular6以上的版本， typescript编译（tsc）、Sass/Less编译成CSS、bundle打包JavaScript/CSS 、Code splitting、根据路由切割代码到不同的chunk文件，这些功能都是在 @ngtools/webpack里实现的，@ngtools/webpack的代码一般放在@angular/cli里。
</p>
</blockquote>