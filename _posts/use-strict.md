- 在```use strict```模式下，使用没有声明的变量会报错，而非严格模式下，会自动创建一个全局变量。

```js
function doSomething() {
    greeting = "halo";
}
doSomething();
console.log(greeting); // 输出 halo

```
在上面的这个例子中，可以看到，当没有声明```greeting```这个变量的时候，直接给这个变量赋值，会自动创建一个全局变量```greeting```。来看看在```use strict```模式下，是什么行为：

```js
"use strict"
function doSomething() {
    greeting = "halo"; //Uncaught ReferenceError: greeting is not defined
}
doSomething();
console.log(greeting);
```
在```use strict```模式下，没有声明变量，直接用该变量会报错。


- ```var```不是块级变量

```js
function something() {
    var special = "JavaScript";
    {
        let special = 42;   // totally fine shadowing
        // ..
    }
}

function another() {
    // ..
    {
        let special = "JavaScript";
        {
            var special = "JavaScript";   // Syntax Error， 因为不是块级变量，相当于在another这个方法作用域里重复定义。
            // ..
        }
    }
}
```

```js
function another() {
    // ..
    {
        let special = "JavaScript";

        whatever(function callback(){
            var special = "JavaScript";   // totally fine shadowing,回调函数在这里是全局变量。
            // ..
        });
    }
}
```