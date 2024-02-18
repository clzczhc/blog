---
title: 创建对象的几种方式(二)
categories: 前端
date: 2022-04-30 17:23:19
tags:
  - JavaScript
---

# 创建对象的几种方式(二)
> 看红宝书+查资料，重新梳理JavaScript的知识。

## 原型模式
原型的知识不会过多赘述，可以查看我之前写的文章，或者自己找资料。

每个函数都会创建一个` prototype `属性，它就是原型对象，在它上面定义的属性和方法可以被对象实例共享。所以在构造函数中赋值给对象的值，可以变成赋值给它们的原型。

```js
function Person() { }

Person.prototype.name = 'clz';
Person.prototype.age = 21;
Person.prototype.listen = function () {
    console.log('Egoist');
}


const person1 = new Person();
const person2 = new Person();

console.log(person1.name, person1.age)  // clz 21

console.log(person1.listen === person2.listen);      // true

person1.listen();        // Egoist
```

但是，这时候直接打印实例，如` person1`，会得到一个空对象和内部属性` [[Prototype]] `，因为它的属性和方法都在原型对象上。

### 手动设置原型

**可以通过` Object.setPrototypeOf `来手动设置原型**

```js
const person = {
    name: 'clz'
};

const father = {
    age: 21
};

Object.setPrototypeOf(person, father);

console.log(person.age)     // 21

console.log(Object.getPrototypeOf(person) === father)   // true
```

但是，使用` Object.setPrototypeOf `可能会严重影响代码性能，因为
> 修改继承关系的影响时微妙而深远的，它的影响会涉及**所有访问了那些修改过`[[Prototype]]`的对象的代码**。

可以通过` Object.create `来创建一个新对象，并同时为其指定原型。这样子可以避免使用` Object.setPrototypeOf `可能造成的性能下降。(因为此时的新对象还没有任何属性，修改它的原型对象造成的影响很明显会小一点)

```js
const father = {
    age: 21
};

const person = Object.create(father);
person.name = 'clz'

console.log(person)         // { name: 'clz' }
console.log(person.age)     // 21

console.log(Object.getPrototypeOf(person) === father)       // true
```

### hasOwnProperty
既然实例可以通过原型链访问到原型对象的属性，那么我们就需要知道什么属性是实例自身的，还是原型对象上的。这时候我们就可以使用` hasOwnProperty`来确定某个属性是在实例上还是在原型对象上。

```js
function Person() {
    this.name = 'clz'
}

Person.prototype.name = 'czh'
Person.prototype.age = 21

const person = new Person()

console.log(person.hasOwnProperty('name'))  // true。来自实例
console.log(person.hasOwnProperty('age'))  // false。来自原型

delete person.name  // 删除掉person.name后，原型上name的属性的联系就恢复了
console.log(person.name)        // czh
console.log(person.hasOwnProperty('name'))  // false。来自原型

delete person.age   // 无法通过删除实例上的属性去删除原型上的属性
console.log(person.age)  // 21
```

### 原型和in操作符
in操作符只存在于` for-in `循环？
不不不，in操作符不只是能在` for-in `循环中使用，还能单独使用。

单独使用时，` in `操作符在能通过对象访问指定属性时返回true，无论它是在实例上还是在原型上。

```js
function Person() {
    this.name = 'clz'
}

Person.prototype.name = 'czh'
Person.prototype.age = 21

const person = new Person()

console.log('name' in person)  // true
console.log('age' in person)  // true

console.log('job' in person)    // false。无法通过对象访问到属性才会为false
```



### 属性枚举顺序

> ` for-in `循环和` Object.keys() `的枚举顺序是不确定的，取决于JavaScript引擎，可能会因浏览器而异。

`Object.getOwnPropertyNames()`返回对象实例的常规属性数组，`Object.getOwnPropertySymbols()`返回对象实例的符号属性数组。

`Object.getOwnPropertyNames()`枚举顺序：
1. 按升序枚举数值键
2. 字面量中定义的键以出现顺序来枚举
3. 插入的键(不包括数值键)以插入顺序来枚举
4. 不会枚举符号键

```js
const k1 = Symbol('k1')
const k2 = Symbol('k2')

let o = {
    0: 0,
    [k1]: 'k1',
    c: 'c',
    1: 1,
    b: 'b',
    C: 'C',
    B: 'B',
}

o[k2] = 'k2'
o[2] = 2
o.a = 'a'
o.A = 'A'

console.log(Object.getOwnPropertyNames(o))   // ['0', '1', '2', 'c', 'b', 'C', 'B', 'a', 'A']

```



需要枚举符号属性的话，用` getOwnPropertySymbols `。符号属性的枚举比较简单暴力：谁先出现枚举谁。

```js
const k1 = Symbol('1')
const k2 = Symbol('2')

const k3 = Symbol('c')
const k4 = Symbol('a')

const k5 = Symbol('3')
const k6 = Symbol('b')

let o = {
    0: 0,
    [k3]: 'c',
    [k1]: '1',
    [k2]: '2',
    [k4]: 'a'
}

o[k5] = '3'
o[k6] = 'b'

console.log(Object.getOwnPropertySymbols(o))   // [ Symbol(c), Symbol(1), Symbol(2), Symbol(a), Symbol(3), Symbol(b) ]
```
