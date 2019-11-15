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

## string数组匹配
有一个stirng数组：words和一个string：chars，找出words中所有的string，这些string的每个字符都能在chars中匹配（chars中的每个字符只能匹配一次），计算出words中符合上述条件string的长度之和。


比如：words = ["cat","bt","hat","tree"]，chars = "atach"，words中符合条件的string为：cat 和hat，所以长度值和为6


 words = ["hello","world","leetcode"]，chars = "welldonehoneyr"，words中符合条件的string为：hello world，所以长度之和为10

```js
    return words.filter(ele => {
        let charsArr = Array.from(chars);
        let wordArr = Array.from(ele);
        let isMatch = true;
        wordArr.forEach(item => {
            if (!isMatch) return;
            var index = charsArr.indexOf(item);
            if (index > -1) {
                charsArr.splice(index, 1);
            } else {
                isMatch = false;
            }
        });
        return isMatch;
    }).join('').length;
```

## 找出给定数字在数组中的位置
有一个升序的数字数组nums，给定一个数字target，找出target数字在数组nums中的索引位置。
比如：nums = [1,3,5,6]，target = 5， 那么返回的结果为2


nums = [1,3,5,6]，target = 2，那么返回结果为1，2在数组中没有找到，所以返回它本应该在的索引位置。

```js
var searchInsert = function (nums, target) {
    var _index;
    var result;
    _index = nums.findIndex(function (ele) {
        return ele >= target;
    });
    if (_index === -1) {
        target > nums[nums.length - 1] ? result = nums.length : result = 0;
    } else {
        result = _index;
    }
    return result;
};
searchInsert([1,3,5,6],5)//2
searchInsert([1,3,5,6],0)//0
searchInsert([1,3,5,6],7)//4
```

## 找到不重复子字符串的最大长度

有一个字符串```pwwkew```, 不重复的字符串为```p``` ```pw``` ```wke``` ```kew``` 所以最大长度为3.

```js
  var start = 0, maxLen = 0;
  var map = new Map(); // key为字符，value当前字符的索引。由于Map的key只能唯一，所以后面存进去的值会覆盖前面的值。

  for(var i = 0; i < s.length; i++) {
      var ch = s[i];
    
      if(map.get(ch) >= start) start = map.get(ch) + 1; // 如果可以找到key值，开始重复，那么start的值就需要从后一个字符开始算，
      map.set(ch, i);
    
      if(i - start + 1 > maxLen) maxLen = i - start + 1; // 比较子字符串长度和当前最大长度。
  }

  return maxLen;
```