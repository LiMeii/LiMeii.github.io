---
title: JS：深入理解JavaScript-原型
tags: JS
layout: post
---


在JS中每个函数都有一个prototype属性（需要注意的是只有函数才有prototype），它实际指向的是一个prototype对象，那Foo.prototype这个对象里到底有什么呢？

```js
function Foo() {

}

Foo.prototype.name = "mei";

console.log(Foo.prototype);
```
输出结果为：
![js-prototype](/assets/images/posts/js/js-prototype01.png){:height="100%" width="100%"}

我们可以看到，整个prototype对象就有两个属性，一个是constructor，一个是__proto__；constructor是构造函数，__proto__是Object。也就是```function Foo() {}```的prototype是Object。


我们再来看一个例子：

```js

function Foo() {

}

Foo.prototype.name = "mei";

var f1 = new Foo();
var f2 = new Foo();

console.log(Foo.prototype);
console.log(f1.__proto__);
console.log(f2.__proto__);
```
<blockquote>
<p>
每一个JavaScript对象(除了 null )都具有的一个属性，叫__proto__，这个属性会指向该对象的原型
</p>
</blockquote>

我们可以看到，三个输出都是一样的：
![js-prototype](/assets/images/posts/js/js-prototype01.png){:height="100%" width="100%"}

在使用```new```关键字创建f1和f2的时候，自动的把f1.__proto__和f2.__proto__指向Foo.prototype这个对象，也就是```f1.__proto__``` ```f2.__proto__``` ```Foo.prototype```指向都是同一个原型对象。


我们可以来验证一下：
```js

function Foo() {

}

Foo.prototype.name = "mei";

var f1 = new Foo();
var f2 = new Foo();

console.log(f1.__proto__ === Foo.prototype); //true
console.log(f2.__proto__ === Foo.prototype); //true

```

我们再来看下原型的原型里有什么：
```js
function Foo() {

}

Foo.prototype.name = "mei";

console.log(Foo.prototype.__proto__ );
```

前面用```console.log(Foo.prototype)```可以```function Foo() {}```的prototype是Object，那么```console.log(Foo.prototype.__proto__ )```就是Object的结果如下：

![js-prototype](/assets/images/posts/js/js-prototype02.png){:height="80%" width="80%"}

构造函数、实例对象、原型和原型的原型的关系图如下：
![js-prototype](/assets/images/posts/js/js-prototype03.png){:height="100%" width="100%"}

原型链就是下图中红色这条线：
![js-prototype](/assets/images/posts/js/js-prototype04.png){:height="100%" width="100%"}

