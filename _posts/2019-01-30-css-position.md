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

写CSS过程中，经常要用到position进行页面布局，positioin有五个值：static，fixed，absolute，relative，inherit。对于 static fixed 和 inherit都很好理解，经常会混淆 absolute 和 relative的用法，这篇文章结合代码应用来解释这五个属性的用法和区别。


### position:static

默认值，没有定位，元素出现在正常的流中，但是会忽略 top right bottom right z-index的声明。


代码如下：

```html
.div-static {
    width: 400px; 
    height: 200px; 
    background-color: gray;
    position: static; 
    top:100px;
    left:100px;
}

<div class="div-static " id="div-static">
    <p> div-static</p>
</div>
```

效果如下：不管怎么更改 top 或者 left 的值，灰色的div位置都不会有变化。
![css position static]( https://limeii.github.io/assets/images/posts/css/css-position-static.png){:height="50%" width="50%"}

### position:fixed
生成绝对定位的元素，是相对于浏览器窗口进行定位，位置是通过 top right bottom right 进行定位


代码如下：

```html
.div-fixed {
    width: 400px;
    height: 200px;
    background-color: cadetblue;
    position: fixed;
    top: 100px;
    left: 400px;
}

<div class="div-fixed " id="div-fixed">
    <p> div-fixed</p>
</div>

```
效果如下: 更改 top right bottom right 属性的值，div-fixed的位置都会相对浏览器位置做调整
![css position fixed]( https://limeii.github.io/assets/images/posts/css/css-position-fixed.png){:height="50%" width="50%"}