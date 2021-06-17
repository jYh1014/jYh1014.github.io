---
title: 原型和原型链
author: jyy
date: 2020-11-10 15:20:00 +0800
tags: [js]
pin: true
---

## **原型prototype**

> 每个函数都有一个prototype属性，指向一个对象，这个对象就是调用该构造函数得到的实例的原型对象。
> 原型：每一个js对象创建的时候都会与之关联另一个对象，这个对象就是原型对象，每一个对象都会从原型继承属性。

## **__proto__**

> 每一个对象（除了null）都有这个__proto__属性，指向该对象的原型对象。

```js
function Person() {

}
var person = new Person();
console.log(person.__proto__ === Person.prototype); // true
```

## **constructor**

> 每个原型对象都有一个construtor属性指向关联的构造函数。
> 为什么实例对象也有construtor属性？其本身并没有这个属性，它会沿着原型链从实例的原型对象上面去找constructor属性。

```js
function Person() {

}
var person = new Person();
console.log(person.constructor === Person.prototype.constructor === Person)
```

## **原型链**

> 当读取对象的属性或者方法，如果找不到，就会查找与之关联的原型对象上的属性，如果还查找不到，就去找原型的原型，一直到最顶层null为止。原型链是通过 **__proto__**属性链接起来的。


![diff](/assets/img/sample/prototype.jpeg)


## es5实现继承的方法

- 继承父类实例的属性。
- 继承父类原型上的属性，也就是实现链的关联。

```js
function Parent(){
    this.name = "parent"
}
function Child(){
    Parent.call(this) //实现父类实例属性的继承
    this.age = 20
}
Child.prototype = Object.create(Parent.prototype) //实现链的关联
//Object.setPrototypeOf(Child.prototype, Parent.prototype)

//Object.create原理如下
function Create(parentProto){
    function Temp(){
        this.constructor = Child
    }
    Temp.prototype = parentProto
    return new Temp()
}

//es6中extends继承不仅包括上面已经提到的继承，还包括父类静态属性的继承

Child.__proto__ = Parent
```