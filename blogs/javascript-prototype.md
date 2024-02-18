---
title: 详解原型与原型链
date: 2022-03-16 18:05:54
categories: 前端
tags:
  - JavaScript
---

# 详解原型与原型链

其实，刚开始学 JavaScript 时，就有学过原型与原型链的相关知识了，只是当时还没有养成写笔记的习惯，导致现在已经忘的七七八八了。

这边文章真是花了很多心思，写了两天，看了很多篇篇博文，其中有小参考的，有解决一点疑惑的，但是最后只标注了一篇帮助最大的。

<br>

## 构造函数

**实例的构造函数属性(` constructor`)指向其构造函数**

```js
function Person(name) {
  this.name = name;
}

const person = new Person("clz");
console.log(person.constructor === Person); // true
```

<br>

<b style="color: red">实例的构造函数并不是自身属性，而是从原型对象上继承的属性</b>

```js
function Person(name) {
  this.name = name;
}

const person = new Person("clz");
console.log(person.constructor === Person); // true
console.log(Person.prototype.constructor === Person); // true

console.log(person.hasOwnProperty("constructor")); // false：constructor属性并不是实例自身的属性，而是继承来的
```

<br>

## 原型对象

- **`__proto__(隐式原型)`：每个对象(除了` null`)都具有的属性**，该属性指向该对象的原型
- **`prototype(显式原型)`：只有函数对象才有的属性**，该属性指向函数的原型对象

来看来看

```js
const arr = [1, 2, 3];
const obj = {
  name: "clz",
};
function add(a, b) {
  return a + b;
}

console.log(arr);
console.log(obj);
console.log(add);
```

![image-20220314161614197](https://s2.loli.net/2022/03/14/8GRutdoLNb27qAz.png)

<br>

<b syule="color: red">红框框中的` [[prototype]]`和` __proto__`意义相同，都是指对象的内部属性</b>

**而所有函数都拥有` prototype`属性**，所以可以通过` f.prototype`得到，那么自然也不需要通过` [[prototype]]`显示出来(毕竟` prototype`是显式原型，而` __proto__`是隐式原型，好吧，这是我猜的)

![image-20220314162705526](https://s2.loli.net/2022/03/14/l1ebmHdsivJWNoT.png)

<br>

<b style="color: red">箭头函数没有` prototype`属性</b>

<br>

### 访问原型

通过实例对象访问原型对象有 3 种方法

- `obj.__proto__`
- `obj.constructor.prototype`
- `Object.getPrototypeOf(obj)`

```js
function Person(name) {
  this.name = name;
}

const person = new Person("clz");

const proto1 = person.__proto__;
const proto2 = person.constructor.prototype;
const proto3 = Object.getPrototypeOf(person);

const proto = Person.prototype; // 原型

console.log(proto1 === proto); // true: 第一种方法
console.log(proto2 === proto); // true: 第二种方法
console.log(proto3 === proto); // true: 第三种方法
```

<br>

**比较安全的做法是`Object.getPrototypeOf(obj)`**

**以下部分会涉及一丢丢原型链的知识(如果没看懂，可以看下原型链再来看)**

- ` __proto__`属性是私有属性，存在浏览器兼容性问题，缺乏非浏览器环境的支持

- 如果 obj 的` constructor`属性被覆盖，那么`obj.constructor.prototype`将会失效。(因为 obj 自身是没有` constructor`属性的，是通过原型链去它的原型上获取` constructor`属性，所以覆盖该属性时，将不会再去原型链上查找)

  ```js
  function Person(name) {
    this.name = name;
  }

  function Temp(name) {
    this.name = name;
  }

  const person = new Person("clz");

  person.constructor = Temp;

  const proto = Person.prototype; // 原型

  console.log(person.__proto__ === proto); // true: 第一种方法
  console.log(person.constructor.prototype === proto); // false: 第二种方法
  console.log(Object.getPrototypeOf(person) === proto); // true: 第三种方法
  ```

<br>

### 设置原型

设置原型对象有 3 种方法

- `obj.__proto__=prototypeObj`
- `Object.setPrototypeOf(obj, prototypeObj)`
- `Object.create(prototypeObj)`

```js
const proto = {
  // 原型对象
  name: "prototype",
};

// 第一种方法
const obj1 = {};
obj1.__proto__ = proto; // 设置原型
console.log(obj1.name); // prototype
console.log(obj1.__proto__ === proto); // true

// 第二种方法
const obj2 = {};
Object.setPrototypeOf(obj2, proto); // 设置原型
console.log(obj2.name); // prototype
console.log(obj2.__proto__ === proto); // true

// 第三种方法
const obj3 = Object.create(proto); // 创建对象并设置原型
console.log(obj3.name); // prototype
console.log(obj3.__proto__ === proto); // true
```

<br>

### 检测原型

使用` obj1.isPrototypeOf(obj2)`方法判断` obj1`是否为·` obj2`的原型

```js
const proto = {
  // 原型对象
  name: "prototype",
};

const proto1 = {
  name: "prototype",
};

const obj = {};
obj.__proto__ = proto; // 设置原型

console.log(proto.isPrototypeOf(obj)); // true
console.log(Object.prototype.isPrototypeOf(obj)); // true

console.log(proto1.isPrototypeOf(obj)); // false
```

<br>

## prototype、`__proto__`、constructor 之间的关系

![image-20220314170210850](https://s2.loli.net/2022/03/14/FO6IgxSiPm9pVWu.png)

```js
function Person(name) {
  this.name = name;
}

const person = new Person("clz");

console.log(person.__proto__ === Person.prototype); // true：因为创建person对象的构造函数是Person，所以person对象的隐式原型(__proto__)指向Person函数的原型(prototype)
console.log(Person.prototype.constructor === Person); // true
```

<br>

**同一个构造函数创建的多个实例的原型是同一个**

```js
function Person(name) {
  this.name = name;
}

const person1 = new Person("clz");
const person2 = new Person("clz");

console.log(person1 === person2); // false: 不是同一个对象
console.log(person1.__proto__ === person2.__proto__); // true：同一个构造函数创建的实例对象的原型是同一个
```

<br>

## 原型链

由上面的知识可以知道，实例对象具有属性` __proto__`，会指向原型对象。而原型对象也是对象，所以也会有属性` __proto__`，也会继续指向它的原型对象。

<b style="color: red">实例对象在查找属性时，如果查找不到，就会沿着` __proto__`去它的原型上查找，还找不到，则继续去原型的原型上查找，直到找到或到最顶层为止。这就是原型链的概念。</b>

<br>

**对象本身的方法(第一层：把方法当成属性)**

```js
function Person(name) {
  this.name = name;
  this.listenMusic = function () {
    console.log("听音乐");
  };
}
const person = new Person("clz");
console.log(person);
console.log(
  "实例对象本身是否有listenMusic方法",
  person.hasOwnProperty("listenMusic")
);
person.listenMusic();
```

![image-20220315094508947](https://s2.loli.net/2022/03/15/wVuWlXUHCbSGhx9.png)

<br>

**对象的原型上添加方法(第二层)**

```js
function Person(name) {
  this.name = name;
}
Person.prototype.listenMusic = function () {
  console.log("听音乐");
};

const person = new Person("clz");
console.log(person);
console.log(
  "实例对象本身是否有listenMusic方法",
  person.hasOwnProperty("listenMusic")
);
person.listenMusic();
```

![image-20220314183755250](https://s2.loli.net/2022/03/14/5bNeTitq3vJIU16.png)

<br>

**原型的原型上的方法(第三层)**

```js
function Person(name) {
  this.name = name;
}
Person.prototype.__proto__.listenMusic = function () {
  console.log("听音乐");
};

const person = new Person("clz");
console.log(person);
person.listenMusic();
```

![image-20220314184300125](https://s2.loli.net/2022/03/14/WwhLDopOEvJuxcG.png)

<br>

但是呢，没法玩第四层，因为已经到顶了(**` Object.prototype`没有原型(原型为 null)**)

```js
function Person(name) {
  this.name = name;
}
Person.prototype.__proto__.__proto__.listenMusic = function () {
  console.log("听音乐");
};

const person = new Person("clz");
console.log(person);
person.listenMusic();
```

![image-20220314184424877](https://s2.loli.net/2022/03/14/xD67UynCkVE4waz.png)

---

**person -> Person.prototype -> Object.prototype -> null**

那么，这里就来看看第三层是不是真的是` Object.prototype`

```js
function Person(name) {
  this.name = name;
}
Person.prototype.__proto__.listenMusic = function () {
  console.log("听音乐");
};

const person = new Person("clz");
console.log(person);

console.log(Person.prototype.__proto__ === person.__proto__.__proto__);
console.log(person.__proto__.__proto__ === Object.prototype); // 这里就是判断处

person.listenMusic();
```

![image-20220315095211650](https://s2.loli.net/2022/03/15/Vq8hgzA6YsF5KLm.png)

发现，确实如此。

**下面这张图就是原型链的简单图**(找不到是在哪里截的图了，侵删)

![image-20220314233252631](https://s2.loli.net/2022/03/14/gvmpCyaXMPxNiZ1.png)

<br>

### 原型链的作用

#### 为对象设置默认值

> 利用原型为对象设置默认值。当原型属性与私有属性同名时，删除私有属性之后，可以访问原型属性，即可以把原型属性值作为初始化默认值。

```js
function Person(name) {
  this.name = name;
}

Person.prototype.name = "赤蓝紫";

const person = new Person("clz");
console.log(person.name); // clz

delete person.name;
console.log(person.name); // 赤蓝紫
```

<br>

#### 继承

继承内容部分之后单独写。

<br>

#### 扩展原型方法

```js
Array.prototype.test = function () {
  console.log("扩展原型方法: 有风险");
};

const arr = [1, 2, 3];
arr.test(); // 扩展原型方法: 有风险
```

<br>

#### typeof

typeof 是判断类型时大多数人的选择(当然也包括我啦)，但是，判断非基本数据类型(`function`除外)时，只能得到` Object`。(null 也是，但是 null 这个属于是历史遗留 bug 了)。

> **js 在底层存储变量的时候，会在变量的机器码的低位 1-3 位存储其类型信息**
>
> - 000：对象
> - 010：浮点数
> - 100：字符串
> - 110：布尔
> - 1：整数
>
> null：所有机器码均为 0
> undefined：用 −2^30 整数来表示

---

` symbol`和` bigint`是后来新增的数据类型

```js
const num = 123;
const str = "hello";
const bool = true;
const n = null;
const u = undefined;
const sym = Symbol(1);
const big = BigInt(123);

const fun = () => console.log(1);

console.log(typeof num); // number
console.log(typeof str); // string
console.log(typeof bool); // boolean
console.log(typeof n); // object
console.log(typeof u); // undefined
console.log(typeof sym); // symbol
console.log(typeof big); //bigint

console.log(typeof fun); //function
```

<br>

**`function`除外的非基本数据类型**

```js
let arr = [];
let obj = {
  name: "clz",
};
let set = new Set([1, 2, 4]);

console.log(typeof arr);
console.log(typeof obj);
console.log(typeof set);
```

**清一色` object`**

<br>

通过`Object.prototype.toString.call(obj)`来识别对象类型。会返回`"[object Type]"`来告诉你所指对象的类型

```JS
let arr = []
let obj = {
  name: 'clz'
}
let set = new Set([1, 2, 4])

console.log(Object.prototype.toString.call(arr))  // [object Array]
console.log(Object.prototype.toString.call(obj))  // [object Object]
console.log(Object.prototype.toString.call(set))  // [object Set]
```

---

#### instanceof

**`instanceof`只要右边变量的 prototype 在左边变量的原型链上，就会返回`true`**

```js
function Person(name) {
  this.name = name;
}

function Test(name) {
  this.name = name;
}

const person = new Person("clz");

console.log(person instanceof Person); // true
console.log(person instanceof Object); // true

console.log(person instanceof Test); // false
```

<br>

## 普通对象与函数对象

- **所有的函数都是通过` new Function()`来创建的，即是函数对象**

- **其他的都是普通对象 **

<br>

```js
function fn1() {}
const fn2 = function () {};
const fn3 = () => {};
const fn4 = new Function();

console.log(typeof fn1); //function
console.log(typeof fn2); //function
console.log(typeof fn3); //function
console.log(typeof fn4); //function

const obj1 = {};
const obj2 = new Object();
const obj3 = new fn1();

console.log(typeof obj1); //object
console.log(typeof obj2); //object
console.log(typeof obj3); //object
```

<br>

上面的例子中，` fn1`、` fn2`、` fn3`、` fn4`是函数对象，` obj1`、` obj2`、` obj3`是普通对象

> - **Object 是构造函数，即也是函数，所以` Object`也是函数对象，相当于`Function`的实例，即` Object.__proto__ === Function.prototype`**
> - **`Object.prototype`是`Object`构造函数的原型，处于原型链的顶端，`Object.prototype.__proto__`已经没有可以指向的上层原型，因此其值为`null`**

```js
console.log(typeof Object); // function
console.log(Object.__proto__ === Function.prototype); // true
console.log(Object.prototype.__proto__); // null
```

<br>

> - **` Function.prototype`是` Function`的原型，是所有函数对象的原型**
> - **`Function.prototype`是一个普通对象，所以` Function.prototype.__proto__ === Object.prototype`**
> - **`Function`函数不通过任何东西创建，`JS`引擎启动时，添加到内存中**，所以**` Function.__proto__ === Function.prototype`**

```js
console.log(typeof Function); // function
console.log(Function.prototype.__proto__ === Object.prototype); // true
console.log(Function.__proto__ === Function.prototype); // true
```

<br>

## 经典原型链图

![img](https://s2.loli.net/2022/03/15/HXe9vi2ADnSVC6J.png)

<br>

## 练手福利

题目来自[JavaScript 之彻底理解原型与原型链](https://juejin.cn/post/7018355953955241997)

有加一道

```js
function User() {}
User.prototype.sayHello = function () {};
var u1 = new User();
var u2 = new User();

console.log(u1.sayHello === u2.sayHello);
console.log(User.prototype.constructor);

console.log(User.prototype === Function.prototype);
console.log(User.prototype.__proto__ === Function.prototype.__proto__);
console.log(User.prototype === u1.__proto__);

console.log(User.__proto__ === Function.prototype);
console.log(User.__proto__ === Function.__proto__);

console.log(u1.__proto__ === u2.__proto__);
console.log(u1.__proto__ === User.__proto__);

console.log(Function.__proto__ === Object.__proto__);
console.log(Function.prototype.__proto__ === Object.prototype.__proto__);
console.log(Function.prototype.__proto__ === Object.prototype);
```

<br>

**校对答案**：(可能有点难，细嚼慢咽后再反复看就行)

![image-20220315162828873](https://s2.loli.net/2022/03/15/h6jEpTSG4RaIfoQ.png)

<br>

参考链接：[JS 原型（prototype）和原型链完全攻略](http://c.biancheng.net/view/5805.html)
