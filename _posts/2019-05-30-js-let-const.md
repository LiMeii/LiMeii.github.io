---
title: JS：let和const
tags: JS
layout: post
---

在JS中用var声明变量存在变量提升，比如：
```js
if (false) {
    var a = 1;
}
console.log(a);//undefined
```
这段代码输出的是undefined，而不是ReferenceError。这是因为var有变量提升，实际这段代码相当于：
```js
var a;
if (false) {
    a = 1;
}
console.log(a);//undefined
```
还有在for循环中：
```js
var result = [];
for (var i = 0; i < 5; i++) {
    result.push(function () { console.log(i) });
}
console.log(result[0]());//5
console.log(result[1]());//5
console.log(result[2]());//5
console.log(result[3]());//5
console.log(result[4]());//5
console.log(i);//5
```
数组里输出的值都是5，并且for循环结束了，还是可以访问i的值。在ES6之前，是没有块级作用域，在ES6中引入了块级作用域，块级作用域是指：
- 函数内部
- 块中(字符 { 和 } 之间的区域)

## let 和 const
let和const是用来声明块级作用域变量，它们有以下特性：
### 1. 不会被提升
```js
if (false) {
    let a = 1;
}
console.log(a);//Uncaught ReferenceError: a is not defined
```
### 2. 重复声明会报错
```js
var a = 1;
let a = 2; //Uncaught SyntaxError: Identifier 'a' has already been declared
```
### 3. 不绑定全局作用域
在全局作用就里使用```var```声明变量或者 ```function```(只用这两种形式)，这个变量会作为全局对象的属性。
```js
var a = 1;
console.log(window.a);//1
```
但是用let和const不会：
```js
let a = 1;
const b = 2;
console.log(window.a);//undefined
console.log(window.b);//undefined
```

### let和const的区别
const用来声明常量，一旦初始化，就不能再被修改，否则会报错。
但是对于const声明的对象，对象的引用不能被修改，但是对象里的属性值是可以被修改的：
```js
const data = {
    a: 1
}

data.a = 2;//可以被修改
data.b = 3;//会加一个属性b

data = {};//报错：Uncaught TypeError: Assignment to constant variable
```

### 暂时性死区(Temporal Dead Zone)

在JS引擎扫描代码发现变量声明时，遇到var声明就提升到作用域顶部，遇到let和const就把这些声明放在暂时性死区。对于let和const变量，如果在执行它们的声明语句之前访问会报错，只有执行完声明语句之后才会从暂时性死区移出。

<blockquote>
<p>
The time between entering the scope of a variable and executing its declaration is called the temporal dead zone (TDZ) of that variable.
</p>
<p>
During this time, the variable is considered to be uninitialized (as if that were a special value it has).
</p>
<p>
If you access an uninitialized variable, you get a ReferenceError.
</p>
<p>
Once you reach a variable declaration, the variable is set to either the value of the initializer (specified via the assignment symbol) or undefined – if there is no initializer.
</p>
</blockquote>


需要注意的是let const 会变量提升，但是在执行声明语句之前，是放在暂时性死区，提前调用就会报错；想象一下如果let const没有变量提升的话，在声明之前调用let const变量，就会当成```var = halo```处理了。

```js
function  saySomething() {
    greeting = "halo"; // Uncaught ReferenceError: Cannot access 'greeting' before initialization
    let greeting;    
    console.log(greeting);
}
saySomething();
```

### 循环中的块级作用域
之前有代码：
```js
var result = [];
for (var i = 0; i < 5; i++) {
    result.push(function () { console.log(i) });
}
console.log(result[0]());//5
console.log(result[1]());//5
console.log(result[2]());//5
console.log(result[3]());//5
console.log(result[4]());//5
```
我们实际想要的输出0 1 2 3 4，解决方案有：
```js
var result = []
for (var i = 0; i < 5; i++) {
    result.push((function (i) { console.log(i) })(i));
}
console.log(result[0]());//0
console.log(result[1]());//1
console.log(result[2]());//2
console.log(result[3]());//3
console.log(result[4]());//4
```
还可以用ES6中的let：
```js
var result = []
for (let i = 0; i < 5; i++) {
    result.push(function () { console.log(i) });
}
console.log(result[0]());//0
console.log(result[1]());//1
console.log(result[2]());//2
console.log(result[3]());//3
console.log(result[4]());//4
```
之前说过，let不提升，不能重复定义，在第二次循环的时候，又用let声明了i，为什么在这里不报错，可以正确输出值呢？
比如：
```js
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i);
}
```
这里会输出三个abc，但是用var：
```js
for (var i = 0; i < 3; i++) {
  var i = 'abc';
  console.log(i);
}
```
只会输出一个abc。


JS引擎对var和let使用了不同的方式处理，对于let每次循环都会创建一个新变量，对于代码：
```js
var result = []
for (let i = 0; i < 5; i++) {
    result.push(function () { console.log(i) });
}
```
实际相当于：
```js
(let i = 0) {
    result[0]= function () { console.log(0) };
}
(let i = 1) {
    result[1]= function () { console.log(1) };
}
(let i = 2) {
    result[2]= function () { console.log(2) };
}
......
......
(let i = 4) {
    result[4]= function () { console.log(4) };
}
```
但是在上面代码里不能用const，虽然我们每次创建了一个新的变量，但是我们尝试修改const的值，所以会报错

## 总结
- 在开发过程中，默认推荐使用const，只有当确定变量的值会发生改变使用let
- 函数提升优先于变量提升，函数提升会把整个函数挪到作用域顶部，变量提升只会把声明挪到作用域顶部
- var 存在提升，我们能在声明之前使用。let、const 因为暂时性死区的原因，不能在声明前使用
- var 在全局作用域下声明变量会导致变量挂载在 window 上，其他两者不会
- let 和 const 作用基本一致，但是后者声明的变量不能再次赋值
- 只有 var 函数 有变量提升，其他的 class let const import 都没有变量提升， 如果函数表达式是 let const 也不会变量提升