---
title: absolute relative difference
layout: post
---

# CSS布局 absolute 和 relative的区别
<div class="title-meta">
    <span><img class="title-category-img" style="margin-top: 0.2em;" src="../../../assets/images/categories/css3.svg" alt="issues"></span>
    <span><a class="github-link" href="/2018/09/19/CSS.html">CSS</a></span>
    <span class="title-bullet">•</span>
    <span>Jan 30, 2019</span>
    <span class="title-bullet">•</span>
    <span>by <a class="github-link" href="http://github.com/limeii" title="http://github.com/limeii">Mei</a></span>
</div>

写CSS过程中，经常要用到position进行页面布局，positioin有五个值：static，fixed，absolute，relative，inherit。对于 static fixed 和 inherit都很好理解，经常会混淆 absolute 和 relative的用法，absolute 和 relative最主要的区别是：relative的元素脱离正常的文档流，但是在文档流中的位置依然存在；absolute的元素是脱离正常文档流，同时在文档流中的位置不存在。


单独使用的时候，relative 和 absolute 区别不大，这几个结合起来用的时候，经常会出问题。 接下来就来看看这几个属性单独或结合使用的效果和区别。


## position:static 和 position:fixed 

先来看两个最简单的：


**static：** 默认值，没有定位，元素出现在正常的流中，但是会忽略 top right bottom right z-index的声明。


**fixed：** 生成绝对定位的元素，是相对于浏览器窗口进行定位，位置是通过 top right bottom right 进行定位


代码如下：

```html
.div-static {
    width: 400px;
    height: 200px;
    background-color: gray;
    position: static;
    top: 100px;
    left: 100px;
}

.div-fixed {
    width: 600px;
    height: 600px;
    background-color: cadetblue;
    position: fixed;
    top: 100px;
    left: 100px;
}
<div class="div-static " id="div-static">
    <p> div-static</p>
</div>
<div class="div-fixed " id="div-fixed">
    <p> div-fixed</p>
</div>
```

div-static 出现在正常文档流本来的位置，不管怎么更改 top right bottom right 属性的值，它的位置都不会发生任何改变。div-fixed 是相对浏览器的位置进行定位，更改 top right bottom right 属性的值，它的位置会相应改变。
![css position static]( https://limeii.github.io/assets/images/posts/css/css-position-fixed-static.png.png){:height="60%" width="60%"}



## position:relative
生成相对定位的元素，通过top right bottom right