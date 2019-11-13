---
title: JS：箭头函数
tags: JS
layout: post
---

先来看下ES6中箭头函数的基本语法：

```js
let func = value => value;
```

相当于：

```js
let func = function (value) {
    return value
}
```

如果需要传入多个参数：

```js
let func = (value, num) => value * num;
```

上面箭头函数例子中都省略了```return```关键字和代码的花括号，在箭头函数中如果方法体中只有一行代码，可以省略关键字和方法体的花括号，直接简化成```value => value```。


如果函数的代码块需要有多条语句：

```js
let func = (value, num) => {
    return value * num;
}
```

如果需要返回一个对象，箭头函数的方法体必须放在大括号```()```中，这样做的原因是：没有大括号，JS引擎没办分区分是正常定义一个对象还是一个箭头函数体:

```js
let func = (value, num) => ({ value: value, num: num }); //正确写法

let func = (value, num) => { value: value, num: num }; //会报错
```

## 与普通函数的区别
###  没有this
箭头函数式的this需要通过查找作用域链来确定，它的this是指包在它外面的作用域的this，我们来看下以下代码中的this分别指的是什么：

```js
const obj = {
    a: function () {
        console.log(this); // obj
    },
    b: () => {
        console.log(this); // windows
    }
}
```

obj.b是一个箭头函数，它的this是包在它外层的词法作用域的this（离它最近的词法作用域），obj对象不是可执行代码，所以它不是离箭头函数最近的词法作用域，再往外就是全局作用域window了，所以obj.b的this指的是windows。

再来看一段代码：

```js
var pageHandler = {

    id: "123456",

    init: function () {
        document.addEventListener("click", function (event) {
            this.doSomething(event.type);     // error
        }, false);
    },

    doSomething: function (type) {
        console.log("Handling " + type + " for " + this.id);
    }
};

```

在调用```pageHandler.init```方法的时候会报错，报错的函数是个回调函数，这个回调函数的this指的全局变量windows，在全局变量里没有doSomething这个方法，所以会报错。那么怎么才能让它不报错呢？


第一种方式是通过bind来指定this：
```js
var pageHandler = {
    id: "123456",
    init: function () {
        document.addEventListener("click", (function (event) {
            this.doSomething(event.type)
        }).bind(this), false)

    },
    doSomething: function () {
        console.log("Handling" + type + " for " + this.id)
    }
}
```

.bind(this)中的this是指pageHandler这个对象，通过bind生成一个this指向pageHandler的新函数，这样执行init方法的时候不会报错，看起来怪怪的有没有。


第二种方式是通过箭头函数：
```js
var pageHandler = {
    id: "12345",
    init: function () {
        document.addEventListener("click", () => {
            this.doSomething();
        }, false)
    },
    doSomething: function () {
        console.log("Handling" + type + " for " + this.id);
    }
}
```
把init中的回调函数改成了箭头函数，箭头函数的this是它最近的作用域链上的this，也就是init这个方法的this，也就是pageHandler这个对象，这样就可以达到目的不报错啦。


**需要注意的是，箭头函数不能改变this的值，普通函数可以通过call、apply、bind来指定this，但是箭头函数的this是不能改的。**

### 没有arguments
访问箭头函数的arguments，其实也是访问包在它外面的非箭头函数的arguments。

```js
function foo(x, y) {
    return () => arguments[0];
}
console.log(foo(1,2)()); //1
```

### 不能通过new关键字调用。
```js
var Foo = () =>{};
var foo = new Foo(); // TypeError: Foo is not a constructor
```
### 没有原型
没有```prototype```, 但是有```__proto__```指向function。

### 没有super，也是通过包在它外面的非箭头函数来决定的。
