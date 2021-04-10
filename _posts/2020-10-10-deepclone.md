---
title: 深拷贝和浅拷贝
author: jyy
date: 2021-02-11 10:10:00 +0800
# categories: [Blogging, Tutorial]
tags: [js]
pin: true
---

## 浅拷贝
> 会创建一个新对象，该对象会完全复制原始对象，如果属性是基础类型，就复制的值，如果属性是引用类型，就复制的地址。
> Object.assign,Array.prototype.concat(),Array.prototype.slice()也都是浅拷贝。Array的slice和concat方法不修改原数组，只会返回一个浅复制了原数组中的元素的一个新数组

```js
function shallowClone(obj){
    var o = {}
    for(var key in obj){
        if(obj.hasOwnProperty(key)){
            o[key] = obj[key]
        }
    }
    return o
}
var obj1 = {
    name: "jyy",
    arr: [1,2,3]
}
var obj2 = shallowClone(obj1)
obj1.name = "abc"
obj1.arr[1] = 100
console.log(obj2) // {name: "jyy",arr: [1,100,3]}
```

```js
var arr1=[
    1,2,{name: "jyy"}
]
var arr2 = arr1.concat()
arr2[2].name = "jyy1"
console.log(arr1)
```

## 深拷贝
> 创建一个新对象，修改对象里面的任何属性都不会影响到原始对象

```js
 function deepClone(value, hash = new WeakMap){
    if(value == null) return value //null undefined
    if(typeof value !== "object") return value //这里包含了函数类型
    if(value instanceof RegExp) return new RegExp(value)
    if(value instanceof Date) return new Date(value)
    //...
    //拷贝的可能是一个对象或者数组 
    
    let instance = new value.constructor
    if(hash.has(value)){
        return hash.get(value)
    }
    hash.set(value, instance)
    for(let key in value){
        if(value.hasOwnProperty(key)){
            instance[key] = deepClone(value[key], hash)
        }
    }
    return instance
 }
var obj1 = {
    a: 1,
    b: {
        name: 'jyy'
    }
}
var obj2 = deepClone(obj1)
obj2.b.name = "jyy1"
console.log(obj1)
```