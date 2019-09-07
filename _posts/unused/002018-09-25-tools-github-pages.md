---
title: 怎么搭建自己的GitHub Pages
tags: GIT
layout: post
---


一直想要有一个自己的技术Blog，想过用现在流行的静态网站生成器（Jekyll,Hexo......）先把自己的blog网页通过这些静态网页生成器写出来以后，然后再在服务器上做部署。
但是比较麻烦的是需要有自己的服务器做部署。

后来发现[GitHub Pages](https://pages.github.com/)满足我对个人blog的一切需求：
- 1) 只需要向github上提交自己的blog网页文本
- 2) github 会自动帮你编译和部署，并且有我自己的域名。
- 3) 不需要我操心服务器的问题
- 4) 不限流量，免费

听起来是不是很棒，这篇文章就是介绍怎么搭建属于自己的github pages。

#### 什么是GitHub Pages
大家肯定对GitHub不陌生，对GitHub最熟悉的就是我们可在在上面建立我们自己的代码仓库，然后把代码托管在上面。
GitHub Pages是用来展示你的代码或者项目的网页，可以理解为它是用来托管静态网页的，同时会编译部署这些网页，所有人通过特定域名可以访问到这些网页。

它是GitHub和静态网站生成器[Jekyll](https://jekyllrb.com)结合产物，你提交你的网页到GitHub，Jekyll立马帮你处理你的网页，然后再部署发布。


构建GitHub Pages前提是你有自己的GitHub账号和懂一些Jekyll的基本用法。
当然大前提还需要你懂一些网页开发。


**第一步，创建一个Github repository**


这个repository的名字是username.github.io, username指的是你GitHub的用户名，一定要确保用户名正确，否则后续都没办法正常工作。
![github pages account](https://limeii.github.io/assets/images/posts/git/git-pages-account.png){:height="100%" width="100%"}


**第二步，clone这个仓库到你本地**
```cmd
git clone https://github.com/LiMeii/LiMeii.github.io.git
```


**第三步，用VS Code打开这个项目 在根目里新增index.html/index.md文件**


在这个文件里，随便写点什么，比如 Hello Mei。
然后把这个文件上传到git。
```cmd
cd LiMeii.github.io
cd add index.html
cd commit -m "add index page"
cd push
```
然后在浏览器里访问 https://limeii.github.io 你可以看到：
![github pages index](https://limeii.github.io/assets/images/posts/git/git-pages-index.png){:height="100%" width="100%"}



好了到了这一步，你的github pages已经搭起来啦。
接下来就是按照你自己喜好，设置github pages布局和样式了，你也可以直接用Jekyll的默认样式，Jekyll有非常多网页主题可以选择[Jekyll主题](https://help.github.com/articles/about-jekyll-themes-on-github/)。
如果你用它默认的样式，那么需要在项目根目录下（跟index同级目录里）加一个Jekyll的配置文件_config.yml，在这个文件里加上你想要的主题,比如minima

```
theme: minima
```
那么默认的样式如下图：
![github pages minima](https://limeii.github.io/assets/images/posts/git/git-pages-minima.png){:height="100%" width="100%"}

当然你也可以写你自己的css，我的github pages主题样式就是按自己想法写的。


**第四步，使用Jekyll实现整个静态网站**


我现在GitHub Pages的结构如下：
![github pages](https://limeii.github.io/assets/images/posts/git/git-pages-jekyll.png){:height="100%" width="100%"}


_layouts 这个目录里用来放所有模板文件，比如 default.html, content表示文章内容。

![github pages](https://limeii.github.io/assets/images/posts/git/git-pages-layout.png){:height="100%" width="100%"}


_posts 这个目录里放着所有的blog，都是用markdown写的，一定要按照 年-月-日-文章名 这个规则来命名。
![github pages](https://limeii.github.io/assets/images/posts/git/git-pages-post.png){:height="100%" width="100%"}


_includes 比如页面之间有共用的模块，比如 header footer 放在这个目录下，在其他页面可以通过 include footer.html 直接引用。


**第五步，在首页里把所有的文章都列出来**


按一下的语法就把_posts下面所有的文title都列出来了，并且点击对应的title会跳转到对应文章。
![github pages](https://limeii.github.io/assets/images/posts/git/git-pages-postlists.png){:height="100%" width="100%"}

现在最简单的一个Blog就算搭好啦。