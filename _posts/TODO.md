
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
- 词法环境(Lexical Environments) [done]
- 运行上下文(Execution Context)、函数创建、函数的运行 [done]
- 闭包 [done]
- this [done]
- 作用域链 [done]
- 原型到原型链 [done]
- 对象创建
- 继承
- 模块化
- promise 
- generator
- async await

- 内置类型
- Typeof
- 类型转换
  - 转Boolean
  - 对象转基本类型
  - 四则预算符
  - == 操作符
  - 比较运算符

- 原型 [done]
- new
- instanceof
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
- promise实现
- Generator实现
- Map FlatMap和Reduce
- async和await
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

# nodejs

# Http

# 算法


# 浏览器
- 事件机制
  - 事件触发三阶段
  - 注册事件
  - 事件代理

- 跨域
  - JSONP
  - CORS
  - document.domain
  - postMessage

- Event Loop
  - Node 中的Event Loop
  - timer
  - I/O
  - idle prepare
  - poll
  - check
  - close callbacks

- 存储
  - cookie localStorage
  - sessionStorage indexDB
  - Service Worker

- 渲染机制
  - load和DOMContentLoaded 区别
  - 图层
  - 重绘（repaint）和回流（reflow）
  - 减少重绘和回流


# 性能
- 网络相关
  - DNS预解析
  - 缓存
  - 强缓存
  - 协商缓存
  - 选择合适的缓存策略
  - 使用HTTP/2.0
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

# react/vue

# 微信小程序开发

