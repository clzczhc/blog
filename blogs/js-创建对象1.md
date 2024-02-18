---
title: 创建对象的几种方式(一)
categories: 前端
date: 2022-04-30 17:22:39
tags:
  - JavaScript
---

# 创建对象的几种方式(一)
> 看红宝书+查资料，重新梳理JavaScript的知识。
## 工厂模式
首先需要一个函数(工厂)，然后在函数中创建具体对象。这种模式可以抽象创建具体对象的过程，这样子，我们想要创建对象，只需要调用函数，让属性值进厂即可。

```js
function createPerson(name, age) {
    const o = new Object()
    o.name = name;
    o.age = age;
    o.listen = function () {
        console.log('Egoist')
    };

    return o;
}

const person1 = createPerson('clz', 21);
const person2 = createPerson('czh', 111);

console.log(person1);   // {name: 'clz', age: 21, listen: ƒ}
console.log(person2);   // {name: 'czh', age: 111, listen: ƒ}

person1.listen();       // Egoist
```



## 构造函数模式

ECMAScript中的构造函数是用于创建特定类型对象的。所以我们可以通过自定义构造函数，以函数的形式来为对象定义属性和方法。

```js
function Person(name, age) {
    this.name = name;
    this.age = age;
    this.listen = function () {
        console.log('Egoist')
    };
}

const person1 = new Person('clz', 21);
const person2 = new Person('czh', 111);

console.log(person1);   // {name: 'clz', age: 21, listen: ƒ}
console.log(person2);   // {name: 'czh', age: 111, listen: ƒ}

person1.listen();       // Egoist
```

此时，我们不再需要通过原生构造函数` Object `来显示地创建对象，而是直接通过` this `关键字来实现，也不再需要把创建的对象` return `出去。但是，构造函数和普通函数的使用方式也不太一样，需要通过` new `操作符来` new `出一个对象。

**构造函数名称的首字母需要大写，非构造函数以小写字母开头**。不这样子虽然也可以，但是按照规范，才能更好地区分构造函数和普通函数。

构造函数和普通函数的使用方式不太一样，需要使用` new `操作符。

为什么需要这样子的方式调用构造函数呢？
因为以这样的方式调用函数会执行以下操作：
1. 在内存中创建一个新对象
2. 新对象的` __proto__`指向构造函数的原型` prototype `(具体可以查看之前的写的原型链文章)
3. 构造函数内部的`this`指向新对象
4. 执行构造函数内的代码
5. 如果构造函数返回非空对象，则返回该对象；否则返回在内存中创建的那个对象。

```js
function person(name, age) {
    this.name = name;
    this.age = age;
    this.listen = function () {
        console.log('Egoist')
    };

    return {
        name: 'clz',
        age: 999
    }
}

const person1 = new person('clz', 21);

console.log(person1);   // { name: 'clz', age: 999 }
```

### 构造函数也是函数
如果构造函数不使用` new `操作符来调用，就是普通函数。这时候，添加的属性、方法会被添加到全局对象上。

```js
function Person(name, age) {
    this.name = name;
    this.age = age;
    this.listen = function () {
        console.log('Egoist')
    };
}

const person1 = Person('clz', 21);

console.log(person1)            // undefined
console.log(globalThis.name);   // clz
console.log(globalThis.age);    // 21
```

上面使用了` globalThis `。` globalThis `是全局对象的标准名称。
因为在浏览器中，全局对象是window，而在Node.js中，全局对象是` global `。但是在浏览器中，访问` global`会报错，在Node.js中，访问` window`也会报错。使用` globalThis `就不会出现这个问题，简单来说的话，就是` globalThis `会变身，在浏览器时，会变成` window `，在Node.js时，会变成` global `


上面给对象添加方法等价于
```js
this.listen = new Function('console.log("Egoist")')
```

也就是说，不同实例上的函数同名但不相等。
```js
console.log(person1.listen === person2.listen)  // false
```

但是呢？都是干一样的事，却要定义两个不同的` Function `实例，这很明显很不合理，毕竟如果需要100个实例，那就需要100个` Function `实例。所以，应该先在外面声明函数，然后才把声明的函数赋给实例的方法。

```js
function Person(name, age) {
    this.name = name;
    this.age = age;
    this.listen = listen
}

function listen() {
    console.log("Egoist");
}


const person1 = new Person('clz', 21);
const person2 = new Person('clz', 111);

console.log(person1.listen === person2.listen)      // true

person1.listen()        // Egoist
```
