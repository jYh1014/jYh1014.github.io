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
### 链表和数组
- 相同点：都属于线性表（包括顺序线性表和链表）
- 不同点：1.数组静态分配内存，链表动态分配内存；2.数组在内存中连续，链表不连续；3.数组元素在栈区，链表元素在堆区；4.数组利用下标定位，时间复杂度为O(1)，链表定位元素时间复杂度O(n)； 5.数组插入或删除元素的时间复杂度O(n)，链表的时间复杂度O(1)。

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
## 集合：set （交集、并集、差集）

```sh
class Set{
    constructor(){
        this.set = {}
    }
    add(ele){
        if(!this.set.hasOwnProperty(ele)){
            this.set[ele] = ele
        }
    }
   
}
let set = new Set() //set特点是key 和 value相同
set.add(1)
```
## 哈希表（松散）如果key重复的话可以再加上链表

```sh
class Map{
    constructor(){
        this.arr = {}
    }
    calc(key) {
        let total = 0
        for (let i = 0; i < key.length; i++) {
            total += key[i].charCodeAt()
        }
        return total % 100
    }
    set(key, value) {
        key = this.calc(key)
        this.arr[key] = value
    }
    get(key) {
        key = this.calc(key)
        return this.arr[key]
    }
}
let map = new Map()
map.set("abc", 123)
console.log(map.arr)
```
## 二叉树

```sh
class Node{
    constructor(ele){
        this.element = ele
        this.left = null
        this.right = null
    }
}
class Tree{
    constructor(){
        this.root = null
    }
    add(ele){
        let node = new Node(ele)
        if(!this.root){
            this.root = node
        }else{
            this.insert(this.root, node)
        }
    }
    insert(root, newNode){
        if(newNode.element < root.element){
            if(root.left == null){
                root.left = newNode
            }else{
                this.insert(root.left, newNode)
            }
        }else {
            if(root.right == null){
                root.right = newNode
            }else{
                this.insert(root.right, newNode)
            }
        }
    }
}
let tree = new Tree()
tree.add(100)
tree.add(60)
tree.add(150)
tree.add(50)
tree.add(120)
tree.add(200)
console.log(tree)
```

