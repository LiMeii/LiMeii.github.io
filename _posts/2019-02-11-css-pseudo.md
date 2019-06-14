---
title: 伪类和伪元素
tags: CSS
layout: post
---

自己在写css过程中，经常用到 :before / :after / :hover 等伪类和伪元素，但是除了这几个常见的，具体还有哪些伪类和伪元素，以及伪类和伪元素区别都不是很清楚。


今天就在这里总结伪类和伪元素的区别和具体用法。


**为什么要有伪类和伪元素**


我们先来看看在[CSS 3 selector recommendation](https://www.w3.org/TR/2011/REC-css3-selectors-20110929/#pseudo-classes)对伪类和伪元素的定义：

![css pseudo-classes]( https://limeii.github.io/assets/images/posts/css/css-pseudo-classes.png){:height="100%" width="100%"}

![css pseudo-elements]( https://limeii.github.io/assets/images/posts/css/css-pseudo-elements.png){:height="100%" width="100%"}

总结来说：伪类和伪元素是用来处理文档树之外的部分。


正常每个 div span p 等元素都是文档树里的节点，物理实际存在文档树里。但是在某些时候，比如想要对第一行或者首字母做处理，或者是用户点击某个link后想要有某种效果，那么只靠文档树是没有办法做到的，伪类和伪元素就是用来修饰不在文档树中的部分。


那么伪类和伪元素之间又有什么区别呢？


**伪类是用来修饰已经存在的元素**，为其添加对应的样式，这个状态是根据用户行为而动态变化的。比如，用户悬停指定某个元素，我们可以用 :hover 来描述这个元素状态。虽然和普通css很类似，但是它只有处于dom树无法描述的状态下才能为元素添加样式。


**伪元素是用来创建不存在文档树中的元素**，并且为其添加对应的样式。比如说，我们可以通过:before来在一个元素前增加一些文本，并为这些文本添加样式，虽然用户可以看到这些文本，但是这些文本实际上不在文档树中。


**伪类**


下图中列出了可以用的伪类，在一个选择器里可以按需连用多个伪类

![css pseudo-classes-img]( https://limeii.github.io/assets/images/posts/css/css-pseudo-classes-img.png){:height="100%" width="100%"}

具体各种伪类的用法可以参考MDN： [https://developer.mozilla.org/zh-CN/docs/Web/CSS/Pseudo-classes](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Pseudo-classes)

**伪元素**


下图列出了所有伪元素，有时你会发现伪元素使用了两个冒号（::）而不是一个冒号（：），在CSS3之前的版本中伪元素是一个冒号，CSS3以后为了区分伪类和伪元素，伪元素改为两个冒号，大多数的浏览器是支持使用这两种方式表达伪元素。

![css css-pseudo-elements-img]( https://limeii.github.io/assets/images/posts/css/css-pseudo-elements-img.png){:height="100%" width="100%"}

具体各种伪元素的用法可以参考MDN： [https://developer.mozilla.org/zh-CN/docs/Web/CSS/Pseudo-elements](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Pseudo-elements)