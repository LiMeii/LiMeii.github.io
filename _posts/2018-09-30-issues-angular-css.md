---
title: Angular CSS编译问题
tags: 问题
layout: post
---


### 问题描述

在搭建Angular项目的时候，直接用angular-cli初始化项目，然后执行 'ng serve' 运行项目，完全正常没有任何问题。
之后ng eject angular-cli之后，自己配置webpack构建打包项目的时候，在浏览器console里面会有error：Expected 'styles' to be an array of strings，导致页面不会正常渲染。


![css error]( https://limeii.github.io/assets/images/posts/issues/issues-angular-css-builderror.png){:height="100%" width="100%"}

这个问题的RootCause是在webpack打包css的时候，'style-loader' 'css-loader'这些loader不能正常识别哪些css文件是global引用的还是在compoent单独引用导致的。

按照angular官方的best practice，项目的目录结构是如下：

![css error]( https://limeii.github.io/assets/images/posts/issues/issues-angular-project-structure.png){:height="100%" width="100%"}

在componet里面通过styleUrls可以找到对应的css文件，用angular-cli的时候编译都没问题，但是用自己配置的webpack打包的时候就会出现上面的问题。

google了下，在GitHub上发现angular的开发者是这么回复这个问题的：

![css error feedback]( https://limeii.github.io/assets/images/posts/issues/issues-angular-csserror-feedback.png){:height="100%" width="100%"}

回复是目前没办法修复这个问题，你们自己找黑魔法去修吧......

### 解决方案

**第一种方式：就直接用global的css文件呗，不用component独立的css文件**


具体就是直接把component对应的css文件删掉，并且把compoent里面的styleUrls这行代码删掉，所有的css都放在比如 assets/styles.css文件里就好了。


**第二种方式：还是保留componet的css文件，但是在compont里引用方式要改成如下**

```ts
  //styleUrls:['./app.component.css']
  styles:[require('./app.component.css').toString()]
```

**第三种方式， 在'style-loader', 'css-loader'的前面在加一个'to-string-loader'**

```js
{ test: /\.css$/, use: ['to-string-loader', 'style-loader', 'css-loader'] }
```

好啦，用以上三种任一一种方式就能解决这个问题了。