---
title: JavaScript之对象(二)
categories: 前端
date: 2022-04-30 17:21:57
tags:
  - JavaScript
---

# JavaScript之对象(二)
> 看红宝书+查资料，重新梳理JavaScript的知识。

## 合并对象` Object.assign()`

` Object.assign()`可以用来将原对象的属性合并到目标对象上，而且这个方法还会返回合并后的目标对象。它会使用源对象上的`[[Get]]`取得属性的值，然后使用目标对象上的`[[Set]]`设置属性的值。

```js
let target = {
    name: 'clz'
};

let source = {
    age: 21
};

let result = null;

// 也会返回修改后的目标对象
result = Object.assign(target, source);

console.log('target: ', target);
console.log('source: ', source);
console.log('result: ', result);

console.log(target === result);
```



### 合并对象是浅克隆

合并对象是浅克隆。所以如果合并的属性是对象，那么修改源对象会修改到目标对象

```js
let target = {
    name: 'clz'
};

let source = {
    job: {
        salary: 1
    }
};

Object.assign(target, source);

source.job.salary = 999

console.log('target: ', target)
console.log('source', source)
```

![image-20220423200456546](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/04c8362f04ac42ed9ea9a60d8402754a~tplv-k3u1fbpfcp-zoom-1.image)



### 多个源对象合并

上面的例子中，`Object.assign`只接收了两个参数，第一个参数是目标对象，第二个参数是源对象。但是呢，其实它并不只是能够接收两个参数，可以接收多个参数，第一个参数是目标对象，其他的都是源对象，都会合并到目标对象上

```js
let target = {};
let source1 = {
    name: 'clz'
};
let source2 = {
    age: 19
};

Object.assign(target, source1, source2);
console.log(target);	// {name: 'clz', age: 19}
```



既然存在多个源对象合并，那么问题来了，如果多个源对象具有相同的属性时，会怎样呢？

多个源对象具有相同的属性时，会使用最后的那个属性值。实际过程就是从左往右合并对象，可以通过在目标对象上添加` set`函数观察过程。

```js
let target, source1, source2, source3;

target = {
    set job(newValue) {
        console.log(newValue)
    }
};

source1 = {
    job: {
        salary: 20,
    },
};

source2 = {
    job: "WebMaster",
};

source3 = {
    job: "Coder",
};

Object.assign(target, source1, source2, source3);
```

![image-20220423204031530](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1bcd9157883f4683aa9bc1cb028bddac~tplv-k3u1fbpfcp-zoom-1.image)



## 相等判定`Object.is()`
ES6 之前的相等判断

```js
console.log(+0 === -0); // true
console.log(+0 === 0); // true
console.log(-0 === 0); // true

console.log(NaN === NaN); // false
console.log(isNaN(NaN)); // true，要验证NaN的相等性需要使用isNaN
```

问题所在：

1. +0、0会等于-0
2. `NaN`不等于`NaN`，验证` NaN`的相等性需要换方法：使用`isNaN`函数



ES6 新增`Object.is()`，和` ===`很像，但是能更好的应对特殊情况。从下面的例子可以看出，` === `本来的功能它有，还能很好的应对上面说的问题。

```js
console.log(Object.is({}, {})); // false
console.log(Object.is("2", 2)); // false
console.log(Object.is(2, 2)); // true

console.log(Object.is(+0, -0)); // false
console.log(Object.is(+0, 0)); // true
console.log(Object.is(-0, 0)); // false

console.log(Object.is(NaN, NaN)); // true
```



## 属性

### 属性名简写

**常用**

当属性名和变量名相同时，我们可以使用简写。因为它会自动去找同名变量。如果找不到，则会报错。

```js
const age = 21;
const person = {
    age, // 当属性名和变量名一样时，可以简写。会自动去找同名变量。如果找不到，则会报错
    // age: age
};

console.log(person);    // {age: 21}
```



方法算是特殊的属性，也能简写。实际上，简写版本很常见

```js
let person = {
    // listen: function () {
    //     console.log('Euterpe')
    // }
    listen() {
        console.log("Euterpe");
    },
};

person.listen();    // Euterpe
```



### 可计算属性
引入可计算属性之前

我们现在有两个变量：` nameKey`和` ageKey`。想要让它的值作为对象的属性名。

```js
const nameKey = "name";
const ageKey = "age";

let person = {
    nameKey: 'clz',
    ageKey: 21
}
console.log(person)     // {nameKey: 'clz', ageKey: 21}
```

发现，结果和我们想象中的不太一样。



这个时候，如果想要实现我们想要的效果，只能够使用下标的形式，因为只有下标形式才会去找变量的值。

```js
const nameKey = "name";
const ageKey = "age";

// let person = {}
// person.nameKey = 'clz'
// person.ageKey = 21

// console.log(person)     // {nameKey: 'clz', ageKey: 21}

let person = {};
person[nameKey] = "clz";
person[ageKey] = 21;
console.log(person);       // {name: 'clz', age: 21}
```



引入可计算属性之后，可以直接在对象字面量中**直接动态命名属性**。

```js
const nameKey = "name";
const ageKey = "age";

let person = {
  [nameKey]: "clz",
  [ageKey]: 21,
};
console.log(person); // {name: 'clz', age: 21}
```



简写方法名可以和可计算属性搭配使用

```js
const methodKey = "listen";
let person = {
  [methodKey]() {
    console.log("Euterpe");
  },
};

person.listen();	// Euterpe
```



## 对象解构

**常用**，通过` {a} = obj`的形式，从` obj`中取出属性a的值，赋值给左边的a。

```js
let person = {
  name: "clz",
  age: 21,
};

// 对象解构
const { name, age } = person;
console.log(name, age);
```



对象解构支持重命名，` {value: newValue} = obj`，不过此时，` name`并不会有值。

```js
// 对象解构重命名
const { name: myname, age: myage } = person;
console.log(myname, myage);
```



如果解构赋值的属性不存在，那么该变量的值就是`undefined`

```js
// 如果解构赋值的属性不存在，那么该变量的值就是undefined
const { job } = person;
console.log(job);		// undefined
```



解构赋值支持定义默认值。

```js
// 定义默认值，当解构赋值的属性不存在时，该变量的值就是默认值
const { myjob = "Coder" } = person;
console.log(myjob);
```



对象解构会把原始值当成对象。**`null`和`undefined`不能被解构**，之前的**解构赋值的属性不存在，那么该变量的值就是`undefined`**对` null`和` undefined`不成立，会直接报错，如`Cannot destructure property '_' of 'null' as it is null.`



### 先声明后解构赋值

```js
const person = {
    name: 'clz',
    age: 21
}

let name, age

{ name, age }   = person    // 报错，`Uncaught SyntaxError: Unexpected token '='`
```



那么，是不是不能先声明，后解构复制呢？
答案是可以的，只不过，此时赋值表达式需要包含在一对括号里

```js
const person = {
    name: "clz",
    age: 21,
};

let name, age;

({ name, age } = person);

console.log(name, age);     // clz 21
```



不只是小括号可以，使用中括号也可以，不过使用花括号就会报错了，因为解构赋值就是用的花括号

### 解构赋值支持嵌套结构

同理，第一层不会有结果，可以打印`job`测试

```js
let person = {
    job: {
        salary: 112,
    },
};

const {
    job: { salary },
} = person;
console.log(salary);    // 112
```



### 部分解构

部分解构：如果一个解构表达式多个赋值，开始的赋值成功，而中间的赋值出错的话，解构赋值开始的部分还是能完成，只是中间报错的部分以及后面的部分不能完成。

```js
const person = {
  name: "clz",
  age: 21,
};

let personName;
let personFoo;
let personAge;

try {
  ({
    name: personName,
    // 因为personFoo没有bar属性，所以会报错
    bar: personFoo.bar,
    age: personAge,
  } = person);
} catch (e) {
  console.log(e);
}
console.log(personName, personAge, personFoo);
```

![image-20220423211723147](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/05a41edeb8b945c8b9505a76d376709c~tplv-k3u1fbpfcp-zoom-1.image)



### 参数上下文匹配
在函数参数中也可以进行解构赋值。

```js
let person = {
    name: "clz",
    age: 27,
};

function mytest1(a, { name, age }, b) {
    console.log(a);
    console.log(name, age);
    console.log(b);
}

function mytest2({ name: myName, age: myAge }) {
    console.log(myName, myAge);
}

mytest1(1, person, 2);

console.log("%c%s", "color:red;font-size:24px;", "============");
mytest2(person);
```

![image-20220423211842229](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2ae577d06d0e44f590dcc74229296ee4~tplv-k3u1fbpfcp-zoom-1.image)
