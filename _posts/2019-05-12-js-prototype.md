---
title: JS：深入理解JavaScript-原型
tags: JS
layout: post
---

在这篇文章里会介绍如下内容：
- 什么是原型和原型链
- ```prototype``` 和```__proto__```有什么区别
- ```new``` 和```Object.create()```创建对象和实现继承的区别

## 原型

在JS中每个函数都有一个prototype属性，它实际指向的是一个prototype对象，比如有一个函数Foo，我们来看下Foo.prototype这个对象里到底有什么：

```js
function Foo() {

}

Foo.prototype.name = "mei";

console.log(Foo.prototype);
```
输出结果为：
![js-prototype](/assets/images/posts/js/js-prototype01.png){:height="100%" width="100%"}

我们可以看到，整个prototype对象就有两个属性，一个是constructor，一个是__proto__；constructor是构造函数，__proto__是Object。也就是```function Foo() {}```的prototype是Object。

<blockquote>
<p>
每一个JavaScript对象(除了 null )都具有的一个属性，叫__proto__，这个属性会指向该对象的原型。
</p>
</blockquote>


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

前面用```console.log(Foo.prototype)```可以看到```function Foo() {}```的prototype是Object，那么```console.log(Foo.prototype.__proto__ )```就是Object.prototype的结果如下：

![js-prototype](/assets/images/posts/js/js-prototype02.png){:height="80%" width="80%"}

我们经常在写代码时候会直接用```toString()```方法，就是定义在Object.prototype里。```hasOwnProperty```是用来判断一个对象是否包含自定义属性而不是原型链上的属性，它是JavaScript中唯一一个处理属性但是不查找原型链的函数。

构造函数、实例对象、原型和原型的原型的关系图如下：
![js-prototype](/assets/images/posts/js/js-prototype03.png){:height="100%" width="100%"}

原型链就是下图中红色这条线：
![js-prototype](/assets/images/posts/js/js-prototype04.png){:height="100%" width="100%"}

原型链的用处就是，如果在当前对象里找不到某个属性或者方法，会沿着原型链向上找，一直找到Object.prototype为止，找到第一个匹配的就停止，如果找不到就返回undefined。JavaScript中的原型继承就是基于原型和原型链实现的。比如在f1和f2中访问name属性，就是从它的上一层Foo.prototype中拿到的，而age在原型Foo.prototype和Object里都没有，就返回undefined：

```js

function Foo() {

}

Foo.prototype.name = "mei";

var f1 = new Foo();
var f2 = new Foo();

console.log(f1.name); //mei
console.log(f2.name); //mei
console.log(f1.age);//undefined

```
### prototype总结
- 所有对象都有一个 ```__proto__ ```指向一个对象，也就是原型
- 每个对象的原型都可以通过```constructor```找到构造函数，构造函数也可以通过```prototyoe```找到原型
- 所有函数都可以通过```__proto__ ```找到```Function```对象
- 所有对象都可以通过```__proto__ ```找到```Object```对象
- 对象之间通过```__proto__ ```连接起来，就是原型链。当前对象不存在的属性，通过原型链层层往上找，直到最上层```Object```对象

## ```prototype``` vs ```__proto__```

- ```prototype``` 是构造函数的属性

- ```__proto__ ``` 是实例的属性

示例代码如下：
```js
function Foo() {
    console.log("hi");
}

obj = {
    a: 1
}
Foo.prototype.name = "mei";
f1 = new Foo();
console.log(Foo.prototype); //{name: "mei", constructor: ƒ Foo(), __proto__:Object}
console.log(f1.__proto__); //{name: "mei", constructor: ƒ Foo(), __proto__:Object}
console.log(f1.prototype); // undefined

console.log(obj.prototype); //undefined
console.log(obj.__proto__); // Object
```

两者之间关系图如下：
![js-common](/assets/images/posts/js/js-common01.png){:height="80%" width="80%"}

## ```new``` vs ```Object.create()```

在JS中，继承是通过prototype实现的，JS中创建对象有两种方式```new``` 和 ```Object.create()```，我们来看看两者的区别，以及是如果通过prototype实现继承的。


两者都是为了用来创建对象实现继承，需要注意的是```new function ()``` ```new Array()``` ```new Boolean()``` ```new Object()``` ```new Number()``` ```new String（）``` 都是可以的，但是```new {a:1}```是非法的，需要用```Object.create({a:1})``` 。

两者最大的区别在于，```Object.create(proto[, propertiesObject])```可以用第一个参数指定新创建对象的原型，它的第二个参数是新对象的可枚举属性（可以省略，但不能为null）。


先来看下用```new```创建对象:

```js
function Foo() {
    console.log("hi");
}

Foo.prototype.name = "mei";

f1 = new Foo();
f2 = new Foo();

console.log(f1.__proto__); //{name: "mei", constructor: ƒ Foo(), __proto__:Object}
console.log(f2.__proto__); //{name: "mei", constructor: ƒ Foo(), __proto__:Object}

console.log(f1.__proto__ === f2.__proto__); //true
console.log(f1.__proto__ === Foo.prototype); //true

console.log(Foo.prototype.__proto__); // Object.prototype
console.log(Foo.prototype.__proto__.__proto__);//null

console.log(f1==f2);//false
```

关系图如下：
![js-common](/assets/images/posts/js/js-common02.png){:height="100%" width="100%"}

**通过上图可以看出，f1和f2通过图中红色的原型链，可以继承```Foo.prototype```和```Object.prototype```的属性和方法，需要注意的是f1和f2是两个空的函数。如果在通过f1改变```Foo.prototype```的引用值，那么同时也会影响f2的这个引用值；如果通过f1改变```Foo.prototype```的原始值，不会影响f2的原始值**

示例代码如下：

```js
function Foo() {
    console.log("hi");
}

Foo.prototype.name = "mei";
Foo.prototype.address = { country: "China", city: "shanghai" };

f1 = new Foo();
f2 = new Foo();

f1.name = "mei li";
f1.address.city = "beijing";

console.log(Foo.prototype.name);//mei
console.log(f1.name);//mei li
console.log(f2.name);//mei

console.log(Foo.prototype.address.city);//beijing
console.log(f1.address.city);//beijing
console.log(f2.address.city);//beijing
```

再来看下用```Object.create()```创建对象：
```js
obj = {
    name: "mei"
}

obj1 = Object.create(obj);
obj2 = Object.create(obj);

console.log(obj.__proto__); // Object.prototype
console.log(obj1.__proto__); // {name: "mei",  __proto__:Object}
console.log(obj2.__proto__); //{name: "mei",  __proto__:Object}

console.log(obj1.__proto__ === obj2.__proto__); // true
console.log(obj.__proto__ === obj1.__proto__); // false

console.log(obj.__proto__.__proto__); // null
console.log(obj1==obj2); //false
```

对于obj1和obj2的原型直接是obj1对象，而不是```obj.__proto__```，关系图如下：
![js-common](/assets/images/posts/js/js-common03.png){:height="100%" width="100%"}

**通过上图可以看出，obj1和obj2可以通过红色的原型链继承```obj```和```Object.prototype```的属性和方法，需要注意的是obj1和obj2是两个空的对象。如果在通过obj1改变```obj```的引用值，那么同时也会影响obj2的这个引用值；如果通过obj1改变```obj```的原始值，不会影响obj2的原始值。**

示例代码如下：
```js
obj = {
    name: "mei",
    address: {
        country: "China",
        city: "shanghai"
    }
}

obj1 = Object.create(obj);
obj2 = Object.create(obj);

obj1.name = "mei li";
obj1.address.city = "beijing";

console.log(obj.name);//mei
console.log(obj1.name);//mei li
console.log(obj2.name);//mei

console.log(obj.address.city);//beijing
console.log(obj1.address.city);//beijing
console.log(obj2.address.city);//beijing
```