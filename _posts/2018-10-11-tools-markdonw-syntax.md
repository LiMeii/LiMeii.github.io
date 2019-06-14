---
title: markdown常用语法
tags: 工具
layout: post
---


这篇文章是介绍markdown 常用的一些语法。


**所有的字符都应该是英文字符，中文字符无效**

# 1 标题语法

```html
#【空格】标题名称
# 表示 <h1>
## 表示 <h2>
### 表示 <h3>
#### 表示 <h4>
##### 表示 <h5>
###### 表示 <h6>
```
效果如下：
# 一级标题
## 二级标题 
### 三级标题 
#### 四级标题
##### 五级标题
###### 六级标题

# 2 一级标题 二级标题
利用任一数量的等号（=）减号（-）
```html
一级标题
===
二级标题
--
```
效果如下：

一级标题
=
二级标题
-

# 3 段落和换行
直接在markdonw中输入文本就可以，段落之间用两个回车。
如果段落里面想换行的话，可以在上一行结尾插入两个以上的空格后再回车

```html
第一段落
【回车】
【回车】
第二段落

这是第一段落，如果想换行输入【空格】【空格】
【回车】
换行成功
```
效果如下：


第一段落


第二段落


这是第一段落，如果想换行输入  

换行成功

# 4 强调
```
*倾斜*
**粗体**
~~删除线~~
> 引用
```
效果如下：


*倾斜*
**粗体**
~~删除线~~
> 引用

# 5 列表
```
无序列表
* 项目 1
* 项目 2
 * 子项目 2a
 * 子项目 2b
```
效果如下：


* 项目 1
* 项目 2
  * 子项目 2a
  * 子项目 2b

```
有序列表
1. 项目 1
2. 项目 2
3. 项目 3
```
 效果如下：


1. 项目 1
2. 项目 2
3. 项目 3


```
任务列表
- [x] 任务列表1
- [ ] 任务列表2
- [x] 任务列表3
```
效果如下：


- [x] 任务列表1
- [ ] 任务列表2
- [x] 任务列表3

# 6 链接
```
[github链接](https://github.com/limeii)
```
效果如下：


[github链接](https://github.com/limeii)

# 7 图片
```
![webpack-css-extract-chunkhash.png](https://limeii.github.io/assets/images/posts/webpack/webpack-css-extract-chunkhash.png){:height="100%" width="100%"}

{:height="100%" width="100%"} 表示图片显示比例

如果想要改表图片样式，比如居中，那么需要自己写<img>的css样式

```
效果如下：


![webpack-css-extract-chunkhash.png](https://limeii.github.io/assets/images/posts/webpack/webpack-css-extract-chunkhash.png){:height="100%" width="100%"}

# 8 代码块
```
多行代码使用 3 个反引号来标记(反引号一般位于键盘左上角，要用英文) ，在第一个 ｀｀｀ 后面可以跟语言类型，没有语言类型可以省略不写.
注意代码块的样式，也需要自己写css

｀｀｀javascript
var a = 1;
console.log(a);
｀｀｀
```
效果如下：


```javascript
var a = 1;
console.log(a);
```

# 9 表格

```
First Header | Second Header
------------ | -------------
Content cell 1 | Content cell 2
Content column 1 | Content column 2
```

效果如下：


First Header | Second Header
------------ | -------------
Content cell 1 | Content cell 2
Content column 1 | Content column 2