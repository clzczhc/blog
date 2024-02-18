---
title: JavaScript实现继承的六种方式
date: 2022-03-17 17:06:21
categories: 前端
tags:
  - JavaScript
---

# JavaScript 实现继承的六种方式

---

**父类**

```js
function Person(name) {
  this.name = name;
  this.say = function () {
    console.log("1 + 1 = 2");
  };
}

Person.prototype.listen = function () {
  console.log("エウテルペ");
};
```

---

## 1. 原型链继承

**将父类的实例作为子类的原型**

```js
function Person(name) {
  this.name = name;
  this.say = function () {
    console.log("1 + 1 = 2");
  };
}

Person.prototype.listen = function () {
  console.log("エウテルペ");
};

function Student() {}

Student.prototype = new Person(); //关键

const stu = new Student();

stu.grade = 3;

console.log(stu.grade); // 3

stu.say(); // 1 + 1 = 2
stu.listen(); // エウテルペ
```

<br>

**优点**：

- 简单易实现

- 父类新增原型方法/原型属性，子类都能访问

- 实例是子类的实例也是父类的实例

  ```js
  stu instanceof Student; // true
  stu instanceof Person; // true
  ```

<br>

**缺点**：

- 为子类新增属性和方法，不能在构造函数中
- 无法实现多继承
- 创建子类实例时，不能向父类构造函数传参数
- 所有新实例都会共享父类实例的属性。（原型上的属性是共享的，一个实例修改了原型属性，另一个实例的原型属性也会被修改！）

<br>

**存在的问题：**

- `prototype`里有个属性`constructor`指向构造函数本身，但是，` Student`的原型已经被父类的实例取代了，所以指向也不正确，所以需要修复构造函数指向(这里网上的教程只是对组合继承、寄生组合式继承进行了修复，不知道是不是因为这个不常用的关系)

  ```js
  function Student() {}
  console.log(Student.prototype.constructor);

  Student.prototype = new Person(); //关键

  const stu = new Student();

  stu.grade = 3;

  console.log(stu.grade); // 3

  stu.say(); // 1 + 1 = 2
  stu.listen(); // エウテルペ
  console.log(Student.prototype.constructor);
  ```

  ![image-20220317095122520](https://s2.loli.net/2022/03/17/5jp73YhObJIxtM2.png)

<br>

**解决问题**

```js
Student.prototype.constructor = Student;
```

![image-20220317095215698](https://s2.loli.net/2022/03/17/4eCavAGMltys9TK.png)

---

## 2. 借用构造函数继承

**在一个类中执行另一个类的构造函数，通过` call`函数设置` this`的指向，这样就可以得到另一个类的所有属性**

```js
function WebsiteMaster(site) {
  this.site = site;
}

function Student(name, grade, site) {
  Person.call(this, name);
  WebsiteMaster.call(this, site);
  this.grade = grade;
}

const stu = new Student("clz", 3, "https://clz.vercel.app");

console.log(stu.name, stu.grade, stu.site); // clz, 3, https://clz.vercel.app

stu.say(); // 1 + 1 = 2
stu.listen(); // Uncaught TypeError: stu.listen is not a function
```

<br>

**优点：**

- 创建子类实例时，可以向父类传递参数
- 可以实现多继承(call 多个对象)
- 不需要修复构造函数指向

<br>

**缺点：**

- 方法在构造函数中定义，无法复用

- 只能继承父类的实例属性，不能继承原型属性、方法

- 实例并不是父类的实例，而只是子类的实例

  ```js
  stu instanceof Student; // true
  stu instanceof Person; // false
  ```

  继承继着不再是人了(笑)

---

## 3. 原型式继承

为父类实例添加属性、方法，作为子类实例。

> 道格拉斯·克罗克福德在一篇文章中介绍了一种实现继承的方法，这种方法并没有使用严格意义上的构造函数。它的想法是借助原型可以基于已有的对象创建新对象，同时还不必因此创建自定义类型。为了达到这个目的，他给出了如下函数。
>
> ```js
> function object(o) {
>   function F() {}
>   F.prototype = o;
>   return new F();
> }
> ```

```js
function object(o) {
  function F() {}
  F.prototype = o;
  return new F();
}

const person = new Person("clz");

const stu = object(person);

stu.grade = 3;
stu.study = function () {
  console.log("FrontEnd");
};

console.log(stu.name, stu.grade); // clz, 3

stu.say(); // 1 + 1 = 2
stu.listen(); // エウテルペ

stu.study(); // FrontEnd
```

![image-20220316224057344](https://s2.loli.net/2022/03/17/HRrLnl6oKQhDiv7.png)

<br>

**优点**：

- 感觉没啥优点，不太像继承

<br>

**缺点：**

- 不支持多继承
- 实例是父类的实例

---

## 4. 寄生式继承

为父类实例添加属性、方法，作为子类实例。(原理和原型式继承一样)

```js
function object(o) {
  function F() {}
  F.prototype = o;
  return new F();
}

function Student(name, grade) {
  const person = object(new Person(name));

  person.grade = grade;
  person.study = function () {
    console.log("FrontEnd");
  };

  return person;
}

const stu = new Student("clz", 3);

console.log(stu.name, stu.grade); // clz, 3

stu.say(); // 1 + 1 = 2
stu.listen(); // エウテルペ

stu.study(); // FrontEnd
```

**优点**：

- 有了子类的雏形，但是换汤不换药，原理和原型式继承一样

<br>

**缺点：**

- 不支持多继承

- 实例是父类的实例，不是子类的实例(因为只是在父类的实例上添加属性、方法而已)

  ```js
  stu instanceof Student; // false
  stu instanceof Person; // true
  ```

---

## 5. 组合继承

原型链继承+借用构造函数继承

```js
function WebsiteMaster(site) {
  this.site = site;
}

function Student(name, grade, site) {
  Person.call(this, name); // 继承属性
  WebsiteMaster.call(this, site);
  this.grade = grade;
}

Student.prototype = new Person(); // 继承方法
Student.prototype.constructor = Student;

const stu = new Student("clz", 3, "https://clz.vercel.app");

console.log(stu.name, stu.grade, stu.site); // clz, 3, https://clz.vercel.app

stu.say(); // 1 + 1 = 2
stu.listen(); // エウテルペ

console.log(stu.constructor);
```

<br>

**优点**：

- 可以继承实例属性、方法，也可以继承原型属性、方法
- 可传参、可复用
- 实例既是子类的实例，也是父类的实例

<br>

**缺点**：

- 调用了两次父类构造函数，耗内存
- 需要修复构造函数指向

<br>

## 6. 寄生组合式继承

通过` Object.create()`来代替给子类原型赋值的过程，解决了两次调用父类构造函数的问题

```js
function WebsiteMaster(site) {
  this.site = site;
}

function Student(name, grade, site) {
  Person.call(this, name); // 继承属性
  WebsiteMaster.call(this, site);
  this.grade = grade;
}

Student.prototype = Object.create(Person.prototype); // 继承方法
Student.prototype.constructor = Student; // 修复构造函数指向

const stu = new Student("clz", 3, "https://clz.vercel.app");

console.log(stu.name, stu.grade, stu.site); // clz, 3, https://clz.vercel.app

stu.say(); // 1 + 1 = 2
stu.listen(); // エウテルペ
```

<br>

有人可能会提出：为什么不可以直接把父类原型赋值给子类原型来实现呢？

这是因为直接赋值的话，那就是引用关系。下面就来看看

```js
function WebsiteMaster(site) {
  this.site = site;
}

function Student(name, grade, site) {
  Person.call(this, name); // 继承属性
  WebsiteMaster.call(this, site);
  this.grade = grade;
}

Student.prototype = Person.prototype; // 继承方法
Student.prototype.constructor = Student; // 修复实例

const stu = new Student("clz", 3, "https://clz.vercel.app");

console.log(stu.name, stu.grade, stu.site); // clz, 3, https://clz.vercel.app

stu.say(); // 1 + 1 = 2
stu.listen(); // エウテルペ

Student.prototype.listen = function () {
  console.log("EGOIST");
};

console.log(Person.prototype.listen);
```

![image-20220316234707764](https://s2.loli.net/2022/03/17/tSLzqmKE1jRk4iJ.png)

可以看到，修改` Student`原型上的方法时，` Person`的原型上的也会跟着变化。

> **`Object.create()`**方法创建一个新对象，使用现有的对象来提供新创建的对象的`__proto__`。
>
> 所以，此时修改` Student`原型上的方法时，` Person`的原型上的不会跟着变化。

---

es6 之前没有` Object.create()`方法，可以自己实现(实际就是原型式继承的关键函数)

**关键**：

- 接受一个对象 obj
- 返回一个新对象 newObj
- 让` newObj.__proto__ === obj`

```js
function object(obj) {
  function F() {} // 新的构造函数
  F.prototype = obj; // 继承传入的参数obj
  return new F(); // 返回新的函数对象
}
```

---

参考链接：

- [js 继承的几种方式 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/37735247)

- [js 继承的 6 种方式 - ranyonsue - 博客园 (cnblogs.com)](https://www.cnblogs.com/ranyonsue/p/11201730.html)
