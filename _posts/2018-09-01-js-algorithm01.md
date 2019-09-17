---
title: 算法
tags: JS
layout: post
---

## 数组去重
### 1. 嵌套循环
```js
function unique(array) {
    var result = [];
    var arrLength = array.length;
    for (var i = 0; i < arrLength; i++) {
        for (var j = 0; j < result.length; j++) {
            if (array[i] === result[j]) {
                break;
            }
        }
        if (j === result.length) {
            result.push(array[i]);
        }
    }
    return result;

}

var arr=[1,1,'1','1'];
console.log(unique(arr)); //[1,'1']
```

### 2. indexof
```js
function unique(array) {
    var result = [];
    var arrLength = array.length;
    for (var i = 0; i < arrLength; i++) {
        if (result.indexOf(array[i]) === -1) {
            result.push(array[i])
        }
    }
    return result;
}

var arr = [1, 1, '1', '1'];
console.log(unique(arr));//[1,'1']

```
### 3. 排序去重
```js
function unique(array) {
    var result = [];
    var sortedArray = array.concat().sort();
    var temp;
    for (var i = 0; i < sortedArray.length; i++) {
        if (!i || temp !== sortedArray[i]) {
            result.push(sortedArray[i]);
        }
        temp = sortedArray[i]
    }
    return result;
}


var arr = [1, 2, 1, '1', '1'];
console.log(unique(arr));//[1,'1',2]
```

### 4. 用ES6的Set
Set和数组很像，但是元素是唯一的，没有重复值。
```js
function unique(array) {
    return Array.from(new Set(array));
}
var arr = [1, 2, 1, '1', '1'];
console.log(unique(arr));//[1,2,'1']
```

## 数组去除指定的值
数组元素都是数字，去除指定的数字，这个算法里不能创建新的数组。比如数组为[2,2,3,3]，指定的数字为2，需要返回的数组为[3,3]。

```js
function removeElement(nums, val) {
    for (var i = nums.length - 1; i >= 0; i--) {
        if (nums[i] === val) {
            nums.splice(i, 1);
        }
    }
    return nums;
}

removeElement([0, 1, 2, 2, 3, 0, 4, 2], 2);//[0, 1, 3, 0, 4]
```