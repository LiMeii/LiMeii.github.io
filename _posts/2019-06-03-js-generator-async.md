---
title: JS：回调函数 Promise Generator Async异步处理应用
tags: JS
layout: post
---

JavaScript是单线程的，正是因为有异步处理，JS才不会卡顿，JS的异步处理方式有：


**回调函数 -> Promise -> Generator -> Async**


之前一直很困惑，为什么JS异步处理方式有这么多种，Promise能解决大部分实际开发工作中的异步处理。看完Generator和Async的具体用法之后，才恍然大悟，从回调函数到Promise到Generator再到Async，其实也是想让异步代码可读性要好一点，异步代码写得越来越像同步代码，这应该是异步处理的终极目标吧。


接下来就分别用这四种方式来实现同一个异步调用，看看这四种方式实现异步处理各有什么优劣。在示例代码里会用API：【[Github Search user API](https://developer.github.com/v3/search/#search-users)】，先根据关键字搜索Github用户，然后根据返回的搜索结果得到搜索结果的数量。

## 回调函数
```js
const https = require('https');

const cbGetGitUser = function (name) {
    https.get('https://api.github.com/search/users?q=' + name, (resp) => {
        let data = "";
        resp.on('data', (chunk) => {
            data += chunk;
        });
        resp.on('end', () => {
            console.log('here is the cb1 result ' + data);
            cbCountGitUser(data);
        });
    }).on('error', (err) => {
        console.log("Error" + err.message);
    })
}

const cbCountGitUser = function (data) {
    let response;
    if (typeof data === 'object') {
        response = data;
    } else if (typeof data === 'string') {
        response = JSON.parse(data);
    }
    console.log('here is the cb2 result ' + response.total_count)
}

cbGetGitUser("limeii");
```
在```cbGetGitUser```里先调用API，等拿到用户搜索结果之后，再调用回调函数```cbCountGitUser```计算搜索结果的用户数量。现在是只有一个回调函数，代码结构看起来还可以，假如回调函数特别多，加上中间还需要调用其他方法，就会有```callback hell```问题，整个代码结构横向发展，代码可读性差，而且很容易出错。

## Promise
```js
const https = require('https');
const getGitUser = function (name) {
    return new Promise(function (resolve, reject) {
        https.get('https://api.github.com/search/users?q=' + name, (resp) => {
            let data = "";
            resp.on('data', (chunk) => {
                data += chunk;
            });
            resp.on('end', () => {
                resolve(data);
            });
        }).on('error', (err) => {
            console.log("Error" + err.message);
            reject(err.message);
        })
    });
}

const countGitUser = function (data) {
    let response;
    if (typeof data === 'object') {
        response = data;
    } else if (typeof data === 'string') {
        response = JSON.parse(data);
    }

    return new Promise(function (resolve, reject) {
        if (response) {
            resolve(response.total_count);
        } else {
            console.log('here is the data: ' + response);
            reject('cannot get the user information!!!');
        }
    });
}


getGitUser('limeii').then(data => {
    console.log('the first promise ' + data);
    return data;
}).then(res => {
    return countGitUser(res);
}).then(data => {
    console.log('the second promise ' + data);
}).catch(error => {
    console.log('has error: ' + error);
});
```
在回调函数之后，JS引入了Promise，相对回调函数横向代码结构，Promise的代码结构是纵向发展的，每一个回调都是放在then里执行，代码可读性要好不少，不会有```callback hell```。但是由于Promise是一次性的，而且一旦创建Promise就不能取消也不能暂停，只能等待成功或者失败的结果，没办法在Promise的中间加入业务逻辑处理。

## Generator
下面示例代码中的```getGitUser```和```countGitUser```方法，共用Promise示例代码中的方法。

```js
function* gen(value) {
    var result1 = yield getGitUser(value);
    yield countGitUser(result1);
}

var g = gen('limeii');
g.next().value.then(data => {
    console.log('the first yeild: ' + data);
    return data;
}).then(res => {
    return g.next(res).value;
}).then((data) => {
    console.log('the second yeild: ' + data);
}).catch((error) => {
    console.log('has error: ' + error);
});
```
在ES6引入了Generator，Generator相对Promise来说，通过```yield```关键字可以暂停异步代码，通过```next```方法重新恢复异步代码执行。从上面的示例代码可以看到执行两个```yield```，Generator的执行代码里有四个then，如果有很多个```yield```，Generator的执行代码里会有一大串then，执行代码看起来很冗余。

TJ Holowaychuk写过一个工具【[co](https://github.com/tj/co)】，用于Generator的自执行。使用这个工具，上面那一串的执行代码，只用一行就可以了：
```js
function* gen_co(value) {
    var result1 = yield getGitUser(value);
    console.log('the first co ' + result1);
    var result2 = yield countGitUser(result1);
    console.log('the second co ' + result2);
}
co(gen_co('limeii'));
```
co还是有一定的局限性，在```yield```命令后面只能是：
- promises
- thunks (functions)
- array (parallel execution)
- objects (parallel execution)
- generators (delegation)
- generator functions (delegation)


需要注意的是执行```yield```是没有返回值的，也就是每次返回值是```undefined```，在```next```方法里可以传入参数，作为上一次```yield```的返回结果。关于Generator的具体语法可以参考阮一峰的【[Generator 函数的语法](http://es6.ruanyifeng.com/#docs/generator)】。

## Async
```js
async function asyncFuc() {
    var value1 = await getGitUser("limeii");
    var value2 = await countGitUser(value1);

    console.log('the first await value ' + value1);
    console.log('the second await value ' + value2);
}
asyncFuc();
```
之前提到过Generator的执行代码非常长，虽然有【[co](https://github.com/tj/co)】可以简化Generator的执行代码，但是co可以支持的对象有限。在ES7引入了Async，它其实就是Generator的语法糖，同样也简化了Generator的自执行。相对于co来说，Async的Await命令后面可以是Promise 对象和原始类型的值（数值、字符串和布尔值，但这时会自动转成立即 resolved 的 Promise 对象）。而且Async的异步代码结构跟同步代码结构没有什么差异，可读性最好。

Async相对于Generator的语法来说，就是把星号（*）改为了```async```，```yield```改成了```await```，具体语法和用法可以参考阮一峰的【[async 函数](http://es6.ruanyifeng.com/#docs/async)】。


## 总结
在JS中，异步处理方式有回调函数、Promise、Generator、Async这四种方式。那么：

### async 会取代 Generator 吗？
Generator本来是用作生成器，使用Generator处理异步请求只是一个比较hack的用法，在异步方面，async可以取代Generator，但是async和Generator两个语法本身是用来解决不同的问题的。

### async 会取代 Promise 吗？
- async函数返回一个Promise对象
- 面对复杂的异步流程，Promise提供的all和race会更加好用
- Promise本身就是一个对象，在代码里可以任意传递
- async的支持率还是很低，即使有Babel，编译后也要增加一千行左右代码
