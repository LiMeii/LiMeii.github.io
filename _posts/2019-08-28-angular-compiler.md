---
title: Angular：深入理解Angular编译机制
tags: Angular
layout: post
---

在这篇文章中会介绍以下内容：

- 为什么Angular需要编译。

- Angular编译机制：JiT vs AoT。

- AoT的工作原理（ngc）。

- AoT对性能的影响。


## 为什么Angular需要编译

Angular是基于TypeScript，编译打包的时候会用tsc将TypeScript编译成es5文件，这样在浏览器JavaScript Virtual Machines (VM)可以直接运行es5代码。那么Angular为什么还需要编译呢？在Angular中除了TypeScript之外，还有HTML模板文件，在这些模板文件里有Angular自带的组件、指令、管道等等，为了让浏览器可以识别和运行这些东西，那么就需要用Angular编译器把这些编译成浏览器可识别和运行的es5代码。

## Angular编译机制：JiT vs AoT

Angular的编译器有两种执行机制：JiT和AoT，```ng build```和```ng serve```是JiT的编译方式，```ng build --prod``` ```ng build --aot``` 或者```ng serve --aot```是AoT的编译方式。一句话概括两者的区别：Angular编译器（ngc）执行的时机不一样，JiT是浏览器在渲染页面的时候先把Angular编译器下载到本地，然后把HTML模板编译成浏览器可识别运行的es5代码；AoT是项目在打包的时候就把HTML模板编译成浏览器可识别运行的es5代码，在浏览器渲染时不需要下载Angular编译器也不需要编译，直接运行这些代码就可以了。整个流程如下：

### JiT
```
- 基于TypeScript开发Angular项目
- 用tsc编译Angular项目
- 打包
- Minification
- 部署

当用户访问这个网站的时候：

- 下载相关的静态资源文件，包括Angular编译器（@angular/compiler）
- 启动Angular
- Angular编译器执行编译（ngc）
- 页面渲染

```

### AoT
```
- 基于TypeScript开发Angular项目
- 编译Angular项目
  - 先把模板和component编译成TypeScript/es6（ngc）
  - 再把TypeScript编译成es5 (tsc)
- 打包
- Minification
- 部署

当用户访问网站的时候：

- 下载相关的静态资源文件，不需要下载Angular编译器（@angular/compiler）
- 启动Angular
- 页面渲染
```

从上面的流程可以看出，用AoT编译方式打包，在页面渲染的时候不需要下载Angular编译器代码，也不需要执行编译，而是直接渲染页面。从性能上来说，AoT要胜过JiT。除了编译时机不同，我们还可以通过代码来看看两种编译方式的bundle文件有什么区别。创建一个Angular项目，然后运行```ng build``` ```ng build --prod```来看看最后编译文件有什么不同。


示例源码在这里：【[angular-performance](https://github.com/LiMeii/angular-performance)】

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

其中deep-understanding-aot组件，用来显示当前时间，代码如下：

```ts
// deep-understanding-aot.component.ts
import { Component, OnInit } from "@angular/core";

@Component({
    selector:"app-deepunderstand-compiler",
    templateUrl: "./deep-understanding-aot.component.html"
})

export class DeepUnderstandingComponent implements OnInit {
    public title ="let's learn angular compiler together！";
    public time = new Date();
    ngOnInit() {
        setInterval(()=>{
            this.time = new Date();
        },1000)
    }
}
```

```html
<!-- deep-understanding-aot.component.html -->
<div class="margin-large">
    <h1>{{title}}</h1>
    <h2>here is current time:  { {  time | date: 'hh:mm:ss a'  } } </h2>
</div>
```


**运行ng build（JiT）**


编译之后的文件如下：
![angular-compiler](/assets/images/posts/angular/angular-compiler01.png){:height="100%" width="100%"}

我们可以看到vendor文件最大，vendor-es5.js有3882KB，vendor-es2015.js有3856KB。我们可以通过工具【[source-map-explorer](https://github.com/danvk/source-map-explorer)】来看看vendor文件里到底有什么。

```
//先安装source-map-explorer
npm install -g source-map-explorer

// 运行如下命令，查看vendor-es5.js
npx source-map-explorer dist/angular-performance/vendor-es5.js
```
vendor-es5.js文件结构如下;
![angular-compiler](/assets/images/posts/angular/angular-compiler02.png){:height="100%" width="100%"}

我们可以看到因为JiT是在页面渲染的时候编译模板文件，所以需要打包compiler源码，compiler源码就有1.32M！DeepUnderstandingComponent对应的bundle文件(JiT)是：modules-deep-understanding-aot-deep-understanding-aot-module-es5.js，我们来看看文件内容：

![angular-compiler](/assets/images/posts/angular/angular-compiler05.png){:height="100%" width="100%"}

从上面的图片我们可以看到，JiT编译在打包过程中不会编译模板文件，HTML直接以内联的方式放在bundle文件里，保留了Angular的 { { } } 和 date pipe；component只是把TypeScript语法编译成了es5的语法。

**运行ng build:prod（AoT）**


默认```ng build:prod```不会产生source map文件，由于我们需要用source-map-explorer工具分析bundle文件结构，所以需要在运行```ng build:prod```也产生source map文件。需要改动angular.json文件配置，改动如下：

```
//angular.json

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

编译之后的文件如下：
![angular-compiler](/assets/images/posts/angular/angular-compiler03.png){:height="100%" width="100%"}

我们看到main文件最大，其中main-es5.241e3ea4617330a70446.js有275KB。

```
// 运行如下命令，查看main-es5.241e3ea4617330a70446.js
npx source-map-explorer dist/angular-performance/main-es5.241e3ea4617330a70446.js
```
main的文件结构如下：
![angular-compiler](/assets/images/posts/angular/angular-compiler04.png){:height="100%" width="100%"}

我们可以看到由于AoT在打包的时候编译模板文件，所以不需要把打包compiler源码，bundle文件明显减小了很多！DeepUnderstandingComponent对应的bundle文件(AoT)是4-es5.4c969b6ad4c13630ad20.js，我们来看看里面的内容：

![angular-compiler](/assets/images/posts/angular/angular-compiler06.png){:height="100%" width="100%"}

整个文件都被minify和uglify，通过关键字```here is current time```搜索，可以看到Angular的 { { } } 和 date pipe这些都被编译es5代码。


来总结一下JiT和AoT的主要区别：
- 打包之后的文件大小不一样，最大的区别是JiT需要把compiler源码打包进bundle文件，而AoT不需要。
- bundle文件的内容也有很大区别：JiT把HTML模板直接内联进bundle文件不做任何处理，AoT会把HTML模板文件编译es5文件。
- 编译执行的时机不一样，JiT是在浏览器里执行，需要内联的HTML文件编译成es5代码；AoT是在编译打包的时候就把HTML文件编译成es5代码。

接下来看看，AoT是如何把模板文件编译成es5文件的。

## AoT的工作原理（ngc）

AoT编译实际分了两个步骤：
- 第一步：用ngc把模板和component编译成es6（或者是TypeScript代码，可以在tsconfig.json里配置）。
- 第二步：用tsc把这些TypeScript编译成es5。

```ng build --prod```只能看到第二步的es5文件，那么有没有办法看到第一步的es6文件呢？当然有的，可以运行ngc命令，我们在```package.json```的scripts节点里加上如下命令：

```
  "scripts": {
    ......
    ......
    "compile": "ngc"
    ......
    ......
  },

```

运行```npm run compile```，我们来看看deep-understanding-aot组件在ngc编译完以后是什么样的：

![angular-compiler](/assets/images/posts/angular/angular-compiler07.png){:height="100%" width="100%"}

编译之后的目录是在```tsconfig.json```文件的outDir定义的，配置如下：

```
  "compilerOptions": {
    "baseUrl": "./",
    "outDir": "./dist/out-tsc",
    "sourceMap": true,
    ......
    ......
  },
```
ngc编译的输出文件目录结构和真正的项目目录结构一样，在对应的文件夹下会有以下文件：
- ```*.metadata.json```：把.ts(component/NgModule)文件里的decorator信息和constructor的依赖注入信息用json的形式记录下来，下次在二次编译的时候不需要再从.ts文件里拿了。二次编译是指在自己项目中引用第三方库，在编译自己项目的时候需要对第三方库进行二次编译打包。如果我们自己项目要用AoT编译，那么第三方库必须要提供.metadata.json文件。

- ```*.ngfactory.js```：里面包含了创建组件、渲染组件(涉及DOM操作)、执行变化检测(获取oldValue和newValue对比)、销毁组件的代码，也就是我们说的component view。

- ```*.js```：是.ts(component/NgModule)文件里除decorator和constructor之外的内容，编译成了es6代码。

- ```*.ngsummary.json```：经包含了.metadata.json中所有的信息，因此如果使用了ngsummary.json，就不需要.metadata.json了。

对于```*.metadata.json``` ```*.js```和```*.ngsummary.json```都很好理解就不详细一一介绍，我们来看看```*.ngfactory.js```文件里到底有什么内容，同样以DeepUnderstandingComponent为例，ngc产生的ngfactory文件为deep-understanding-aot.component.ngfactory.js，内容如下：

```js
/**
 * @fileoverview This file was generated by the Angular template compiler. Do not edit.
 *
 * @suppress {suspiciousCode,uselessCode,missingProperties,missingOverride,checkTypes}
 * tslint:disable
 */
import * as i0 from "@angular/core";
import * as i1 from "@angular/common";
import * as i2 from "./deep-understanding-aot.component";
var styles_DeepUnderstandingComponent = [];
var RenderType_DeepUnderstandingComponent = i0.ɵcrt({ encapsulation: 2, styles: styles_DeepUnderstandingComponent, data: {} });
export { RenderType_DeepUnderstandingComponent as RenderType_DeepUnderstandingComponent };

export function View_DeepUnderstandingComponent_0(_l) {
    return i0.ɵvid(0,

        [i0.ɵpid(0, i1.DatePipe, [i0.LOCALE_ID]),
        (_l()(), i0.ɵeld(1, 0, null, null, 5, "div", [["class", "margin-large"]], null, null, null, null, null)),
        (_l()(), i0.ɵeld(2, 0, null, null, 1, "h1", [], null, null, null, null, null)),
        (_l()(), i0.ɵted(3, null, ["", ""])),
        (_l()(), i0.ɵeld(4, 0, null, null, 2, "h2", [], null, null, null, null, null)),
        (_l()(), i0.ɵted(5, null, ["here is current time: ", ""])),
        i0.ɵppd(6, 2)],

        null,

        function (_ck, _v) {
            var _co = _v.component; var currVal_0 = _co.title;
            _ck(_v, 3, 0, currVal_0);
            var currVal_1 = i0.ɵunv(_v, 5, 0, _ck(_v, 6, 0, i0.ɵnov(_v, 0), _co.time, "hh:mm:ss a"));
            _ck(_v, 5, 0, currVal_1);
        }
    );
}

export function View_DeepUnderstandingComponent_Host_0(_l) {
    return i0.ɵvid(0,
        [(_l()(), i0.ɵeld(0, 0, null, null, 1, "ng-component", [], null, null, null,
            View_DeepUnderstandingComponent_0, RenderType_DeepUnderstandingComponent)),
        i0.ɵdid(1, 114688, null, 0, i2.DeepUnderstandingComponent, [], null, null)
        ],
        function (_ck, _v) { _ck(_v, 1, 0); }, null);
}
var DeepUnderstandingComponentNgFactory = i0.ɵccf("ng-component", i2.DeepUnderstandingComponent, View_DeepUnderstandingComponent_Host_0, {}, {}, []);
export { DeepUnderstandingComponentNgFactory as DeepUnderstandingComponentNgFactory };
//# sourceMappingURL=deep-understanding-aot.component.ngfactory.js.map
```

我们可以看到在ngfactory文件里有:
- ```View_{COMPONENT}{COUNTER}```(View_DeepUnderstandingComponent_0) 是：the internal component，负责(根据template)渲染出组件的视图和进行变化检测。

- ```View_{COMPONENT}_Host{COUNTER}```(View_DeepUnderstandingComponent_Host_0) 是：the internal host component，负责渲染出宿主元素 < app-compiler > < / app-compiler > ，并且使用"the internal component"管理组件的内部视图，也是通过这个构建整个组件树。


在View_DeepUnderstandingComponent_0的视图创建和变化检测代码如下：
![angular-compiler](/assets/images/posts/angular/angular-compiler08.png){:height="100%" width="100%"}

关系图如下：
![angular-compiler](/assets/images/posts/angular/angular-compiler09.png){:height="100%" width="100%"}


总结来说：AoT之后，HTML模板文件会被编译成一个视图(es6)，在这个视图里会把页面元素(div h1 h2)都渲染出来，并且生成绑定和变化检测代码，同时也host component的信息。

## AoT对性能的影响
不管是JiT还是AoT，最终的结果文件是一样的。JiT编译的bundle文件，在浏览器渲染之前需要下载compiler源码，然后compiler源码对JiT的bundle文件进行编译，也会在本地生成```*.ngfactory.js```文件，最后进行渲染。在浏览器里先编译再渲染肯定没有AoT直接在浏览器里渲染性能高。


除此之外，还有一点好处是，AoT把模板文件编译成es6文件，可以做Tree-Shaking，也会提高性能。