---
title: JS：深入理解JavaScript-闭包
tags: JS
layout: post
---

在文章【[JS：深入理解JavaScript-执行上下文](https://limeii.github.io/2019/05/js-execution-context/)】中介绍了代码在执行栈是如何运行的，假设有如下代码：
```js
function foo() {
    var a = 2;

    function bar() {
        console.log(a);
    }

    bar();
}

foo()
```

引擎在执行这段代码的步骤如下：
- 1：创建一个新的**执行上下文（Execution Context）**
- 2：创建一个新的**词法环境（Lexical Environment）**
- 3：把**LexicalEnvironment**和**VariableEnvironment**指向新创建的词法环境
- 4：把这个执行上下文压入**执行栈**并成为**正在运行的执行上下文**
- 5：执行代码
- 6：执行结束后，把这个执行上下文弹出执行栈

代码在执行完1-4步以后，整个环境看起来是这样的：

![js-closures](/assets/images/posts/js/js-closures01.png){:height="100%" width="100%"}

执行第五步，执行到foo会先给变量```a```赋值，然后给bar方法创建一个新的执行上下文，然后再执行```console.log(a)```:

![js-closures](/assets/images/posts/js/js-closures02.png){:height="100%" width="100%"}

执行第六步，```foo``` ```bar```执行完后被弹出执行栈，这两个function对象（红色区域1和2）还在内存中，等待垃圾回收。

![js-closures](/assets/images/posts/js/js-closures03.png){:height="100%" width="100%"}

在执行完上面的代码以后，可以看到```foo``` ```bar```的词法环境访问链路断掉了，虽然它们还在内存了（红色区域1和2），但是我们再也没办法访问这两个词法环境里的变量。


这时候如果还想访问```foo``` ```bar```的词法环境，比如还想用a的值，我们把代码改一下：

```js
function foo() {
    var a = 2;

    function bar() {
        console.log(a);
    }

    return bar;
}

var baz = foo();

baz();
```

运行```baz()```会输出2（2是foo词法环境里的值），也就是变量a在foo词法环境之外被访问了，这就是**闭包**。


正常情况下，在方法foo执行完以后，foo的执行上下文被弹出执行栈，它的词法环境链路也就失联了。我们知道每个方法在执行的时候都会创建一个新的执行上下文，同时也会创建它们自己的词法环境，每个方法的词法环境里有一个```scope```会保存（指向）它上一层的词法环境。 那么foo方法执行完以后返回bar，这个bar的scope里还保留着整个foo方法的词法环境，那么在执行```baz()```的时候也就是执行```bar()```，这样就可以访问失联的```foo方法的词法作用域```，也就是可以拿到变量```a```的值。


**闭包**就是指：执行完的```执行上下文```被弹出执行栈，它的词法环境处于失联状态，后续的执行上下文没办法直接访问这个失联的词法环境。在这种情况下还保留了对那个词法环境的```引用```，从而可以通过这个```引用```去访问失联的词法环境，这个```引用```就是闭包。


其实我们每天写的代码，基本会用到闭包，JS也有很多闭包的应用有以下几种方式：

- 第一种：
     ```js
        function foo() {
        var a = 2;

        function bar() {
            console.log( a );
        }

        return bar;
        }

        var baz = foo();

        baz(); // closure!
     ```

- 第二种：
     ```js
        function foo() {
        var a = 2;

        function baz() {
            console.log( a ); // 2
        }

        bar( baz );
        }

        function bar(fn) {
            fn(); // closure!
        }
     ```

- 第三种：
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
            fn(); // closure!
        }

        foo();

        bar(); // 2
     ```

- 第四种：
  ```js
    function wait(message) {

        setTimeout( function timer(){
            console.log( message );
        }, 1000 );

    }

    wait( "Hello, closure!" ); // 打印出 hello closure！ 回调函数的message是wait方法作用域的值。
  ```

  需要注意的是，如果代码写成这样：

    ```js
    function wait(message) {

        setTimeout( function timer(){
            console.log( this.message );
        }, 1000 );

    }

    wait( "Hello, closure!" ); // 打印出undefined, 回调函数的this值的是全局变量，全局变量没有这个值，所以是undefined。
   ```
   
 - 第五种是模块化