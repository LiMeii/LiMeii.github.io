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


- shadowing

```var```不是块级变量，shadowing会不起作用。

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

window.something = 24;

let something = "limeii";

console.log(something); // limeii, 这个是合法的。
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


- anonymous function expressions  vs named function expressions

函数声明和函数表达式的主要区别：函数声明有变量提升，函数表达式不会有变量提升。

**anonymous function express:**
```js
  // askQuestion 是个全局变量，在函数执行之前，被赋值为undefined
  var askQuestion = function (){
    // ..
};
```

**named function express:**
```js
// askQuestion是个全局变量， ofTheTeacher是function内部的变量，而且是read-only
  var askQuestion = function ofTheTeacher(){
    // ..
};
```

```js
var askQuestion = function ofTheTeacher(){
    console.log(ofTheTeacher);
}

askQuestion();//function ofTheTeacher();

console.log(ofTheTeacher);// ReferenceError: "ofTheTeacher" is not defined
```

```js
var askQuestion = function ofTheTeacher() {
    "use strict";
    ofTheTeacher = 42;   // this assignment fails, it's readonly

    //..
};

askQuestion();
// TypeError
```

- arrow function
arrow function 是一种特殊的匿名函数，它和function一样，会有自己的scope，唯一有区别的是它没有this，它的this是离他最近的this。

- function hoisting 方法提升 (只有函数声明会提升)

```js
greeting();
// Hello!

function greeting() {
    console.log("Hello!");
}
```

```js
greeting();//uncaught TypeError: greeting is not a funtion

var greeting = function () {
    console.log('hello');

}
```
<blockquote>
<p>
注意这边不是报ReferenceError，ReferenceError表示当前变量不存在，在这里greeting在内存里存在的，并且值为undefined，undefined不是一个方法，所以第一行会报错，这个时候报的是uncaught TypeError。
</p>
</blockquote>

<blockquote>
<p>
TypeError 和 syntaxError的区别在于：TypeError发生在代码执行阶段，syntaxError发生在代码解析阶段。
</p>
</blockquote>

- 闭包的应用
  外层的scope可以访问里面作用的变量，也就是执行上下文执行完后，它还保留了一个引用，其他的执行上下文可以通过这个引用访问它内部的变量。


  **第一种:**
     ```js
        function foo() {
        var a = 2;

        function bar() {
            console.log( a );
        }

        return bar;
        }

        var baz = foo();

        baz(); // 2 -- Whoa, closure was just observed, man.
     ```

  **第二种:**
     ```js
        function foo() {
        var a = 2;

        function baz() {
            console.log( a ); // 2
        }

        bar( baz );
        }

        function bar(fn) {
            fn(); // look ma, I saw closure!
        }
     ```

  **第三种:**
     ```js
        var fn;

        function foo() {
            var a = 2;

            function baz() {
                console.log( a );
            }

            fn = baz; // assign `baz` to global variable
        }

        function bar() {
            fn(); // look ma, I saw closure!
        }

        foo();

        bar(); // 2
     ```

     
  **第四种:回调**
  ```js
    function wait(message) {

        setTimeout( function timer(){
            console.log( message );
        }, 1000 );

    }

    wait( "Hello, closure!" ); // 打印出 hello closure！ 回调函数的message是wait方法作用域的值。
  ```

  需要注意的是，如果代码写成下面的样子：
  
  ```js
    function wait(message) {

        setTimeout( function timer(){
            console.log( this.message );
        }, 1000 );

    }

    wait( "Hello, closure!" ); // undefined, 回调函数的this值的是全局变量，全局变量没有这个值，所以是undefined。
  ```

  **第五种：模块**

  ```js
    var obj = (function foo() {
        var a = 2;

        function baz() {
            console.log(a); // 2
        }

        return {
            baz: baz()
        };
    })();

    obj.baz;
  ```
