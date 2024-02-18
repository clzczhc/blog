---
title: JS数组常用的方法
date: 2021-11-14 00:09:45
keywords: js JavaScript Array
categories: 前端
tags:
  - JavaScript
---

# JS 数组常用的方法(个人感觉)

## 1. forEach()

循环，无法在中间停止

![](https://pic.imgdb.cn/item/61500fc62ab3f51d91ee7252.jpg)

## 2. some()

循环，找到符合条件的之后，可以通过 return true 退出循环

![](https://pic.imgdb.cn/item/6150106d2ab3f51d91ef3344.jpg)

## 3. every()

测试数组中的所有元素是否都能通过某个指定函数的测试。返回一个布尔值。

```js
const arr = [-1, 3, 4, 5, 6];

let result = arr.every((value) => value > 0); //-1不满足条件，所以返回false
console.log(result);

arr[0] = 1;
result = arr.every((value) => value > 0); //全部满足条件，所以返回true
console.log(result);
```

## 4. filter()

创建一个新数组，它包含符合某个指定函数的测试的所有元素

![](https://pic.imgdb.cn/item/615013102ab3f51d91f258ee.jpg)

## 5. reduce()

实现累加

` arr.reduce((累加的结果, 当前循环项) => {}, 初始值)`

```js
const arr = [1, 3, 7];

let result = arr.reduce((sum, value) => (sum += value), 0);
console.log(result); //输出11
```

搭配 filter()实现，累加选中状态的总价格

```js
const arr = [
  {
    id: 1,
    name: "西瓜",
    state: true,
    price: 10,
    count: 1,
  },
  {
    id: 2,
    name: "榴莲",
    state: false,
    price: 80,
    count: 2,
  },
  {
    id: 3,
    name: "草莓",
    state: true,
    price: 20,
    count: 3,
  },
];

const amount = arr
  .filter((item) => item.state)
  .reduce((amount, item) => (amount += item.price * item.count), 0);
console.log(amount); //返回70
```

## 6. map()

map()方法把调用它的数组的每一个元素分别传给指定的函数，返回这个函数的返回值构成的数组

```js
let a = [1, 2, 3];
let newA = a.map((v) => v * v);
console.log(newA);

let a = [1, , 2, , 3]; // 如果数组是稀疏的，缺失元素不会调用函数，但是返回的数组也会和原始数组一样稀疏
let newA = a.map((v) => v * v);
console.log(newA);
```

## 7. find()

遍历数组，寻找第一个符合条件的元素，找到就停止迭代

```js
let a = (a = [1, 2, 3, 4, 5]);
console.log(a.find((v) => v >= 2));
console.log(a.find((v) => v < 0)); // 找不到符合条件的，返回undefined
```

## 8. flat()

用于打平数组(把嵌套数组变为普通的数组元素)

```js
let a = [1, [2, 3]];
console.log(a.flat());

a = [1, [2, [3]]];
console.log(a.flat()); // 不带参，只会打平一级嵌套

a = [1, [2, [3, [4]]]];
console.log(a.flat(1));
console.log(a.flat(2)); // 带参，可以打平多级嵌套
console.log(a.flat(3));
console.log(a.flat(4)); // 打平后的数组如果没有嵌套数组，则不会再被打平

a = [1, [2, 3, [4, 5, 6, [7, 8, [9, 10, [11, 12]]]]]];
console.log(a.flat(Infinity)); // 通过Infinity实现把任意嵌套数组打平为不嵌套的数组
```

## 9. concat()

```js
let a = [1, 2, 3];
let b = [4, 5, 6];
console.log(b.concat(a)); // 拼接两个数组
console.log(b.concat(1, 2)); // 也可以添加元素。不会改变原数组，而是创建新数组
```

## 10. push()、pop()、shift()、unshift()

```js
let a = [];
a.push(1);
console.log(a.push(2, 3)); // push()在数组末尾添加元素，并返回数组的新长度
console.log(a);

console.log(a.pop()); // pop()删除数组末尾的元素，并返回删除的元素
console.log(a);

console.log(a.unshift(11, 22)); // unshift()在数组的开头添加元素，并返回数组的新长度
console.log(a);

console.log(a.shift()); // shift()删除数组开头的元素，并返回删除的元素
console.log(a);
```

## 11. slice()、splice()

```js
let a = [1, 2, 3, 4, 5, 6, 7, 8];
console.log(a.slice(3, 6)); // 截取数组的一部分，返回一个新数组，第一个参数是起点，第二个是终点(但不包含终点)
console.log(a);

console.log(a.slice(2)); // 一个参数，则是从起点截到数组的末尾

console.log(a.slice(-4, -2)); // 可以是负数，-1表示倒数第一，-2表示倒数第二

console.log(a.slice(6, 0)); // 返回[], 不能往回截

/*************************/
a = [1, 2, 3, 4, 5, 6, 7, 8]; // splice()会改变数组
console.log(a.splice(2, 3, "Hello", "Hi")); // splice()的第一个参数是起点，第二个参数是要删除的元素个数，之后的参数是要插入的元素，返回删除的数组
console.log(a);

console.log(a.slice(3)); // 只有一个参数，则删除数组开头到起点的全部元素
```

## 12. indexOf()、lastIndexOf()

```js
let a = [1, 2, 3, 4, 2, 1];
console.log(a.indexOf(2)); // 返回1， 数组a中第一个是2的元素的索引值是1
console.log(a.lastIndexOf(2)); // 返回4， 数组a中最后一个是2的元素的索引值是4
console.log(a.lastIndexOf(99)); // 返回-1，找不到

console.log(a.indexOf(2, 2)); // 从索引值为2的元素开始找
console.log(a.lastIndexOf(2, -3)); // 从倒数第三个元素开始找
```

## 13. includes()

```js
let a = [1, "Hello", true];
console.log(a.includes(true)); // 返回true，因为数组中有true这个元素
console.log(a.includes(2)); // 返回false
```

## 14. fill()

```js
let a = new Array(7);
console.log(a.fill(0)); // 可以用于数组的初始化, 返回一个新数组
console.log(a.fill(7, 2)); // 从索引2开始把数组元素设置为7
console.log(a.fill(3, 2, -2)); // 第一个参数是要设置的值，第二个参数是起点，第三个参数是终点，可以是负数
```

## 15. join()、split()

```js
// join()用于把所有元素转换成字符串
let a = [1, 2, 3];
console.log(a.join()); // 默认使用逗号当分隔符
console.log(a.join(" ")); // 使用空格当作分隔符

// split()用于把字符串按指定分隔符转换成数组
a = "Hello world!";
console.log(a.split(""));
a = "Hello, world!";
console.log(a.split(","));
```

## 16. sort()

sort()方法对数组元素按字母顺序对数组元素排序

```js
let arr = [1, 2, 11, 23, 22, 111, 12, 9, 8];
console.log(arr.sort());
```

结果：

![](https://pic.imgdb.cn/item/6162e07f2ab3f51d91f3bcb3.jpg)

可以传入一个回调函数，来自定义排序规则。

回调函数的格式是

```js
(a, b) => {
  // a, b是数组中任意两个数
  return xxx;
};
```

<b style="color: red">当返回值大于 0 时，a 排在 b 的后面；</b>

<b style="color: red">当返回值小于 0 时，a 排在 b 的前面；</b>

<b style="color: red">当返回值等于 0 时，a 和 b 的顺序不改变。</b>

所以，要实现升序排序，可以按下面的方法

```js
let arr = [1, 2, 11, 23, 22, 111, 12, 9, 8];
console.log(
  arr.sort((a, b) => {
    if (a > b) {
      return 1;
    } else if (a < b) {
      return -1;
    } else {
      return 0;
    }
  })
);
```

上面的代码还可以简化为：

```js
let arr = [1, 2, 11, 23, 22, 111, 12, 9, 8];
console.log(arr.sort((a, b) => a - b));
/* 要实现降序的话，则是变为return b - a; */
```

原因(看不懂的话，结合上面红字)：

当 a > b 时，

a - b > 0, return a - b 排序的结果 ==> b, a (升序)

b - a < 0, return b - a 排序的结果 ==> a, b (降序)

当 a < b 时，

a - b < 0, return a - b 排序的结果 ==> a, b (升序)

b - a > 0, return b - a 排序的结果 ==> b, a (降序)

当 a = b 时，

a - b = b - a = 0, return a - b 和 return b - a 排序结果都不变

从上面所有的情况，可以发现，无论 a > b，还是 a < b, return a - b 总能得到升序的结果，return b - a 总能得到降序的结果
