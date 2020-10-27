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
## 链表（单向）
```sh
class LinkList{
    constructor(){
        this.head = null
        this.length = 0
    }
    append(ele){
        let node = new Node(ele)
        if(!this.head){
            this.head = node
        }else{
            let index = 0 //先把链表的头部拿出来，然后从1开始查找
            let current = this.head
            while(++index < this.length){
                current = current.next  //如果当前不是最后一项，就从当前节点的下一项继续查找
            }
            current.next = node
            this.length++
        }
    }
    insert(position,ele){
        let node = new Node(ele)
        if(!this.head){
            this.head = node
        }else{
            let index = 0
            let current = this.head
            let previous = null
            while(index++ < position){
                previous = current
                current = current.next
            }
            previous.next = node
            node.next = current
        }
    }
}
class Node{ //链表的一个节点
    constructor(ele){
        this.element = ele
        this.next = null
    }
}
let ll = new LinkList()
ll.append(1) //末尾增加
ll.append(2)
ll.append(3)
ll.insert(1,100) //插入
console.log(ll)
```




