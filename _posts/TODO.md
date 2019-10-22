
# Angular change detection series:
- change detection [done]
- change detection: unidirectional data flow && ExpressionChangedAfterItHasBeenCheckedError error  [done]
- lefe cycle and hooks [done]
- change detection: ngdocheck execute timing [done]
- angular change detection strategy: observable onpush immutable [done]

# Angular performance series:
- https://github.com/mgechev/angular-performance-checklist#lazy-loading-of-resources
- how to make your angular app faster: onpush strategy [done]
- how to make your angular app faster: tree shaking [done]
- how to make your angular app faster:  simple ngFor / ChangeDetectorRef.detach()
- angular service work 
  - https://angular.io/guide/service-worker-intro
- Use pure pipes
- *ngFor directive Use trackBy option

- how to make your angular app faster: AoT (ahead-of-time compilation) [done!!!!!!!]
  - https://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/
  - https://blog.jiayihu.net/angular-aot-compilation-with-webpack/
  - https://segmentfault.com/a/1190000011562077
  - https://blog.angularindepth.com/a-deep-deep-deep-deep-deep-dive-into-the-angular-compiler-5379171ffb7a

- ngzone introduce
  - how to make your angular app faster: NgZone.runOutsideAngular()
  - https://blog.angularindepth.com/i-reverse-engineered-zones-zone-js-and-here-is-what-ive-found-1f48dc87659b
  - https://blog.angularindepth.com/do-you-still-think-that-ngzone-zone-js-is-required-for-change-detection-in-angular-16f7a575afef

- Server-Side Rendering
  - Angular Universal
  - Preboot


- Web Workers 

- Ivy renderer 


# Angular userful directive/component
- searchable dropdown
- numberic
- numberic with no start 0
- float with specific length
- alphabet with no special characters

# Angular series:
- ng eject(angluar<6.0), then config webpack [done]
- angular6+, angular-builders to config customize webpack configuration [done]
- ng-bootstrap [code-done]
- ngrx
- angular material
- router
- multiple https will be canceled in angular4


# RxJS series
- RxJS intorduce [done]
- RxJS code vs hot: [done]
  - .publish().refCount() [done]
  - .publish() connect [done]
  - .share() [done]
  - .publishReplay(1).refCount() [done]
  - shareReplay() [done]
- RxJS Subject BehaviorSubject ReplaySubject AsyncSubject [done]
- RxJS angular && http [done]
- RxJS cache [done]
- RxJS notification [done]
- RxJS ConnectableObservable connect refCount [done]
- RxJS Subscription [done]

- why need unsubsrible, httpclient need unsubscribe? takeuntil first ect operator will unsubscribe ? [done]
  - https://medium.com/@benlesh/rxjs-dont-unsubscribe-6753ed4fda87
  - https://stackoverflow.com/questions/35042929/is-it-necessary-to-unsubscribe-from-observables-created-by-http-methods
  - https://stackoverflow.com/questions/38008334/angular-rxjs-when-should-i-unsubscribe-from-subscription

- how to debug RxJS
  - https://staltz.com/how-to-debug-rxjs-code.html 

- forkjoin zip combineLatest withLastFrom

- RxJS map mergeMap switchMap contactMap flatMap

- RxJS input search with debounce [done]
- RxJS scheduler
- RxJS VS Promise VS Generator
- RxJS marble diagrams
- RxJS common operator
- RxJS error handle
  - catchError / catchError chain
  - throwError
  - Finalize / finalize chain
  - retry
  - retryWhen
  - delayed retry
  

# webpack
- code spliting in webpack4

# JS
- event loop task miscrotask/job [done]
- 浏览器与Node的事件循环(Event Loop)有何区别?
- 词法环境(Lexical Environments) [done]
- 运行上下文(Execution Context)、函数创建、函数的运行 [done]
- 闭包 [done]
- this [done]
- 作用域链 [done]
- 原型到原型链 [done]
- 对象创建 [done]
- 继承: 6种继承方式和优缺点，ES6中的class是怎么实现继承的。https://github.com/ljianshu/Blog/issues/20 [done]
- 说一下原型链，对象，构造函数之间的一些联系 https://github.com/jawil/blog/issues/13 [done]
- 对象创建的多种方式 https://github.com/mqyqingfeng/Blog/issues/15
- 模块化
- promise [done]
- generator [done]
- async await [done]
- Map Set

- 内置类型
- Typeof
- instanceof
- 类型转换 https://github.com/jawil/blog/issues/1
  - 转Boolean
  - 对象转基本类型
  - 四则预算符
  - == 操作符
  - 比较运算符
- JavaScript的数据类型及其检测
- JavaScript数据类型转换
- 如何判断相等性： https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Equality_comparisons_and_sameness

- 原型 [done]
- new [done]
- this [done]
- 执行上下文 [done]
- 闭包 [done]
- 深浅拷贝 [done]
- 模块化
  - CommonJS
  - AMD
- 防抖
- 节流
- 继承
- call applay bind区别 [done]
  - 模拟实现call和apply [done]
- Map FlatMap和Reduce
- proxy
- 为什么 0.1+0.2！==0.3
- undefined vs null
- 正则表达式
  - 元字符
  - 修饰语
  - 字符简写
- V8下的垃圾回收机制


# ES6
-  let 和 const [done]

# TypeScript

# nodejs

# Http
- TCP 三次握手四次挥手： https://github.com/jawil/blog/issues/14 [done]
- SSL/TLS 四次握手 [done]
- HTTPS工作原理 [done]

# 算法


# 浏览器 Web
- DNS 解析
- 浏览器路由原理（history）
- 工作原理： https://juejin.im/post/5a6547d0f265da3e283a1df7
- 事件机制
  - 事件触发三阶段
  - 注册事件
  - 事件代理

## 跨域
  - ```<img src=XXXX> <link href=XXX> <script src=XXX>```去服务器下载，是允许跨域加载资源
  - 九种跨域方式实现原理（完整版）https://github.com/ljianshu/Blog/issues/55

  - **JSONP：**：利用```<script>``` 标签没有跨域限制的漏洞，网页可以拿到其他来源动态产生的JSON数据，JSONP请求一定需要对方的服务器支持才可以做到，缺点就是只支持GET方法。JSONP实现流程如下：
    -- 声明一个回调函数，函数名（show）当作参数值，要传递给跨域请求的服务器，函数形参为获取目标数据（服务器返回的data）
    -- 创建一个```<script>```，把那个跨域的API数据接口地址赋值给```<script>```的```src```，还要在这个地址中向服务器传递该函数名（可以通过问好传参？callboack=show）
    -- 服务器接收到请求后，需要进行特殊的处理，把传递进来的函数名和它需要给你的数据拼接成一个字符串，传递进去的函数名是show，他准好的数据是show（"call back data"）
    -- 最后服务器把准备的数据通过HTTP协议返回给客户端，客户端再调用执行之前声明的回调函数（show），对方会的数据进行操作。

  - **CORS：**主要是服务器端在http header里设置 Access-Control-Allow-Origin，该属性表示哪些域名可以访问资源，如果设置通配符就表示所有网站都可以访问。复杂请求（非简单请求）需要先发一个option预检请求，然后再发真正的请求。简单请求是：同时满足 条件一 get head post；条件二 Content-Type = text/plain,multipart/form-data,application/x-www-form-urlencoded
  
  - **document.domain**
  - **postMessage**

  - **nginx反向代理**，原理和proxy一样的：都是先把request发到中间代理，中间代理设置了cors，中间代理再把request转发给服务器，服务器的response发给中间代理，中间代理再把response发给前端。

  - **nodejs proxy** 
  
  - IIS之间为什么不需要要配置跨域：首选跨域只是针对浏览器的同源策略，后端服务器之间是不存在跨域的问题；所有的前端请求都是先发到前端IIS server上，在前端IIS上配置了URL rewrite，会把相应的http请求发到不同的后端IIS服务上，IIS配置如下：
  ![js-prototype](/assets/images/posts/web/crossdomain-IIS.png){:height="100%" width="100%"}


- Event Loop
  - Node 中的Event Loop
  - timer
  - I/O
  - idle prepare
  - poll
  - check
  - close callbacks

- 存储
  - cookie localStorage [done]
  - sessionStorage indexDB [done]
  - Service Worker

- 渲染机制
  - load和DOMContentLoaded 区别
  - 图层
  - 重绘（repaint）和回流（reflow）
  - 减少重绘和回流

- 深入理解浏览器的缓存机制 [done]
- 从URL输入到页面展现到底发生什么？
- 深入理解HTTPS工作原理 [done]
- why chrome will cancel https, when multiple http request send out at the same time.
- Ajax原理
- H5新特性

# 性能
- 网络相关
  - DNS预解析
  - 缓存 [done]
  - 强缓存 [done]
  - 协商缓存 [done]
  - 选择合适的缓存策略 [done]
  - Service Worker
  - 使用HTTP/2.0 push
  - 预加载
  - 预渲染
- 优化渲染过程
  - 懒执行
  - 懒加载
- 文件优化
  - 图片优化
  - 计算图片大小
  - 图片加载优化
  - 其他文件优化
  - CDN
- 其他
  - 使用webpack优化项目
  - 监控


# 安全
- XSS
  - 如何攻击
  - 如何防御
  - CSP
- CSRF
  - 如何攻击
  - 如何防御
- 密码安全
  -加盐





# interview
- https://github.com/jawil/blog/issues/22
- 说一下你了解CSS盒模型。
- 说一下box-sizing的应用场景。
- 说一下你了解的弹性FLEX布局.
- 说一下一个未知宽高元素怎么上下左右垂直居中。
- 说一下原型链，对象，构造函数之间的一些联系。
- 说一下你项目中用到的技术栈，以及觉得得意和出色的点，以及让你头疼的点，怎么解决的
- 有没有了解http2.0,websocket,https，说一下你的理解以及你所了解的特性。
- webpack的入口文件怎么配置，多个入口怎么分割啥的
- 简历上看见你了解http协议。说一下200和304的理解和区别
- DOM事件的绑定的几种方式
- 有没有了解http2.0,websocket,https，说一下你的理解以及你所了解的特性
- DOM事件中target和currentTarget的区别
- 说一下你平时怎么解决跨域的。以及后续JSONP的原理和实现以及cors怎么设置。
- 说一下深拷贝的实现原理。
- 有没有去研究webpack的一些原理和机制，怎么实现的。
- babel把ES6转成ES5或者ES3之类的原理是什么，有没有去研究。
- 什么是函数柯里化？以及说一下JS的API有哪些应用到了函数柯里化的实现？
- 说一下深拷贝的实现原理。
- 有没有去研究webpack的一些原理和机制，怎么实现的。
- ES6的箭头函数this问题，以及拓展运算符
- JS模块化Commonjs,UMD,CMD规范的了解，以及ES6的模块化跟其他几种的区别，以及出现的意义
- 怎么获取一个元素到视图顶部的距离。
- getBoundingClientRect获取的top和offsetTop获取的top区别
- 事件委托
- 比如说百度的一个服务不想让阿里使用，如果识别到是阿里的请求，然后跳转到404或者拒绝服务之类的？
- 二分查找的时间复杂度怎么求，是多少
- XSS是什么，攻击原理，怎么预防。
- 白板写代码，用最简洁的代码实现数组去重。
- https://mp.weixin.qq.com/s/OUeoshYYui9EsB8SC3D6MA

## bytedance 算法： https://leetcode-cn.com/explore/interview/card/bytedance

## 介绍项目，你做了什么？为什么要这么做。

## 遇到过什么问题，解决方案是什么，学到了什么

## angular框架类 js基础的知识点
### angularjs vs angular2
- 变化检测不一样：
   - anuglar2+ 通过ngzone捕获到所有的异步事件，然后触发整个组件树从上到下进行变化检测，可以通过onpush变化策略，减少组件的变化检测；
   - angularjs的脏检查机制至少要跑两遍保证数据稳定，如果有变量a b，b依赖a的变化 a依赖b，angularjs的变化检测就死循环了，要跑十次变化检测才会停止，angular2+ 使用单向数据流，提高了脏检查机制性能
   如有方法不再angularjs的脏检查机制fa
- angularjs中的module其实并不是真正模块的概念，它的controller service才是真正模块，这也是在不同module里用了重名的service后面调用的service会覆盖前一个service而导致问题的原因；angular2+的module就是ES6的模块
- angular2+ 有内置的编译器ngc
- angular2+ 有tree-shaking   
- angularjs 没有自己的路由lib，需要引用第三方的lib，没有实现lazy loading
- angularjs 通过promise处理http callback； angular2+通过rxjs处理http callback




# react/vue

# 微信小程序开发


