---
title: JS：深入理解JavaScript-继承
tags: JS
layout: post
--- 

在文章【[JS：深入理解JavaScript-原型](https://limeii.github.io/2019/05/js-prototype/)】详细介绍了原型和```new``` ```Object.create()```在创建对象和实现继承的区别。JS的继承是基于原型（prototype）实现的，再理解原型之后，就很容易理解JS的继承方式了。


在这篇文章里将会详细介绍6中常见的JS继承方式和ES6 class继承以及它们各自的优缺点。

## 1. 原型继承
```js

function Animal() {
    this.category = "category";
}

Animal.prototype.getCategory = function () {
    console.log(this.category);
}

Animal.prototype.price = 100;
Animal.prototype.breed = {
    color: "black",
    age: 1
}

function Cat(name) {
    this.name = name;
    this.getName = function () {
        console.log(this.name);
    }
}

Cat.prototype = new Animal();

var cat1 = new Cat("kitty");
var cat2 = new Cat("hua");

console.log(cat1, cat2);
```
![js-inheritance](/assets/images/posts/js/js-inheritance01.png){:height="100%" width="100%"}

这种方式把子类的原型指向了**父类的实例**，所以**子类的示例可以通过原型链访问到父类的实例```new Animial()```，然后通过原型链向上可以访问到```Animal.prototype``**，就可以实现子类实例可以继承和访问父类的属性和方法。


这种方式的原型链如下：
![js-inheritance](/assets/images/posts/js/js-inheritance02.png){:height="100%" width="100%"}

我们都知道在操作基本数据类的时候操作的是值，在操作引用数据类型的时候操作的是引用地址，那么在cat1中更改breed属性的值，会同时影响cat2中这个属性的值，如下：

```js

cat1.price = 150;
console.log(cat1.price);//150
console.log(cat2.price);//100

cat1.breed.color = "white";
console.log(cat1.breed.color);//white
console.log(cat2.breed.color);//white cat2的bredd属性也会跟着变化
```

优点：
- 父类/父类原型新增的属性和方法，子类都可以访问
- 简单，易于实现

缺点：
- 无法实现多继承，cat1.breed.color改变，会同时导致cat2.breed.color变化
- 原型对象的所有属性都被多有实例共享，不管是私有还是公有属性
- 创建子类实例，无法像父类构造函数传参
- 想要为子类新增属性和方法，必须要在```Cat.prototype = new Animal()```之后执行，不能放在构造器中