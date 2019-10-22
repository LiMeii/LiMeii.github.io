---
title: 浏览器数据存储方式
tags: web
layout: post
---

前端浏览器数据存储方式有：Cookies、SessionStorage、LocalStorage、IndexedDB，这篇文章主要是比较这几种存储方式的区别，需要注意的是这些存储方式都受同源策略的约束，跨域是不能访问。

## Cookies
Cookies的出现并不是为了在浏览器里保存数据，而是为了保存HTTP状态的，因为HTTP协议是没有状态的，也就是服务器不知道用户上一次做了什么，有了Cookies，服务器可以设置或者读取Cookies中的信息，每次HTTP请求都会带上Cookies，从而可以维护用户跟服务器会话的状态。


Cookies有以下两种设置方式：
- 1：用JS的```document.cookie```读取和设置
- 2：在服务器端，通过HTTP协议规定的```set-coookie```在浏览器里种下cookie，之后浏览器里的每个HTTP请求都会自动带上这些Cookies信息

第二种方式是最常见的方式，一般用来保存用户密码，下次自动登录，它的流程为：
![web-storage](/assets/images/posts/web/web-storage01.png){:height="100%" width="100%"}

以github为例，我们可以在```devtools-Application-Cookies```里查看当前的Cookies信息:

![web-storage](/assets/images/posts/web/web-storage02.png){:height="100%" width="100%"}

从上图中可以看到，Cookies里有以下属性：
- Name：Cookies里键值对的名字
- Value：Cookies里键值对的值
- Domain：Cookies是同源策略，这个是当前Cookies生效的域名
- Path：表示Cookies影响的路径，匹配到这个路径才发送这个Cookies
- Expires/Max-Age：Expires告诉浏览器Cookies什么时候过期，Max-Age是相对过期时间，不设置这两个时间，默认用户关闭浏览器就被清除
- Size：表示当前保存值的大小
- HttpOnly：当这个值为True的时候，表示浏览器不能通过```document.cookie```更改Cookies的值，可以避免被xss攻击更改Cookies的值
- Secure：当这个值为True的时候，Cookies在HTTP中无效，在HTTPS中才有效
- SameSite：规定浏览器不能在跨域请求中携带Cookies，减少CSRF攻击

Cookies有以下缺陷：
- Cookies的大小限制在4kb左右，对于复杂的存储需求肯定不够用，当Cookies超过4kb，就会被直接截断。
- 过多的Cookies会有性能浪费，同一个域名下面的所有请求，都会带上Cookies，随着请求的增多，肯定会有带宽性能的浪费。

由于Cookies的局限，在H5中新增了本地存储的方案：WebStorage，它分为两类：SessionStorage和LocalStorage。

## LocalStorage

LocalStorage有以下特点：
- 保存的数据，会一直存在内存里，没有过期时间，除非主动清除
- 可以用在Tab之间数据通信
- 大小为5M左右
- 只用在客户端，不和服务器端通信
- 接口封装较好
- 遵循同源策略

关于LocalStorage的接口用法可以参考MDN文档：【[Window.localStorage](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/localStorage)】

以github为例，我们可以在```devtools-Application-LocalStorage```里查看当前的LocalStorage信息:
![web-storage](/assets/images/posts/web/web-storage03.png){:height="100%" width="100%"}


## SessionStorage
相对于LocalStorage，SessionStorage保存的数据只用于浏览器的一次会话，当前会话（通常是该窗口关闭），数据就会被清空。

SessionStorage有以下特点：
- 会话级别的浏览器存储，数据不能在不同的Tab之间共享
- 大小为5M左右
- 只用在客户端，不和服务器端通信
- 遵循同源策略

关于SessionStorage的接口用法可以参考MDN文档：【[Window.sessionStorage](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/sessionStorage)】


LocalStorage和SessionStorage都是使用键值对的形式进行存储，它只能存储字符串，要存储对象，需要进行序列化和解析，而且大小都在5M左右。用来存一些非敏感数据完全够用。但是如果遇到大规模而且结构复杂的数据时，LocalStorage和SessionStorage没办法满足这种需求，这时候就可以用IndexedDB。

## IndexedDB
IndexedDB是一种低级API，用于客户端存储大量结构化数据（包括文件和blobs）。该API使用索引实现高性能数据搜索，它是一个运行在浏览器上的非关系型数据库，它的大小是没有存储上限的。

IndexedDB有以下特点：
- 键值对存储
- IndexedDB操作是异步的，不会造成浏览器卡死
- 支持事务
- 遵循同源策略
- 存储空间大，一般不少于250MB，没有上限
- 支持二进制存储

关于IndexedDB的接口用法可以参考MDN文档：【[IndexedDB](https://developer.mozilla.org/zh-CN/docs/Web/API/IndexedDB_API)】


## 总结

 特性| Cookies | LocalStorage | SessionStorage | IndexedDB
:-------: | :-------: | :-------:  | :-------:  | :-------: 
数据生命周期 | 一般有服务器生成，可以设置过期时间|除非被清除，否则一直存在|页面关掉就清除|除非被清除，否则一直存在
数据存储大小|4k|5M|5M|无限
与服务器端通信|每次都会带在header中，对请求性能有影响|不参与|不参与|不参与