---
title: 用node代理解决前后端联调跨域问题
tags: JS
layout: post
---


最近做的项目，后端API是用的.NET， 前端开发用的是Angular4, 是一个前后端分离的项目。在本地开发的时候，我用node express搭了一个proxy用来解决前后端跨域联调的问题。


最近项目来了新人，每次带新人开发的时候，都会问到为什么要用这个node proxy，本地开发的时候直接调用后端DEV-INT服务器上的API不就好了吗？ 每次都解释这个是用来解决联调跨域的，刚工作的同事基本都是懵比状态，有经验的一般会问，不是在服务器端设置好CROS就可以了，但是对于代理实现跨域并不是很了解。在这篇文章里就会解释为什么用要跨域以及如果用node proxy解决联调跨域问题。


**为什么会有跨域？**

浏览器有一个[同源策略](https://en.wikipedia.org/wiki/Same-origin_policy), 这里的同源指的是以下三个要一直：

```
- 协议相同
- 域名相同
- 端口相同
```

这样做的目的是为了保证用户信息安全，防止恶意的网站攻击。现在如果是非同源，以下三种行为会受到限制：

```
1. cookie localstorage 和 indexdb 无法读取
2. DOM 无法获得
3. AJAX 请求不能发送
```

基于浏览器这个同源策略，会导致前后端分离开发项目势必会存在跨域问题，比如前端本地开发的时候request url为：https://localhost:4200/useragents/requesttokens 后端开发写好这个API后发布到DEV-INT服务器上暴露出来的API是：https://expamle.com/useragents/requesttokens, 这样就是域名不一样，端口也不一样，就是非同源，如果在前端代码里直接调用https://expamle.com/api/useragents/requesttokens 我们来看一下会有什么问题:

![node proxy]( https://limeii.github.io/assets/images/posts/js/node-proxy-1.png){:height="100%" width="100%"}

![node proxy]( https://limeii.github.io/assets/images/posts/js/node-proxy-2.png){:height="100%" width="100%"}

因为这是一个非简单请求，浏览器在发送真正的请求之前都会发一个预检请求，post method为options。浏览器先询问服务器，当前网络所有在的域名是否在服务器许可名单之中，以及可以使用哪些HTTP方法和头信息字段。只有得到肯定答复，浏览器才会发出正式的XMLHttpRequest请求，否则就报错。从上面截图的错误信息可以看出来，服务器端没有设置CROS跨域，这就是我们说的跨域问题。

**node proxy来解决跨域问题**

这个代理具体做以下几个步骤：

##### 1. 接受客户端请求
##### 2. 将请求转发给服务器
##### 3. 拿到服务器响应数据
##### 4. 将响应数据转发给客户端

具体代码如下：

```js
//app.js
var createError = require('http-errors');
var express = require('express');
var path = require('path');
var cookieParser = require('cookie-parser');
var logger = require('morgan');
var request = require('request');
var cors = require('cors');

var app = express();

// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'jade');

app.use(logger('dev'));
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));


function setResHeader(res) {
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, OPTIONS, PUT, PATCH, DELETE'); // If needed
  res.setHeader('Access-Control-Allow-Headers', 'Authorization,Access-Control-Allow-Headers, Origin,Accept, X-Requested-With, Content-Type, Access-Control-Request-Method, Access-Control-Request-Headers,RequestID,X-Content-Type-Options,X-Content-Type-Options,X-Frame-Options,X-Powered-By,X-Version,x-xss-protection,Strict-Transport-Security') // If needed
  res.setHeader('Access-Control-Allow-Credentials', true); // If neede
}

app.options('/**', cors(), function (req, res) {
  this.setResHeader(res);
    var redirectURL = 'http://example.com/api' + req.path;
    res.send(redirectURL);
});

app.post('/**', function (req, res, next) {
  this.setResHeader(res);
  var redirectURL = 'http://example.com/api' + req.path;

  request.post({ url: redirectURL, headers: req.headers, body: JSON.stringify(req.body) }, function (error, response, body) {
    if (response) {
      if (response.statusCode !== 200 && response.statusCode !== 201) {
        res.status(response.statusCode).send(response.body ? JSON.parse(response.body) : '');
      } else {
        res.send(response.body);
      }
    }
    else {
      next(createError(404));
    }


  });
});

app.get('/**', function (req, res, next) {
  this.setResHeader(res);
  var redirectURL = 'http://example.com/api' + req.originalUrl;
  request.get({ url: redirectURL, headers: req.headers }, function (error, response, body) {
      if (response) {
        if (response.statusCode !== 200 && response.statusCode !== 201) {
          res.status(response.statusCode).send(response.body ? JSON.parse(response.body) : '');
        } else {
          res.send(response.body);
        }
      }
      else {
        next(createError(404));
      }

    });
});

app.use(function (req, res, next) {
  next(createError(404));
});


// error handler
app.use(function (err, req, res, next) {
  // set locals, only providing error in development
  res.locals.message = err.message;
  res.locals.error = req.app.get('env') === 'development' ? err : {};

  // render the error page
  res.status(err.status || 500);
  res.render('error', {
    message: err.message,
    error: err
  });
});

module.exports = app;

```

同时在WWW.js文件中设置好监听端口为9001

```js
var port = normalizePort(process.env.PORT || '9001');
app.set('port', port);

```

然后在package.json文件中设置好命令行：

```js
  "scripts": {
    "node:9001": "node ./bin/www"
  },
```

前端在调用API的时候需要把API endpoint设置为http://localhost:9001


这样就解决了跨域问题， node会把监听到的API都转发到后端服务器，node和后端服务器就不存在跨域这种说法，所以可以直接调用。跨域只是针对浏览器同源策略。