---
title: CSS布局absolute和relative的区别
tags: CSS
layout: post
---

写CSS过程中，经常要用到position进行页面布局，positioin有五个值：static，fixed，inherit，absolute，relative。前面三个还很好理解，后面两个在使用过程中经常会混淆，每次用到这几个值的时候，都要google查下这几个值的区别，这次直接把这五个的区别整理一下，巩固下这个知识点也方便日后自己查看。


先直接把这五个的区别列出来：


**static** 默认值，没有定位，元素出现在正常的文档流中，但是会忽略 top right bottom right z-index的声明。


**fixed** 生成绝对定位的元素，是相对于浏览器窗口进行定位，位置是通过 top right bottom right 进行定位。


**inherit** 从父元素继承 position 属性的值。


**relative** 元素脱离正常的文档流，但是在文档流中的位置依然存在。


**absolute** 元素是脱离正常文档流，同时在文档流中的位置不存在。

```
单独使用 relative absolute的时候，跟fixed区别不大。
只不过fixed是相对于浏览器进行定位，relatve absolute是相对于文档根节点进行定位

relative和其他属性结合使用的时候，虽然它脱离了文档流，但是在文档流中的位置依然存在，会相对于离它最近的父元素进行定位。
无论父元素是何种定位方式，找不到就直接相对文档根节点进行定位。

absolute和其他属性结合使用的时候，它脱离正常文档流，同时在文档流中的位置不存在，
会相对于离他最近的父元素（父元素的定位方式是relative/absolute）进行定位，找不到就直接相对文档根节点进行定位。
```

接下来看看具体案例（黑色背景是页面的body）：


**1. position:static 和 position:fixed**


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
![css position fixed static]( https://limeii.github.io/assets/images/posts/css/css-position-fixed-static.png){:height="60%" width="60%"}



**2. position:relative**


生成相对定位的元素，通过top right bottom right 进行定位, 相对它正常文档流中的位置进行定位。

```html
.div-relative {
    width: 550px;
    height: 400px;
    background-color: burlywood;
    position: relative;
    top: 5px;
    left: 100px;
}

<div class="div-relative " id="div-relative">
    <p> div-relative</p>
</div>
```

效果如下：
![css position relative]( https://limeii.github.io/assets/images/posts/css/css-position-relative.png){:height="60%" width="60%"}


**3. postion：static + relative，两个平行div，一个是static, 一个是relative**


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
.div-relative {
    width: 550px;
    height: 400px;
    background-color: burlywood;
    position: relative;
    top: 5px;
    left: 100px;
}

<div class="div-static " id="div-static">
    <p> div-static</p>
</div>

<div class="div-relative " id="div-relative">
    <p> div-relative</p>
</div>

```
div-static在文档流的第一个位置，div-relative在它后面，然后通过 top: 5px;left: 100px; 相对于离它最近的div-static的bottom和left进行了定位。


效果如下：
![css position relative]( https://limeii.github.io/assets/images/posts/css/css-position-static-relative1.png){:height="60%" width="60%"}



**4. postion：static + relative，两个Div，父元素是static, 子元素是relative**


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
.div-relative {
    width: 550px;
    height: 400px;
    background-color: burlywood;
    position: relative;
    top: 5px;
    left: 100px;
}

<div class="div-static " id="div-static">
    <p> div-static</p>
    <div class="div-relative " id="div-relative">
        <p> div-relative</p>
    </div>
</div>
```

div-static在文档流的第一个位置，div-relative在它后面，然后通过 top: 5px;left: 100px; 相对于父元素div-static的top和left进行了定位。


效果如下：
![css position relative]( https://limeii.github.io/assets/images/posts/css/css-position-static-relative2.png){:height="60%" width="60%"}


**5. position: absolute**


脱离文档流，生成相对定位的元素，同时在文档流中的位置不存在，通过top right bottom right 进行定位。

```html
.div-absolute {
    width: 450px;
    height: 300px;
    background-color: cornflowerblue;
    position: absolute;
    top: 5px;
    left: 100px;
}

<div class="div-absolute " id="div-absolute">
    <p> div-absolute</p>
</div>
```

效果如下：
![css position absolute]( https://limeii.github.io/assets/images/posts/css/css-position-absolute.png){:height="60%" width="60%"}


**6. postion：static + absolute，两个平行div，一个是static, 一个是absolute**


```html
.div-static {
    width: 400px;
    height: 200px;
    background-color: gray;
    position: static;
    top: 100px;
    left: 100px;
}
.div-absolute {
    width: 450px;
    height: 300px;
    background-color: cornflowerblue;
    position: absolute;
    top: 5px;
    left: 100px;
}


<div class="div-static " id="div-static">
    <p> div-static</p>
</div>

<div class="div-absolute " id="div-absolute">
    <p> div-absolute</p>
</div>
```

效果如下：可以看出，在这种情况下div-absolute脱离文档流，没有父元素，相对于文档根节点进行定位，并且有一部分会悬浮覆盖在div-static 
![css position absolute]( https://limeii.github.io/assets/images/posts/css/css-position-static-absolute1.png){:height="60%" width="60%"}

**7. postion：static + absolute，两个div，父元素是static, 子元素是absolute**

```html
.div-static {
    width: 400px;
    height: 200px;
    background-color: gray;
    position: static;
    top: 100px;
    left: 100px;
}
.div-absolute {
    width: 450px;
    height: 300px;
    background-color: cornflowerblue;
    position: absolute;
    top: 5px;
    left: 100px;
}


<div class="div-static " id="div-static">
    <p> div-static</p>
    <div class="div-absolute " id="div-absolute">
        <p> div-absolute</p>
    </div>
</div>
```
效果如下：效果跟上面那种case一样，div-absolute脱离文档流，虽然有父元素div-static，但是父元素的定位方式是static，在这种情况下一直往上寻找定位方式为absolute/releative的父元素，没有找到就根据文档根节点进行定位。
![css position absolute]( https://limeii.github.io/assets/images/posts/css/css-position-static-absolute1.png){:height="60%" width="60%"}

**8. postion：relative + absolute，两个平行div，一个是relative, 一个是absolute**


```html
.div-relative {
    width: 550px;
    height: 400px;
    background-color: burlywood;
    position: relative;
    top: 20px;
    left: 10px;
}
.div-absolute {
    width: 450px;
    height: 300px;
    background-color: cornflowerblue;
    position: absolute;
    top: 5px;
    left: 100px;
}


<div class="div-relative " id="div-relative">
    <p> div-relative</p>
</div>

<div class="div-absolute " id="div-absolute">
    <p> div-absolute</p>
</div>
```
效果如下：div-relative div-absolute都是根据文档根节点进行定位。
![css position absolute]( https://limeii.github.io/assets/images/posts/css/css-position-relative-absolute1.png){:height="60%" width="60%"}


**9. postion：relative + absolute，两个div，父元素是relative, 子元素是absolute**

```html
.div-relative {
    width: 550px;
    height: 400px;
    background-color: burlywood;
    position: relative;
    top: 20px;
    left: 10px;
}
.div-absolute {
    width: 600px;
    height: 200px;
    background-color: cornflowerblue;
    position: absolute;
    top: 10px;
    left: 100px;
}


<div class="div-relative " id="div-relative">
    <p> div-relative</p>
    <div class="div-absolute " id="div-absolute">
        <p> div-absolute</p>
    </div>
</div>
```
效果如下：div-relative 相对文档根节点定位，div-absolute相对于div-relative定位。
![css position absolute]( https://limeii.github.io/assets/images/posts/css/css-position-relative-absolute2.png){:height="60%" width="60%"}

**10. postion：relative + absolute，两个div，父元素是absolute, 子元素是relative**

```html
.div-relative {
    width: 550px;
    height: 400px;
    background-color: burlywood;
    position: relative;
    top: 20px;
    left: 20px;
}
.div-absolute {
    width: 600px;
    height: 200px;
    background-color: cornflowerblue;
    position: absolute;
    top: 10px;
    left: 100px;
}

<div class="div-absolute " id="div-absolute">
    <p> div-absolute</p>
    <div class="div-relative " id="div-relative">
        <p> div-relative</p>
    </div>
</div>
```
效果如下：div-absolute相对根节点定位，div-relative相对于div-absolute定位
![css position absolute]( https://limeii.github.io/assets/images/posts/css/css-position-relative-absolute3.png){:height="60%" width="60%"}


**11. postion：static + relative + absolute，这三个div平行**

```html

.div-static {
    width: 400px;
    height: 200px;
    background-color: gray;
    position: static;
    top: 100px;
    left: 100px;
}
.div-relative {
    width: 550px;
    height: 400px;
    background-color: burlywood;
    position: relative;
    top: 20px;
    left: 20px;
}
.div-absolute {
    width: 600px;
    height: 200px;
    background-color: cornflowerblue;
    position: absolute;
    top: 10px;
    left: 100px;
}

<div class="div-static " id="div-static">
    <p> div-static</p>
</div>
<div class="div-relative " id="div-relative">
    <p> div-relative</p>
</div>

<div class="div-absolute " id="div-absolute">
    <p> div-absolute</p>
</div>
```
效果如下：div-static出现在文档流本身的位置，top left属性对div-static没有任何影响；div-relative 相对于div-static通过top left进行了定位；div-absolute还是相对于文档根节点进行了定位。为什么这里 div-relative没有相对于文档根节点定位，是因为div-relative还存在于文档流中，它在div-static后面，需要根据static进行定位。而div-absolute完全脱离文档流并且在文档流中没有实际位置了，所以依旧根据根节点进行定位。
![css position absolute]( https://limeii.github.io/assets/images/posts/css/css-position-static-relative-absolute1.png){:height="60%" width="60%"}

我们调整三个Div位置如下：

```html
<div class="div-static" id="div-static">
    <p> div-static</p>
    <div class="div-relative " id="div-relative">
        <p> div-relative</p>
    </div>

    <div class="div-absolute " id="div-absolute">
        <p> div-absolute</p>
    </div>
</div>

```
div-relative 和 div-absolute为平行元素，并且都是div-static的子元素。效果如下:


div-relative 会相对于div-static进行定位，而div-absolute还是相对于根节点进行定位。
![css position absolute]( https://limeii.github.io/assets/images/posts/css/css-position-static-relative-absolute2.png){:height="60%" width="60%"}


我们再次调整三个Div位置如下：

```html
<div class="div-static" id="div-static">
    <p> div-static</p>
        <div class="div-relative " id="div-relative">
             <p> div-relative</p>
            <div class="div-absolute " id="div-absolute">
                 <p> div-absolute</p>
            </div>
        </div>
</div>

```
div-absolute 是 div-relative的子元素，div-relative是div-static的子元素。


div-relative 会相对于div-static进行定位，而div-absolute会相对于它的父元素div-relative进行定位。
![css position absolute]( https://limeii.github.io/assets/images/posts/css/css-position-static-relative-absolute3.png){:height="60%" width="60%"}

**12. postion：relative + relative + absolute，这三个Div层层嵌套**

```html

.div-relative1 {
    width: 400px;
    height: 100px;
    background-color: gray;
    position: relative;
    top: 100px;
    left: 10px;
}
.div-relative {
    width: 550px;
    height: 400px;
    background-color: burlywood;
    position: relative;
    top: 50px;
    left: 20px;
}
.div-absolute {
    width: 600px;
    height: 200px;
    background-color: cornflowerblue;
    position: absolute;
    top: 10px;
    left: 100px;
}
<div class="div-relative1 " id="div-relative1">
    <p> div-relative1</p>
    <div class="div-relative " id="div-relative">
        <p> div-relative</p>
        <div class="div-absolute " id="div-absolute">
            <p> div-absolute</p>
        </div>
    </div>
</div>

```

效果如下：div-relative相对div-relative1进行定位，div-absolute 相对于div-relative进行了定位，它们各自都找到自己的父元素从而进行定位。

![css position absolute]( https://limeii.github.io/assets/images/posts/css/css-position-relative-relative-absolute.png){:height="60%" width="60%"}
