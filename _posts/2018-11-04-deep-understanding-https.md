---
title: 深入理解HTTPS
tags: web
layout: post
---

## 什么是HTTPS

我们知道HTTP（HyperText Transfer Protocol）是超文本传输协议，是网络通信的基础。HTTP通信是分层的，从上到下有：应用层、传输层、网络层、链路层。

HTTPS（HyperText Transfer Protocol Secure）是超文本传输安全协议，是安全通信通道，简单来说是HTTP的安全版本，是使用TLS/SSL加密的HTTP协议。
![web-http](/assets/images/posts/web/web-https01.png){:height="100%" width="100%"}

我们在浏览器里访问某个网站的时候，如果看到类似下面这种小锁头，表示用的就是HTTPS协议：
![web-http](/assets/images/posts/web/web-https02.png){:height="100%" width="100%"}

## 为什么要有HTTPS

HTTP是采用明文传输的，存在信息窃听、信息篡改和信息劫持的风险。想象一下你要访问GitHub网站，在中间的某个DNS被黑客拦截到你的HTTP请求，黑客返回一个跟GitHub一模一样的页面给你，由于没有身份认证，你以为当前的页面就是GitHub官网，输入你自己的用户名密码登入，从而黑客就是拿到这些信息，进行一些非法操作。如果是一些银行转账之类的网站，后果将更加严重。

为了解决HTTP的安全问题，HTTPS在HTTP上建立一层SSL/TLS加密，对传输的数据进行加密，会有以下优势：
- 信息加密：内容进行对称加密，每一次连接都生成一个唯一的加密密钥。
- 数据完整性：数据传输会经过完整性校验。
- 身份认证：第三方无法伪造客户端身份。

SSL/TLS全称安全传输协议Transport Layer Security，是介于TCP和HTTP之间的一层安全协议，不影响原来的HTTP和TCP协议。
![web-http](/assets/images/posts/web/web-https03.png){:height="100%" width="100%"}

<blockquote>
<p>
SSL是TLS的前身，SSL2和SSL3分别在2011和2015年被废弃了，TLS可以说是SSL的新版本。
</p>
</blockquote>

## SSL/TLS
前面讲到HTTPS主要是依赖SSL/TLS协议对数据进行加密，我们来看下SSL/TLS的工作原理。


SSL/TLS的功能实现主要依赖于三类基本算法：散列函数、Hash、对称加密和非对称加密，其利用非对称加密实实现身份认证和密钥协商，对称加密算法采用协商的密码对数据加密，基于散列函数验证信息的完整性。
![web-http](/assets/images/posts/web/web-https04.png){:height="100%" width="100%"}

### 散列函数Hash
常见的有MD5、SHA1、SHA256，该类函数特点是函数单向不可逆、对输入非常敏感、输出长度固定，针对数据的任何修改都会改变散列函数的结果，用于防止信息篡改并验证数据的完整性。


在信息传输过程中，散列函数不能单独实现信息防篡改，因为明文传输，中间人可以修改信息之后重新计算信息摘要，因此需要对传输的信息以及信息摘要进行加密。


通过这种加密方式生成数字签名，从而保证数据的完整性。


数字签名如何生成：
![web-http](/assets/images/posts/web/web-https06.png){:height="100%" width="100%"}

将一段文本用Hash函数生成消息摘要，然后用发送者的私钥加密生成数字签名，与原文一起传送给接受者。


接受者如何验证数字签名：
![web-http](/assets/images/posts/web/web-https07.png){:height="100%" width="100%"}


接收者只有用发送者的公钥才能解密被加密的摘要信息，然后用HASH函数对收到的原文产生一个摘要信息，与上一步得到的摘要信息对比。如果相同，则说明收到的信息是完整的，在传输过程中没有被修改，否则说明信息被修改过，因此数字签名能够验证信息的完整性。

### 对称加密
常见的有AES-CBC、DES、3DES、AES-GCM等，相同的密码可以用于信息的加密和解密，掌握密钥才能获取信息，能够防止信息窃听，通信方式是1对1。


对称加密的优势是信息传输1对1，需要共享项目的密码，密码的安全是保证信息安全的基础，服务器和N个客户端通信，需要维持N个密码记录，且确实修改密码的机制。

### 非对称加密
即常见的RSA算法，还包括ECC、DH等算法，算法特点是，密码成对出现，一般称为公钥（公开）和私钥（保密），公钥加密的信息只能私钥解开，私钥加密的信息只能公钥解开。因此掌握公钥的不同客户端之间不能互相解密信息，只能和掌握私钥的服务器进行加密通信，服务器可以实现1对多得通信，客户端也可以用来验证掌握私钥的服务器身份。


非对称加密的特别是信息传输1对多，服务器只需要维持一个私钥就能够和多个客户端进行加密通信，但服务器发出的信息能够被所有客户端解密，由于该算法的计算复杂，加密速度慢。

### SSL/TLS工作原理
结合三类算法的特点，TLS的基本工作方式是：
- 客户端使用非对称加密与服务器进行通信拿到公钥信息（数字证书），实现身份验证。
- 由于服务器端私钥加密复杂，耗时较长，所以在客户端身份验证成功以后，会协商对称加密使用的密钥。
- 然后对称加密算法采用协商密钥对信息以及信息摘要进行加密通信，不同的阶段之间采用的对称密码不同（每次连接的对称密钥都不一样），从而可以保证只能通信双方获取。

## 证书、CA和HTTPS工作流程
客户端第一次发请求到服务器端的时候，需要先跟服务器端建立安全通道（通过SSL/TLS），之后才能和服务器进行数据通信；而SSL/TLS结合散列函数、Hash、对称加密和非对称加密实现身份认证、密钥协商、之后再根据协商的密钥进行数据加密通信。


客户端在第一次建立安全通道的时候，需要结合非对称加密实现身份认证，那它怎么确保服务器发过来的公钥是真正有效的呢？为了解决这个问题，就由了CA(Certificate Authorities)，CA是第三方的认证机构。整个认证流程如下：

- 服务器的运营人员向第三方机构CA提交公钥、组织信息、个人信息(域名)等信息并申请认证;
- CA通过线上、线下等多种手段验证申请者提供信息的真实性，如组织是否存在、企业是否合法，是否拥有域名的所有权等;
- 如信息审核通过，CA会向申请者签发认证文件-证书。证书包含以下信息：申请者公钥、申请者的组织信息和个人信息、签发机构CA的信息、有效时间、证书序列号等信息的明文，同时包含一个签名。 其中签名的产生算法：首先，使用散列函数计算公开的明文信息的信息摘要，然后，采用CA的私钥对信息摘要进行加密，密文即签名;
- 客户端Client向服务器Server发出请求时，Server 返回证书文件;
- 客户端Client读取证书中的相关的明文信息，采用相同的散列函数计算得到信息摘要，然后，利用对应CA的公钥解密签名数据，对比证书的信息摘要，如果一致，则可以确认证书的合法性，即服务器的公开密钥是值得信赖的。
- 客户端还会验证证书相关的域名信息、有效时间等信息; 客户端会内置信任CA的证书信息(包含公钥)，如果CA不被信任，则找不到对应CA的证书，证书也会被判定非法。

### 整个HTTPS的工作流程

![web-http](/assets/images/posts/web/web-https05.png){:height="100%" width="100%"}

- Client发起一个HTTPS（比如https://github.com/limeii）的请求，默认需要连接Server的443（默认）端口。
- Server把事先配置好的公钥证书（public key certificate）返回给客户端。
- Client验证公钥证书：比如是否在有效期内，证书的用途是不是匹配Client请求的站点，是不是在CRL吊销的列表里，它的上一级证书是否有效，这是一个递归的过程，指导验证到根证书（操作系统内置的Root证书或者Client内置的Root证书）。如果验证通过则继续，不通过则显示警告信息。
- Client使用伪随机数生成器生成加密使用的对称密码，然后用证书的公钥加密这个对称密码，发给Server。
- Server使用自己的私钥解密这个消息，得到对称密码，到这一步，Client和Server双方都持有了相同的对称密码。
- Server使用对称密码加密“明文内容A”，发送给Client。
- Client使用对称密钥解密响应的密文，得到“明文内容A”。
- Client再次发起HTTPS的请求，使用对称密码加密请求的“明文内容B”，然后Server使用对称密钥解密密文，得到“明文内容B”

## HTTP和HTTPS的区别
- HTTPS是加密传输协议，HTTP是明文传输协议
- HTTPS需要用到SSL/TLS证书，而HTTP不用
- HTTPS比HTTP更安全，对搜索引擎更友好，利于SEO
- HTTS标准端口443，HTTP标准端口80
- HTTPS基于传输层，HTTP基于应用层
- HTTPS在浏览器显示绿色安全锁，HTTP没有


## TCP三次握手
HTTP是基于传输层的，在建立连接传输数据之前有三次握手:
![web-http](/assets/images/posts/web/http-tcp-three-handshakes.png){:height="80%" width="80%"}

为什么要有三次握手？
- 客户端主动提出要建立连接，发出客户端seq：```seq=client_isn```。
- 服务器收到后返回```ack=client_isn+1```和服务器端seq：```seq=server_isn```。
- 客户端收到后返回```ack=server_isn+1```表示收到了。

只有通过这三次通信，才能确保客户端和服务器都能正常接收对方的消息。如果没有第三次握手，只能说明服务器可以正常收到客户端的消息，但是不能确保客户端是否能正常收到服务器端消息，所以需要第三次握手。

## TCP四次挥手
在断开连接之前，需要有四次挥手：
![web-http](/assets/images/posts/web/http-tcp-four-handshakes.png){:height="80%" width="80%"}

为什么要有四次挥手呢？

- 客户端第一次发送消息给服务器端告诉它需要断开连接。
- 服务器端收到这个消息后返回一个消息告诉客户端：知道了，为了确保服务器端收到了之前所有的http请求，服务端需要等一等再断开连接。
- 服务器确认所有的http请求都收到了，主动发消息给客户端：我这边所有的请求都处理完了，我也可以断开连接了。
- 客户端收到这个请求后，返回消息告诉服务器：我知道，断开连接吧。

断开连接，需要客户端和服务器都主动断开连接，所以至少需要四次挥手才能确保连接断开。

## SSL四次握手
前面提到过，HTTPS是HTTP的安全版本，在HTTP的基础上加了一层SSL/TLS安全协议。


HTTPS在建立连接的时候，其实有七次握手：TCP的三次握手和SSL的四次握手，前面已经解释了TCP的三次握手，我们来看下SSL的四次握手（握手阶段的通信都是明文的）：

![web-http](/assets/images/posts/web/https-ssl-four-handshakes.png){:height="70%" width="70%"}

### 客户端先向服务器发出加密通信的请求（ClientHello请求）
在这一步客户端会提供如下信息：
- 支持的协议版本，比如TLS1.0版本
- 一个客户端生成的随机数，稍后用于生成”对话密钥“
- 支持的加密算法，比如RSA公钥加密
- 支持的压缩方法

### 服务器回应（SeverHello）
服务响应内容包括：
- 确认使用的加密通信协议版本，比如TLS1.0，如果客户端和服务器支持的版本不一样，服务器关闭通信
- 一个服务器生成的随机数，稍后用户生成”对话密钥“
- 确认使用的加密算法，比如RSA公钥加密
- 服务器证书

### 客户端响应
客户端收到服务器端响应以后，首先验证服务器证书，如果服务器证书不是可信机构颁布或者证书域名与实际域名不一致或者证书过期，就会向访问者显示一个警告，由其选择是否好要继续通信。如果证书没有问题，客户端就会从证书中取出服务器公钥，然后会发送如下信息：
- 一个随机数，改随机数用与服务器公钥加密，防止窃听
- 码改变通知，表示随后的信息都将用双方商定的加密方法和密钥发送
- 客户端握手结束通知，表示客户端的握手阶段已经结束了

### 服务器的最后响应
服务器收到消息后，会发送如下信息：
- 编码改变通知，表示随后的信息都将用双方商定的加密方法和密钥发送。
- 服务器握手结束通知，表示服务器的握手阶段已经结束。这一项同时也是前面发送的所有内容的hash值，用来供客户端校验。
