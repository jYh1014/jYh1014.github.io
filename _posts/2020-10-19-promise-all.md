---
title: promise对象的几个方法分析
author: jyy
date: 2020-10-19 10:55:00 +0800
tags: [promise]
pin: true
---


## promise.all

```sh
const p = Promise.all([p1, p2, p3]);
``` 

上面代码p的状态由p1、p2、p3决定，分成两种情况。

（1）只有p1、p2、p3的状态都变成fulfilled，p的状态才会变成fulfilled，此时p1、p2、p3的返回值组成一个数组，传递给p的回调函数。

（2）只要p1、p2、p3之中有一个被rejected，p的状态就变成rejected，此时第一个被reject的实例的返回值，会传递给p的回调函数。

```sh
const p2 = new Promise((resolve, reject) => {
    resolve("ok")
}).then(res => res).catch(e => console.log("p2:  " + e))
Promise.all([fs.readFile("./name.txt", "utf8"), fs.readFile("./age.txt", "utf8"), 1, p2]).then(data => {
    console.log(data)
})
先假设这个方法的参数数组成员都是promise实例，以下是源码实现
//判断数据是否是promise对象
function isPromise(value) {
    if (typeof value === "object" && value !== null || typeof value === "function") {
        return typeof value.then === "function"
    } else {
        return false
    }
}
Promise.all = (promises) => {
    return new Promise((resolve, reject) => {
        let arr = []
        let index = 0
        let processData = (i, data) => {
            arr[i] = data
            if (++index === promises.length) {
                resolve(arr)
            }
        }
        for (let i = 0; i < promises.length; i++) {
            let cur = promises[i]
            if (isPromise(cur)) {
                cur.then(y => {
                    processData(i, y)
                    // resolve(y)
                }, r => reject(r))
            } else {
                processData(i, cur)
            }
        }
    })
}
```

## promise.race

```
const p = Promise.race([p1, p2, p3]);
```
上面代码中，只要p1、p2、p3之中有一个实例率先改变状态，p的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给p的回调函数。

```sh
let p1 = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve("ok1")
    }, 1000)
})
let p2 = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve("ok2")
    }, 2000)
})
Promise.race([p1, p2]).then(data => {
    console.log(data)
})

先假设这个方法的参数数组成员都是promise实例，以下是源码实现
Promise.race = (promises) => {
  return new Promise((resolve, reject) => {
    for (let i = 0; i < promises.length; i++) {
      promises[i].then(resolve, reject)
    }
  })
}
```

## promise.finally

>用于指定不管 Promise 对象最后状态如何，都会执行的操作.本质上是then方法的特例
```sh
Promise.prototype.finally = function (callback) {
    //等待finally中的函数执行完毕，finally函数可能返回的是一个promise，用promise.resolve等待返回的promise执行完
    return this.then((value) => {
        Promise.resolve(callback()).then(() => value)
        // return value;
    }, err => {
        Promise.resolve(callback()).then(() => { throw err })
        // throw err;
    })
}
```

## promise.try

> 模拟try代码块

```sh
function fn(){
    //函数中抛出的同步错误要用try catch捕获异常
    // throw new Error("error")
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            reject("error123")
        },1000)
    })
}
Promise.try = function(callback){
    return new Promise((resolve, reject) => {
        //Promise.resolve只能返回一个成功的promise
        return Promise.resolve(callback()).then(resolve, reject)
    })
}
Promise.try(fn).catch(err => {
    //同步和异步异常都能捕获
    console.log(err)
})
```








