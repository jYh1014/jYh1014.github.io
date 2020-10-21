---
title: generator
author: jyy
date: 2020-10-20 14:20:00 +0800
# categories: [Blogging, Tutorial]
tags: [js]
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