---
title: JS：深浅拷贝
tags: JS
layout: post
---

我们先来看下数组的拷贝，通常我们会用```slice()``` ```concat()```方法实现数组拷贝：

```js
var arr = [1, 2, 3, 4, 5];
var arr1 = arr.concat();
var arr2 = arr.slice();
console.log(arr1); //[1, 2, 3, 4, 5]
console.log(arr2); //[1, 2, 3, 4, 5]
```
```slice()``` ```concat()```都是返回了一个新数组，没有改变原来的数组，看起来像是深拷贝。

我们再来看一个例子：
```js
var arr = [1, 2, 3, 4, { value: 5 }];
var arr1 = arr.concat();
var arr2 = arr.slice();
arr[4].value = 6;
console.log(arr1); //[1, 2, 3, 4, { value: 6 }]
console.log(arr2); //[1, 2, 3, 4, { value: 6 }]
```
这边可以看到，如果数组里有对象，那么不会拷贝对象的值，而是拷贝对象的引用，原来数据对象的值发生改变，由于拷贝的是对象的引用，新拷贝的数组中对象的值也发生改变，拷贝的不是很彻底，```slice()``` ```concat()```是**浅拷贝**，对于对象，我们通常通过```assign()```或者展开运算符```...```来实现拷贝，同样如果对象里嵌套了对象，也只能实现浅拷贝：

```js
var obj = {
    name: "mei",
    address: {city: "shanghai"}
}
var obj1 = Object.assign({},obj);
var obj2 = {...obj};
obj.address.city = "beijing";
console.log(obj1); //{name: "mei", address:{city: "beijing"}
console.log(obj2); //{name: "mei", address:{city: "beijing"}
```

## 利用JSON.stringify实现对数组和对象的深拷贝
```js
var arr = [1, 2, 3, 4, { value: 5 }];
var arr1 = JSON.parse(JSON.stringify(arr));
arr[4].value = 6;
console.log(arr1); //[1, 2, 3, 4, { value: 5 }]

var obj = {
    name: "mei",
    address: {city: "shanghai"}
}
var obj1 = JSON.parse(JSON.stringify(obj));
obj.address.city = "beijing";
console.log(obj1); //{name: "mei", address:{city: "shanghai"}
```
该方法不能对```undefined``` ```symbol``` ```函数``` 进行深度拷贝。

## 利用递归实现数组和对象深拷贝
```js
function deepCopy(obj) {
    if (typeof obj !== 'object') return;
    var newObj = obj instanceof Array ? [] : {};
    for (var key in obj) {
        if (obj.hasOwnProperty(key)) {
            newObj[key] = typeof obj[key] === 'object' ? deepCopy(obj[key]) : obj[key];
        }
    }
    return newObj;

}
```
我们来验证下：
```js
var arr = [1, 2, 3, 4, { value: 5 }];
var arr2 = deepCopy(arr);
arr[4].value = 6;
console.log(arr);//1, 2, 3, 4, { value: 6 }]
console.log(arr2);//1, 2, 3, 4, { value: 5 }]
```
对于数组和对象的浅拷贝，去掉递归就可以了：
```js
function shallowCopy(obj) {
    if (typeof obj !== 'object') return;
    var newObj = obj instanceof Array ? [] : {};
    for (var key in obj) {
        if (obj.hasOwnProperty(key)) {
            newObj[key] = obj[key];
        }
    }
    return newObj;

}
```
验证一下：
```js
var arr = [1, 2, 3, 4, { value: 5 }];
var arr2 = shallowCopy(arr);
arr[4].value = 6;
console.log(arr);//1, 2, 3, 4, { value: 6 }]
console.log(arr2);//1, 2, 3, 4, { value: 6 }]
```