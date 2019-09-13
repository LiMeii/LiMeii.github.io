---
title: JS：模拟实现call apply和bind方法
tags: JS
layout: post
---

首先来说下这三个的区别：
- ```call``` ```apply```都是为了解决this的指向，默认第一参数是this的指向，剩下的参数是函数形参，```call```接收的形参是一个列表用逗号隔开，```apply```接收的是一个参数数组。
- ```call``` ```apply```改变函数的this指向以后立马执行该函数，而```bind```是返回一个绑定上下文的新函数，后续再执行。
- bind函数返回的新函数不可以再通过apply call改变它的this指向。

```js
let a = {
    value: 1
}
function getValue(name, age) {
    console.log(name)
    console.log(age)
    console.log(this.value)
}
getValue.call(a, 'mei', '24'); // mei 24 1; 立即执行
getValue.apply(a, ['li', '24']);// li 24 1；立即执行
getValue.bind(a)('mei', '24'); // mei 24 1；('mei', '24')这部分才是执行函数
getValue.bind(a)(['li', '24']);// ['li', '24'] undefined 1；(['li', '24'])才是执行函数
```

## 模拟实现call
```js
Function.prototype.myCall = function (context) {
  var context = context || window
  // 给 context 添加一个属性
  // getValue.call(a, 'mei', '24') => a.fn = getValue
  context.fn = this
  // 将 context 后面的参数取出来
  var args = [...arguments].slice(1)
  // getValue.call(a, 'mei', '24') => a.fn('mei', '24')
  var result = context.fn(...args)
  // 删除 fn
  delete context.fn
  return result
}

```
测试一下：
```js
let a = {
    value: 1
}
function getValue(name, age) {
    console.log(name)
    console.log(age)
    console.log(this.value)
}
getValue.myCall(a, 'mei', '24');//mei 24 1
```
## 模拟实现apply
```js
Function.prototype.myApply = function (context) {
    var context = context || window;
    context.fn = this;
    var result;
    if (arguments[1]) {
        result = context.fn(...arguments[1]);
    } else {
        result = context.fn()
    }
    delete context.fn;
    return result;
}
```
测试一下：
```js
let a = {
    value: 1
}
function getValue(name, age) {
    console.log(name)
    console.log(age)
    console.log(this.value)
}
getValue.myApply(a, ['mei', '24']);//mei 24 1
```
## 模拟实现bind
```js
Function.prototype.myBind = function (context) {
    if (typeof this !== 'function') {
        throw new TypeError('error');
    }
    var _this = this;
    var args = [...arguments].slice(1);
    return function F() {
        if (this instanceof F) {
            return new _this(...args, ...arguments)
        }
        return _this.apply(context, args.concat(...arguments))
    }
}
```
测试一下：
```js
let a = {
    value: 1
}
function getValue(name, age) {
    console.log(name)
    console.log(age)
    console.log(this.value)
}
getValue.myBind(a, 'mei', '24')();//mei 24 1
```