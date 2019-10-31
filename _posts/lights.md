
interview juejin books
- https://juejin.im/book/5bdc715fe51d454e755f75ef/section/5bdc71fbf265da6128599324

# interview
- https://github.com/jawil/blog/issues/22
- 说一下你了解CSS盒模型。
  - [后面有介绍]

- 说一下box-sizing的应用场景。
  - [后面有介绍]

- 说一下你了解的弹性FLEX布局. 
  - [后面有介绍]

- 说一下一个未知宽高元素怎么上下左右垂直居中。
  - [后面有介绍]

- 说一下原型链，对象，构造函数之间的一些联系。

- 说一下你项目中用到的技术栈，以及觉得得意和出色的点，以及让你头疼的点，怎么解决的 [TBD]

- 有没有了解http2.0,websocket,https，说一下你的理解以及你所了解的特性。[TBD]

- webpack的入口文件怎么配置，多个入口怎么分割 
  - [打包以后的bundle文件，其实就是一个立即执行函数+闭包，这个立即执行函数里有一个数组参数，这个数组的值是入口文件代码，如果入口文件里有import其他模块，数组里就有多个值，在立即执行函数的最后会把入口文件中的代码以数组索引放回，在代码执行的时候，用到某个模块，就通过这个数组索引把对象模块代码加载出来，并且会放到缓存里，下次再用到直接通缓存里拿]

- 简历上看见你了解http协议。说一下200和304的理解和区别
  - [200从服务器请求数据，还有就是强缓存没过期从缓存里加载，chrome开发工具里也是显示200，304协商缓存没有过期]

- DOM事件的绑定的几种方式 [TBD]

- 有没有了解http2.0,websocket,https，说一下你的理解以及你所了解的特性
  - [强缓存-协商缓存-cookie(httponly/secure)-HttpVSHttps-xss(csp)]

- DOM事件中target和currentTarget的区别 [TBD]

- 说一下你平时怎么解决跨域的。以及后续JSONP的原理和实现以及cors怎么设置。
  -[后面有详细介绍]
- 说一下深拷贝的实现原理。[TBD]

- 有没有去研究webpack的一些原理和机制，怎么实现的。
   -[立即执行函数+闭包，把入口文件代码作为IIFE的数组参数，数组index是模块的id，后续用webpackloader把通过moduleid从模块中加载出来执行]

- babel把ES6转成ES5或者ES3之类的原理是什么，有没有去研究。[TBD]

- 什么是函数柯里化？以及说一下JS的API有哪些应用到了函数柯里化的实现？[TBD]

- ES6的箭头函数this问题，以及拓展运算符 [TBD]

- JS模块化Commonjs,UMD,CMD规范的了解，以及ES6的模块化跟其他几种的区别，以及出现的意义
  - [后面有介绍]

- 怎么获取一个元素到视图顶部的距离。[TBD]
  - [Element.getBoundingClientRect()方法返回元素的大小及其相对视口的位置]
    ![web-getBoundingClientRect](/assets/images/posts/web/getBoundingClientRect.png){:height="100%" width="100%"}
    这个方法的返回结果是DOMRect对象，它的属性值如下：
    ![web-getBoundingClientRect](/assets/images/posts/web/getBoundingClientRect-result.png){:height="100%" width="100%"}
    **需要注意的是，这个方法放回的是相对视图顶部的距离，如果需要获得相对网页左上角定位的属性值，那么就需要给top left属性值加上当前的滚动位置（window.pageXOffset 和 window.pageYOffset）**
    
- getBoundingClientRect获取的top和offsetTop获取的top区别 
  - [一个是相对位置距离，一个是加上了滚动的距离]

- 事件委托 [TBD]

- 比如说百度的一个服务不想让阿里使用，如果识别到是阿里的请求，然后跳转到404或者拒绝服务之类的？ [TBD]

- 二分查找的时间复杂度怎么求，是多少 [TBD]

- XSS是什么，攻击原理，怎么预防。
  - [后面有详细介绍]

- 白板写代码，用最简洁的代码实现数组去重。[TBD]

- https://mp.weixin.qq.com/s/OUeoshYYui9EsB8SC3D6MA

- script下载执行和css渲染之间的顺序
  -[script下载解析会阻塞DOM/CSSOM树/渲染树的生成]

# 浏览器渲染原理 
- 浏览器接收到HTML文件，从上到下从左到右，解析HTML文件，构建DOM树，将每个元素都看成一个节点，所有节点结合起来就是一个DOM树

- 当解析HTML文件的时候，浏览器会遇到CSS和JS文件，浏览器会去下载并解析这些文件

- 解析CSS文件，并把CSS文件转换为CSSOM树，这个过程非常消耗资源，浏览器需要递归CSSOM树，确定具体的的元素是什么样式。向下面这种写法，第二种要比第一种效率更差，第一种直接给所有的span标签设置颜色，第二种需要先找到所有sapn标签，然后找到span标签上的a标签，最后要找到div标签，然后给符合这些条件的span设置颜色，这个递归过程就已经非常复杂了。**所以我们在写css的时候，尽可能的避免写过于具体的css选择器，对于HTML来说也尽量少的添加无意义的标签，保证层级扁平。**
  ```html
    <div>
    <a> <span></span> </a>
    </div>
    <style>
        span {
            color: red;
        }
        div > a > span {
            color: red;
        }
    </style>
  ```
- 在生成DOM和CSSOM树以后，会将这两棵树合为渲染树，在这个过程中并不是简单的合并，渲染树里只会包括需要显示的节点和样式信息，```display:none```的，就不会在渲染树里显示。生成渲染树以后，就会根据渲染树进行布局（也可以叫做回流），然后调用GPU绘制，合成图层，显示在屏幕上。

   ![web-browser](/assets/images/posts/web/borwser-renderTree.png){:height="100%" width="100%"}


- **为什么JS操作DOM性能很慢**，JS是JS引擎运行，DOM是渲染引擎，两个是不同的线程，JS操作DOM就意味着这两个线程之间需要通信，会有性能损耗。操作DOM次数越多，那么这两个线程之间的通信也就越多，也就会导致性能问题。

- **插入几万个DOM，如何实现页面不卡顿**，通过```requestAnimationFrame```

- **什么情况会阻塞渲染** 
   - 渲染的前提是生成渲染树，所以HTML和CSS文件下载肯定会阻塞渲染。如果想要渲染的越快，降低渲染文件大小，扁平CSS层级，优化选择器。
   - 浏览器在解析```script```标签的时候，会暂停DOM，script解析完以后，会继续构建DOM树，首屏加载的时候，按需加载，还就是就是把script标签放在body标签底部，或者是在标签里添加```defer``` 或者 ```async``` 属性。
      - defer表示JS会并行下载，但是会放在HTML解析完以后顺序执行，所以可以把script标签放在任意位置。
      - 对于没有任何依赖关系的JS可以加上async属性，表示JS文件下载和解析不会阻塞渲染。

- **重绘（Repaint）和回流（Reflow）**，如果这两个在设置节点样式的时候频繁出现，同时也会在很大程度上影响性能
   - 重绘是指在节点需要改外观但是不影响布局，比如改变color就要重绘
   - 回流是布局或者几何属性需要改变
   
   - 回流一定会导致重绘，但是重绘不一定会导致回流，重绘和回流其实也和Eventloop有关
     - 当Eventloop执行完Microtasks后，会判断document是否需要更新，因为浏览器是60Hz的刷新率，每16.6ms才会更新一次。
     - 然后判断是否有```resize```或者```scroll```事件，有的话会触发事件，所以resize和scroll事件也是16.6ms才触发一次，并且自带节流功能
     - 判断是否触发media query
     - 更细动画平且发送事件
     - 判断的是否有全屏操作事件
     - 执行```requestAnimationFrame```回调
     - 执行```IntersectionObserver ```回调，该方法用于判断元素是否可见，用于懒加载，但是兼容性不好
     - 跟新页面
     - 以上就是一帧中可能会做的事情。如果在一帧中有空闲时间，就会去执行```requestIdleCallback``` 回调。



- **减少重绘和回流**
  - 使用 transform 替代 top
  ```html
    <div class="test"></div>
    <style>
    .test {
        position: absolute;
        top: 10px;
        width: 100px;
        height: 100px;
        background: red;
    }
    </style>
    <script>
    setTimeout(() => {
        // 引起回流
        document.querySelector('.test').style.top = '100px'
    }, 1000)
    </script>
  ```

  - 使用 visibility 替换 display:none，前者只会引起重绘，后者会引发回流（改变了布局）

  - 不要把节点的属性放在一个循环里当成循环里的变量
  ```js
    for(let i = 0; i < 1000; i++) {
        // 获取 offsetTop 会导致回流，因为需要去获取正确的值
        console.log(document.querySelector('.test').style.offsetTop)
    }
  ```

  - 不要使用 table 布局，可能很小的一个小改动会造成整个 table 的重新布局

  - 动画实现数度选择，动画速度越快，回流次数越多，也可以选择使用 requestAnimationFrame

  - CSS选择器**从右向左**匹配查找，避免节点层级过多

  - 将频繁重绘或者回流的节点设置为图层，图层能够阻止该节点的渲染行为影响别的节点。比如对于 video 标签来说，浏览器会自动将节点变为图层
   ![web-browser](/assets/images/posts/web/borwser-imglayer.png){:height="100%" width="100%"}
    will-change / video / iframe标签都会生成新图层

- **不考虑缓存和优化网络协议的前提下，可以通过哪些方式来最快的渲染页面**
  - 如何测量到底有没有加快渲染速度呢？
     ![web-browser](/assets/images/posts/web/borwser-domContentLoaded.png){:height="100%" width="100%"}
     当 DOMContentLoaded 事件后，就会生成渲染树，生成渲染树就可以进行渲染了，这一过程更大程度上和硬件有关系了。
  
  - 提示如何加速：
    - 从文件大小考虑
    - 从 ```<script>``` 标签使用上考虑
    - 从CSS HTML的代码书写上来考虑
    - 从需要下载的内容是否需要在首屏使用上来考虑
    - 如果是框架写的应用，像Angular，考虑使用AoT,而不是JiT，这样就把编译的时间省下来了。

- **<font color="red">JS下载解析 CSS下载解析 图片资源下载解析，DOM CSSOM树构建，渲染树构建之间的阻塞关系</font>**
  - 总结来说：JS全阻塞，CSS半阻塞（会阻塞JS执行和渲染树）
  - JS阻塞DOM CSSOM树的构建，再就是阻塞渲染树构建而且阻塞其他静态资源下载（图片），原因是JS会操作DOM，所以一旦开始下载解析JS，会阻塞HTML解析和渲染。**所以要把```<script>```标签放到body最后**
  - CSS文件下载解析，不会阻塞HTML文档解析，不会阻塞DOM CSSOM树的构建，但是会阻塞渲染树，因为渲染树是有DOM和CSSOM树合并成的，css文件的link可以放在head里，不影响。
  - CSS文件下载同样也不会阻塞其他文件下载，但是会阻塞JS的文件的执行（原因之一是JS执行代码可能依赖于css样式）。
  - 现在浏览器可以在```<link>```标签里设置```prefetch```属性，浏览器在获得 HTML 文档之后会对页面上的引用资源提前进行下载

# 输入URL到页面渲染的整个流程 （www.google.com）
 - 首先是DNS查询，如果这一步做了智能DNS解析的话，会提供访问速度最快的IP地址回来
   -  DNS的作用就是通过域名查询到具体的IP
   -  操作系统会首先在本地缓存中查询IP
   -  没有的话去系统配置的DNS服务器中查询
   -  如果这时候还没有的话，回去DNS根服务器查询，这一步查询会找出扶着com这个一级域名的服务器
   -  然后去改服务器查询 google 这个二级域名
   -  接下来三级域名的查询其实是我们配置的
   以上是通过DNS迭代查询，还有种是递归查询，区别是前者是由客户端做请求，后者是由系统配置的DNS服务器做请求，得到结果后返回给客户端

 - 接下来是TCP三次握手，然后是TLS的四次握手，握手成功，之后就是正式传输数据
 - 客户端发送请求去服务器请求文件或数据，服务器响应文件或数据
 - 客户端得到响应以后，会判断状态码，如果是200就继续解析，如果是400/500就会报错，如果是300就会重定向
 - 浏览器开始解析文件，如果是gzip格式会先解压一下，然后通过文集爱你的编码格式知道如何去解码文件
 - 之后开始渲染流程 **在前面有介绍**
 - 

# 模块化
- 解决命名冲突
- 提供复用性
- 提高代码可维护性

- 其实就是立即执行函数 + 闭包
- 解决了命名冲突问题 （JS作用域是基于funtion，加上变量提升，导致很多变量和函数表达式都是全局变量，虽然放在了不同的JS文件里，还是会有命名冲突）
- 代码复用 （模块化以后，把另一个模块作为立即执行函数的参数，就可以实现引用其他的模块里的方法和属性，实现代码复用）

- MVVM的框架都是基于模块化
- webpack打包方式其实也是基于 立即执行函数 + 闭包，把入口文件作为立即执行函数的参数，以数组的形式，数组的index就是模块的ID，在代码执行的时候，直接用webpack的模块加载器通过moduleID从内存中把对应的代码加载起来，从而可以在JS引擎里运行相应的代码。

- 模块化的发展有，CommonJS AMD CMD， 之后才有了ES6中的module

## 闭包
- 函数执行完以后，直接从执行栈中pop出去了，完全处于失联状态，没有办法再访问它里面的任何变量，在内存中等着垃圾回收。闭包就是实现一个引用，虽然它已经不在执行栈上，但是还是可以通过这个引用访问这个函数里的变量。

- 常用的实现方式是，在函数A里，定义一个函数B返回A里的变量或者方法，然后在返回B，因为在执行上下文中，函数B除了有词法记录和scope，其中scope指向了它上一层方法A的词法作用域。这样就是通过方法B可以访问到失联方法A的变量和方法。
  ```js
    function foo() {
        var a = 123;

        var bar = function (b) {
            a = a + b;
            console.log("this is the value in foo: " + a);
        }

        var baz = function () {
            return {
                a: a,
                bar: bar
            }
        }
        return baz;
    }
  ```
- 还有一种方式是，把方法里的变量绑定到window对象上，这样方法执行完，从执行栈上pop出去，但还是可以通过window对象访问到方法内部的方法和变量
  ```js
    (function foo() {
        var person = window.person = {
            name: "frank",
        }
    })()
  ```


# 跨域
- ```<img src=XXXX> <link href=XXX> <script src=XXX>```去服务器下载，是允许跨域加载资源
- 九种跨域方式实现原理（完整版）https://github.com/ljianshu/Blog/issues/55

- **JSONP：**：利用```<script>``` 标签没有跨域限制的漏洞，网页可以拿到其他来源动态产生的JSON数据，JSONP请求一定需要对方的服务器支持才可以做到，缺点就是只支持GET方法。JSONP实现流程如下：
    -- 声明一个回调函数，函数名（show）当作参数值，要传递给跨域请求的服务器，函数形参为获取目标数据（服务器返回的data）
    -- 创建一个```<script>```，把那个跨域的API数据接口地址赋值给```<script>```的```src```，还要在这个地址中向服务器传递该函数名（可以通过问好传参？callboack=show）
    -- 服务器接收到请求后，需要进行特殊的处理，把传递进来的函数名和它需要给你的数据拼接成一个字符串，传递进去的函数名是show，他准好的数据是show（"call back data"）
    -- 最后服务器把准备的数据通过HTTP协议返回给客户端，客户端再调用执行之前声明的回调函数（show），对方会的数据进行操作。

- **CORS：**主要是服务器端在http header里设置 ```Access-Control-Allow-Origin```，该属性表示哪些域名可以访问资源，如果设置通配符就表示所有网站都可以访问。复杂请求（非简单请求）需要先发一个option预检请求，然后再发真正的请求。简单请求是：同时满足 条件一 get head post；条件二 Content-Type = text/plain,multipart/form-data,application/x-www-form-urlencoded
  
- **document.domain**
- **postMessage**

- **nginx反向代理**，原理和proxy一样的：都是先把request发到中间代理，中间代理设置了cors，中间代理再把request转发给服务器，服务器的response发给中间代理，中间代理再把response发给前端。

- **nodejs proxy** 

- IIS之间为什么不需要要配置跨域：首选跨域只是针对浏览器的同源策略，后端服务器之间是不存在跨域的问题；所有的前端请求都是先发到前端IIS server上，在前端IIS上配置了URL rewrite，会把相应的http请求发到不同的后端IIS服务上，IIS配置如下：
  ![web-cross-domain](/assets/images/posts/web/crossdomain-IIS.png){:height="100%" width="100%"}

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


# 安全
- **XSS（跨站脚本攻击）**
  - 比如在页面，有一个输入框，用户A输入一段恶意的```<script></script>```，并保存在输入库，用户B访问该页面拿到这段恶意脚本，浏览器会执行该脚本，从而可以窃取用户的cookie等信息，或者让用户重定向到A写的恶意网站，或者下载一些恶意程序，从而进行一些非法操作，比如转账啊
  - 如何攻击
  - 如何防御
    -- 从用户输入角度来看：对特殊字符```<``` ```>```进行转义，页面再次渲染这些的时候，就不会当成脚本执行，而是直接当成内容显示。（对HTML充分转义）
    -- 从server端对cookie设置来看：security=true，这样在前端浏览器里是没有办法读取和修改cookie的值。
    -- 严格的输入长度控制
    -- CSP（内容安全策略）：在http header中设置```Content-Security-Policy```, 让浏览器指执行从白名单里获取到的脚本文件，而忽略所有的其他脚本。
      --- 比如所有内容均来自同一个源：```Content-Security-Policy: default-src 'self'```
      --- 允许内容来自信任的域名以及子域名：```Content-Security-policy: default-src 'self' *.trusted.com```
    

- **CSRF（跨站请求伪造）**
  - **如何攻击**
    - https://tech.meituan.com/2018/10/11/fe-security-csrf.html
    - 受害者登入a.com，并保留了登录凭证（cookie）
    - 攻击者引诱受害者访问了b.com
    - b.com向a.com发送了一个请求：a.com/act=xx，浏览器会默认携带a.com的cookie
    - a.com接收到请求后，对请求进行验证，并确认是受害者的凭证，误以为是受害者自己发送的请求
    - a.com以受害者的名义执行了act=xx
    - 攻击完成，攻击者在受害者不知情的情况下，冒充受害者，让a.com执行了自己定义的操作

  - **如何防御** 利用CSRF token
    - 在用户打开页面的时候，服务器端生产一个随机的token，然后通过JS遍历整个DOM数，对DOM中所有的a和form标签加入token。
    - 页面提交的请求就会携带这个token
    - 服务器验证token是否正确，会有一个算法，对比token和cookie的值。

    ```html
    <form action="/transfer.do" method="post">
        <input type="hidden" name="CSRFToken"
            value="OWY4NmQwODE4ODRjN2Q2NTlhMmZlYWEwYzU1YWQwMTVhM2JmNGYxYjJiMGI4MjJjZDE1ZDZMGYwMGEwOA==">
            ...
    </form> 
    ```

- **密码安全**
  -加盐

- **cookie设置**
  - Secure=true 表示该cookie在http中打开无效，只能在https中打开
  - HttpOnly = true，表示cookie只能在服务器端设置值，不能在浏览器通过```document.cookie```改变cookie值。
  - SameSite = true，表示不能跨域访问cookie


# 协议
- http vs https: https = http + TSL/SSL安全协议，通过散列算法实现数字签名保证请求的完整性，通过非对称密码实现握手建立安全通道（四次握手），通过对称加密实现数据传输
  - http 基于传输层，https基于TLS/SSL安全协议
  - http 明文传输不安全，https加密传输 保证数据安全和网站性
  - http 标准端口是80，https标准端口是443

- 200和304的理解和区别
  - 强缓存失效以后，会带着```If-None-Match``` ```If-Modified-Since```发一个请求到服务器，服务对比这两个值，如果没有过期就返回一个304的response告诉客户端缓存还没有过期，可以直接用本地缓存，如果已经过期就会发一个200的response，新的资源返回给客户端。
  - 强缓存是通过 expires 和 cache-control来控制，expires是http1.0的产物 cache-control是http1.1的产物，同时用这两个是为了兼容http1.0协议，cache-contorl的优先级要高
  - 协商缓存是通过 last-modify 和 ETag来控制，在强缓存过期的情况下，会把这两个值分别以```If-Modified-Since``` ```If-None-Match``` 带到服务器端做检验

- 常见的http状态码
  - 2×× 成功
    - 200，表示客户端发来的请求在服务器被正确处理
    - 204 No Content，表示请求成功，但是响应报文不含实体的主体部分
    - 205 reset content，表示请求成功，但响应报文不含实体的主体部分，但是与204响应不同字啊与要求对方重置内容
    - 206 partial content，进行范围请求
  - 3×× 重定向
    - 301 move poermanently，永久重定向，表示资源已被分配了新的URL
    - 302 found，临时性重定向，表示资源临时被分配了新的URL
    - 303 see other，表示资源存在着另一个URL，应该使用GET方法获取资源
    - 304 not modified，表示服务器允许访问资源，但是发生请求未满足条件的情况，比如缓存未过期
    - 307 temporary redirect，临时重定向，和302含义类似，但是期望客户端保持请求方法不变向新的地址发出请求、
  - 4×× 客户端错误
    - 400 bad request，请求报文存在语法错误，比如api接口传的参数不对
    - 401 unauthorized，表示发送的请求需要有通过HTTP认证的认证信息
    - 403 forbidden，表示对请求资源的访问被服务器拒绝
    - 404 not found，表示在服务器上没有找到请求的资源
  - 5×× 服务器错误
    - 500 internal server error，表示服务器端在执行请求时发生了错误
    - 501 not implemented，表示服务器不支持当前请求所需要的某个功能
    - 503 service unavailable，表示服务器暂时处于超负载或者正在停机维护，无法处理请求  


# 盒子模型

![css-box-model](/assets/images/posts/css/box-model.jpg){:height="100%" width="100%"}

IE 和 W3C标准的区别就是，宽度计算的方式不一样，IE的计算方式是：```width=content+padding+border```，而W3C只是```width=content```。两者是通过```box-sizing```进行转换。

```css
*, *:before, *:after {
　　-moz-box-sizing: border-box;
  　-webkit-box-sizing: border-box;  /* border-box是指 width=content+padding+border； content-box: width=content；默认是content-box*/
　　box-sizing: border-box;
　　}
```
使用场景是，如果有好几个元素都设置宽度为100%，实际宽度有长有短，就加上```box-sizing：border-box```，就可以了。或者是本身是几个元素的宽度是一样的，其中一个设置border后导致宽度发生变化或者位置发生变化，就加上```box-sizing：border-box```也可以解决。


# FLEX弹性布局
- 水平垂直居中
```html
<style>
    .container {
        display: flex;
        justify-content: center;  /* 水平居中 */
        align-items: center; /* 垂直居中 */
        width: 1000px;
        height: 600px;
        background-color: black;
    }

    .box {
        width: 80px;
        height: 80px;
        background-color: red;
    }
</style>

<div class="container">
    <div class="box">
    </div>
</div>

```

# bytedance 
 - 算法： https://leetcode-cn.com/explore/interview/card/bytedance



