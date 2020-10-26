---
title: 数据结构
author: jyy
date: 2020-07-20 15:20:00 +0800
# categories: [Blogging, Tutorial]
tags: [js]
pin: true
---

## 栈

> 特点：先进后出

```sh
class Stack{
    constructor(){
        this.stack = []
    }
    add(ele){
        this.stack.push(ele)
    }
    pop(){
        this.stack.pop()
    }
}
let stack = new Stack()
stack.add(1)
stack.add(2)
stack.pop()
console.log(stack)
```
## 队列

> 特点：先进先出

```sh
class Queue{
    constructor(){
        this.queue = []
    }
    enqueue(ele){
        this.queue.push(ele)
    }
    dequeue(){
        this.queue.shift()
    }
}
let queue = new Queue()
queue.enqueue(1)
queue.enqueue(2)
queue.dequeue()
console.log(queue)
```
## 链表




