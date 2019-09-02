---
title: 在Win10里安装Ruby和Jekyll遇到的问题
tags: 问题
layout: post
---

## 问题描述

之前在搭自己的github.io的时候，需要在Win10里把整个Jekyll的环境搭起来，按照官网的步骤（如图所示）：

![rubyJekyll-win10-setup.png]( https://limeii.github.io/assets/images/posts/issues/rubyJekyll-win10-setup.png){:height="100%" width="100%"}


ruby装成功以后，按照官网步骤跑命令行：gem install jekyll bundler， 会报如下错误：

```
ERROR:  SSL verification error at depth 1: unable to get local issuer certificate (20)
ERROR:  You must add /DC=com/DC=nextestate/CN=GDCSUBCA01 to your local trusted store
ERROR:  SSL verification error at depth 1: unable to get local issuer certificate (20)
ERROR:  You must add /DC=com/DC=nextestate/CN=GDCSUBCA01 to your local trusted store
ERROR:  SSL verification error at depth 1: unable to get local issuer certificate (20)
ERROR:  You must add /DC=com/DC=nextestate/CN=GDCSUBCA01 to your local trusted store
ERROR:  Could not find a valid gem 'bundler' (>= 0), here is why:
          Unable to download data from https://rubygems.org/ - SSL_connect returned=1 errno=0 state=error: certificate verify failed (unable to get local issuer certificate) (https://rubygems.org/specs.4.8.gz)
ERROR:  SSL verification error at depth 1: unable to get local issuer certificate (20)
ERROR:  You must add /DC=com/DC=nextestate/CN=GDCSUBCA01 to your local trusted store
ERROR:  SSL verification error at depth 1: unable to get local issuer certificate (20)
ERROR:  You must add /DC=com/DC=nextestate/CN=GDCSUBCA01 to your local trusted store

```

## 问题分析

一看错误信息，很明显的SSL certificate的问题，那直接关掉 SSL verfication好了。


## 解决方案

在Win10里把Ruby SSL verfication关掉的步骤如下：


1. 在当前用户根目录下找到文件：.gemrc


2. 编辑.gemrc文件，在里面加上:ssl_verify_mode: 0，或者把https://rubygems.org 改为http://rubygems.org


### 需要注意的是：

1. 怎么找到当前用户根目录：在cmd中跑命令行 echo %USERPROFILE%

2. 在根目录里没有.gemrc这个文件，而且在Win10里根本没办自己创建.gemrc文件，在[https://gist.github.com/LiMeii/.gemrc](https://gist.github.com/LiMeii/3d43bf4159b1ef870abbb44000c6003f) 直接把文件下载下来放到当前跟目录下就可以了。

3. 当环境都搭好了，需要把jekyll跑起来，在Windows下直接跑命令行：bundle exec jekyll serve