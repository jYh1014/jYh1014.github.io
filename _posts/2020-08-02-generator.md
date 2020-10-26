---
title: generator
author: jyy
date: 2020-08-02 14:20:00 +0800
tags: [js]
pin: true
---

## iterator遍历器

>es6上是这样定义的：它是一种接口，为各种不同的数据结构提供统一的访问机制。任何数据结构只要部署 Iterator 接口，就可以完成遍历操作

```sh
模拟一个遍历器生存函数
function makeIterator(arr){
    let index = 0
    return {
        next: function(){
            if(index < arr.length){
                return {
                    value: arr[index++],
                    done: false
                }
            }else{
                return {
                    value: undefined,
                    done: true
                }
            }
        }
    }
}
let it = makeIterator(["a","b"])
console.log(it.next())
console.log(it.next())
console.log(it.next())
```

ES6 规定，默认的 Iterator 接口部署在数据结构的Symbol.iterator属性，或者说，一个数据结构只要具有Symbol.iterator属性，就可以认为是“可遍历的”.  
原生具备 Iterator 接口的数据结构如下:
- Array
- Map
- Set
- String
- TypedArray
- 函数的 arguments 对象
- NodeList 对象

下面是为一个对象添加iterator接口的例子
```sh
let obj = {
  data: [ 'hello', 'world' ],
  [Symbol.iterator]() {
    const self = this;
    let index = 0;
    return {
      next() {
        if (index < self.data.length) {
          return {
            value: self.data[index++],
            done: false
          };
        } else {
          return { value: undefined, done: true };
        }
      }
    };
  }
};
for(let item of obj){
    console.log(item)
}
```

## generator函数

> 在了解了回调函数和promise解决异步编程的不足之后，generator函数可以解决这些问题。
> 还有一种处理异步编程的方案叫做协程：意思是多个线程互相协作，完成异步任务。  

它的运行流程大致如下：
- 第一步，协程A开始执行
- 第二步，协程A执行到一半，进入暂停，执行权转移到协程B
- 第三步，（一段时间后）协程B交还执行权
- 第四步，协程A恢复执行

上面流程的协程A，就是异步任务，因为它分成两段（或多段）执行。
Generator 函数是协程在 ES6 的实现，最大特点就是可以交出函数的执行权（即暂停执行）,yield命令是异步两个阶段的分界线。

下面看一个用Generator 函数来执行异步任务的例子：
```sh
let fetch = require('node-fetch');

function* gen(){
  let url = 'https://api.github.com/users/github';
  let result = yield fetch(url);
  console.log(result.bio);
}
let g = gen();
let result = g.next();

result.value.then(function(data){
  return data.json();
}).then(function(data){
  g.next(data);
});
可以看出虽然 Generator 函数将异步操作表示得很简洁，但是流程管理却不方便。
```
es6文档上指出thunk函数可以用于 Generator 函数的自动流程管理。

## Thunk函数

> 编译器的“传名调用”实现，是将参数放到一个临时函数之中，再将这个临时函数传入函数体。这个临时函数就叫做 Thunk 函数

```sh
function f(m) {
  return m * 2;
}
f(x + 5);

// 等同于

let thunk = function () {
  return x + 5;
};
function f(thunk) {
  return thunk() * 2;
}

```
> JavaScript 语言是传值调用,Thunk 函数替换的不是表达式，而是多参数函数，将其替换成一个只接受回调函数作为参数的单参数函数。
任何函数，只要参数有回调函数，就能写成 Thunk 函数的形式

```sh
// 正常版本的readFile（多参数版本）
fs.readFile(fileName, callback);

// Thunk版本的readFile（单参数版本）
let Thunk = function (fileName) {
  return function (callback) {
    return fs.readFile(fileName, callback);
  };
};

let readFileThunk = Thunk(fileName);
readFileThunk(callback);

```
## Thunkify 模块
```sh
//使用方式：以读取文件为例
let thunkify = require('thunkify');
let fs = require('fs');
let read = thunkify(fs.readFile);
read('package.json')(function(err, str){
  // ...
});
//源码实现：
function thunkify(fn) {
  return function() {
    let args = new Array(arguments.length);
    let ctx = this;

    for (let i = 0; i < args.length; ++i) {
      args[i] = arguments[i];
    }

    return function (done) {
      let called;
      args.push(function () {
        if (called) return;
        called = true;
        done.apply(null, arguments);
      });

      try {
        fn.apply(ctx, args);
      } catch (err) {
        done(err);
      }
    }
  }
};
```
## Generator 函数的流程管理
> Thunk 函数现在可以用于 Generator 函数的自动流程管理

```sh
let fs = require('fs');
let thunkify = require('thunkify');
let readFileThunk = thunkify(fs.readFile);

let gen = function* (){
  let r1 = yield readFileThunk('/etc/fstab');
  console.log(r1.toString());
  let r2 = yield readFileThunk('/etc/shells');
  console.log(r2.toString());
};
//先看如何手动执行上面这个 Generator 函数
let g = gen();
let r1 = g.next();
r1.value(function (err, data) {
  if (err) throw err;
  let r2 = g.next(data);
  r2.value(function (err, data) {
    if (err) throw err;
    g.next(data);
  });
});
//可以发现 Generator 函数的执行过程，其实是将同一个回调函数，反复传入next方法的value属性。
这使得我们可以用递归来自动完成这个过程
```

## Thunk 函数的自动流程管理

```sh
//基于 Thunk 函数的 Generator 执行器
function run(fn) {
  let gen = fn();
//next就是thunk函数的回调函数
  function next(err, data) {
    let result = gen.next(data);
    if (result.done) return;
    result.value(next);
  }

  next();
}

function* g() {
  // ...
}

run(g);

//前提是每一个异步操作，都要是 Thunk 函数，也就是说，跟在yield命令后面的必须是 Thunk 函数
let g = function* (){
  let f1 = yield readFileThunk('fileA');
  let f2 = yield readFileThunk('fileB');
  // ...
  let fn = yield readFileThunk('fileN');
};

run(g);
```
## co模块

> 用于 Generator 函数的自动执行，co模块约定，yield命令后面只能是 Thunk 函数或 Promise 对象

```sh
const fs = require("fs").promises

function* read() {
    let name = yield fs.readFile("name.txt", "utf8")
    let age = yield fs.readFile(name, "utf8")
    let xx = yield { a: age + 20 }
    return xx
}
co(read).then(data => {
    console.log(data)
})
//源码实现如下：
function co(gen) {
    let self = this
    return new Promise((resolve, reject) => {
        if (typeof gen === "function") {
            gen = gen.call(self)
        }
        function next(data) {
            let { value, done } = gen.next(data)
            if (done) {
                resolve(value)
            } else {
                Promise.resolve(value).then(data => {
                    console.log(data)
                    next(data)
                }, err => console.log(err))
            }

        }
        next()
    })
}
```