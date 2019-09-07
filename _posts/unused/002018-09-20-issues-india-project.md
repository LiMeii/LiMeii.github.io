---
title: 印度人的代码
tags: 问题
layout: post
---


之前公司把放在印度外包的一个做到一半的angular项目拿回来，这个项目是基于 angular4.2.4 和 webapck3.10.0做的。

做项目最痛苦的大概就是从印度人手里接手做到一半的项目了。

看到项目源码的时候是非常懵的，为了满足我司angular的要求，印度人强行把angular的代码包在.NET MVC里面，先用webpack打包生成好bundle文件，然后通过.NET Nuget编译打包上线。

#### 原来框架的大体结构是：

1) 整个 angular 前端代码放在 .NET MVC project下面的一个folder里



2) 项目 index 页面是 .NET 里的 index.cshtml


3) 项目所有 html/CSS/translation json/imgs 文件放在angular root 文件夹之外的MVC project里


4) 项目跑起来的一个流程是：


- a) webpack build，这个过程只打包compoent/service typescript文件，最后生成 app.bunlde.js / vendor.bundle.js / polyfills.bunlde.js 还有一些chunk文件
- b) 在MVC的.csproj文件里，把webpack没有打包的 html/CSS/JSON/imgs文件手动include到MVC project中。
- c) 在MVC的 index.cshtml文件里，手动引用 css/bundle文件。
- d) 最后是编译MVC Nuget，发布项目。



```html
.csproj文件是C#的工程文件，其中记录了与工程有关的相关信息，例如包含的文件，程序的版本，所生成的文件的类型和位置的信息等
```


#### 这样的结构最明显的几个问题：
1) MVC 在整个前端结构里到底有什么用？完全是多余的？完全get不到当时搭这个框架人的想法。


2) webpack 打包只打包typescript代码而且最后的bundle名字永远是 app.bunlde.js / vendor.bundle.js / polyfills.bunlde.js，其他的CSS/JSON/img 都不打包，有浏览器缓存问题。关于浏览器缓存问题可以参考 [缓存问题](https://limeii.github.io/2018/09/issues-cache-busting)。


3 index.cshtml文件是事先写好的并且是MVC框架里的，不是webpack build生成的，所以如果改成webpack contenthash打包文件的话，这个index.cthml有点麻烦。



拿到这个源码的时候，动手改之前，跟公司的release team确认了下，整个部署流程现阶段不能改，那就是MVC框架需要保留, index.cshtml 文件也需要保留。
能改的是：


1) webpack 打包方式，需要解决浏览器缓存问题。


2）解决每次build的时候，动态更新index.cshtml文件。


解决以上两个问题的详细可以参考：


[缓存问题](https://limeii.github.io/2018/09/issues-cache-busting)


[如何在webpack build结束后移动dist文件中的文件](https://limeii.github.io/2018/09/issues-webpack-file-management)

