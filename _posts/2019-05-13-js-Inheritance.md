---
title: JS：深入理解JavaScript-继承
tags: JS
layout: post
--- 

在文章【[JS：深入理解JavaScript-原型](https://limeii.github.io/2019/05/js-prototype/)】详细介绍了原型和```new``` ```Object.create()```在创建对象和实现继承的区别。JS的继承是基于原型（prototype）实现的，再理解原型之后，就很容易理解JS的继承方式了。


在这篇文章里将会详细介绍6中常见的JS继承方式。

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

这种方式把子类的原型指向了**父类的实例**，所以**子类的实例可以通过原型链访问到父类的实例```new Animial()```，然后通过原型链向上可以访问到```Animal.prototype```**，就可以实现子类实例可以继承和访问父类的属性和方法。


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
- 无法实现多继承
- 原型对象的引用属性都被多个实例共享，不管是私有还是公有属性
- 创建子类实例，无法像父类构造函数传参
- 想要为子类原型新增属性和方法，必须要在```Cat.prototype = new Animal()```之后执行，不能放在构造器中

## 2. 借用构造函数继承（经典继承）
这种方式的关键在于，在子类的构造函数中通过call()调用父类的构造函数。

```js
function Animal(category) {
    this.category = category;

    this.others = {
        other1: 1,
        other2: 2
    };
    this.setCategory = function () {
        console.log("set category");
    }
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

    Animal.call(this, "cat")
}


var cat1 = new Cat("kitty");
var cat2 = new Cat("hua");

console.log(cat1, cat2);
```
![js-inheritance](/assets/images/posts/js/js-inheritance03.png){:height="100%" width="100%"}

在执行```var cat1 = new Cat("kitty")```的时候，子类构造函数```Cat```中的```this```指的是```cat1```这个实例对象，```cat1```这个实例对象继承了父类```Animal```中的属性和方法，但是不能访问```Animal.prototype```中的属性和方法，```cat2```也是同理。


原型链关系图如下：
![js-inheritance](/assets/images/posts/js/js-inheritance04.png){:height="100%" width="100%"}

子类实例cat1和cat2可以继承Cat.prototype和Animal的属性和方法，但是不能访问Animal.prototype的属性和方法，如下：

```js
cat1.getCategory();//Uncaught TypeError: cat1.getCategory is not a function
cat1.price // undefined
```

子类实例cat1和cat2从父类Animal那继承的属性和方法不共享，在各自的内存中有一份独立的，如下：
```js
cat1.others.other1=123；
console.log(cat1.others.other1);//123
console.log(cat2.others.other1);//1
```

优点：
- 解决了原型链中子类实例共享父类引用属性的问题
- 创建子类实例，可以向父类传递参数
- 可以实现多继承（call多个父类对象）

缺点：
- 实例并不是父类的实例，只是子类的实例
- 只能继承父类的实例属性和方法，不能继承父类原型属性和方法
- 无法实现函数复用，每个子类都有父类实例函数的副本，影响性能

## 3. 原型链+借用构造函数的组合继承

这种方式的关键是：通过调用父类构造函数，继承父类的属性并且可以向父类传递参数，然后再通过将父类实例作为子类原型，实现函数复用。
```js
function Animal(category) {
    this.category = category;
    this.others = {
        other1: 1,
        other2: 2
    };
    this.setCategory = function () {
        console.log("set category");
    }
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
    Animal.call(this, "cat");
}

Cat.prototype= new Animal();
Cat.prototype.constructor = Cat;//组合方式需要修复构造函数指向

Cat.prototype.sayHello = function() {}

var cat1 = new Cat("kitty");
console.log(cat1);
```
![js-inheritance](/assets/images/posts/js/js-inheritance05.png){:height="100%" width="100%"}

这种方式的原型链关系和原型继承是一样的：
![js-inheritance](/assets/images/posts/js/js-inheritance02.png){:height="100%" width="100%"}

与原型继承不同在于：通过在子类构造函数通过call()调用父类的构造函数，可以向父类构造函数传递参数，并且不会共享父类中引用属性的值，如下：
```js
var cat1 = new Cat("kitty");
var cat2 = new Cat("hua");
cat1.others.other1=123
console.log(cat1.others.other1);//123
console.log(cat2.others.other1);//1
```
这种方法结合了原型继承和借用构造函数继承的优点，是JS中最常用的继承模式，不过也存在缺点，就是无论在什么时候都会调用两次父类构造函数：


一次是设置子类型实例的原型的时候：
```js
Cat.prototype= new Animal();
```
一次在创建子类型实例的时候：
```js
var cat1 = new Cat("kitty"); // Parent.call(this, name);
```

优点：

- 可以继承父类的属性和方法，也可以继承父类原型的属性和方法
- 不存在引用属性共享问题
- 可以传参给父类构造函数
- 函数可以复用

缺点：

- 调用了两次构造函数，生成了两份实例



## 4. 原型式继承

通过```Object.create```，将出入的第一个参数作为创建对象的原型。
```js
function Animal(category) {
    this.category = category;
    this.others = {
        other1: 1,
        other2: 2
    };
    this.setCategory = function () {
        console.log("set category");
    }
}

Animal.prototype.getCategory = function () {
    console.log(this.category);
}
Animal.prototype.price = 100;
Animal.prototype.breed = {
    color: "black",
    age: 1
}

var animal1 = Object.create(Animal);
var animal2 = Object.create(Animal);

```
优点：

- 父类/父类原型新增的属性和方法，子类都可以访问
- 简单，易于实现

缺点：

- 无法实现多继承
- 原型对象的引用属性都被多有实例共享，不管是私有还是公有属性
- 创建子类实例，无法像父类构造函数传参

## 5. 寄生式继承
创建一个尽用于封装过程的函数，该函数在内部以某种形式做增强对象，最后返回对象。
```js
function createObj(o) {
    var clone = Object.create(o);
    console.sayHi = function () {
        console.log(hi);
    }
    return clone;
}
```
缺点：跟借用构造函数模式一样，无法实现函数复用，每次创建对象都会创建一遍方法。

## 6. 寄生组合式继承
在原型链+借用构造函数的组合继承这种方式中，最大的缺点就是会调用两次父类的构造函数，那可不可以避免一次重复调用呢？


不使用```Cat.prototype = new Animal()```，而是间接就让Cat.prototype访问Animal.prototyoe呢？
```js
function Animal(category) {
    this.category = category;
    this.others = {
        other1: 1,
        other2: 2
    };
    this.setCategory = function () {
        console.log("set category");
    }
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
    Animal.call(this, "cat")
}

var F = function () { } //核心代码
F.prototype = Animal.prototype; //核心代码

Cat.prototype = new F();

var cat1 = new Cat("kitty");
console.log(cat1);
```

