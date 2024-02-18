---
title: 自定义工具函数库(二)   数组相关
date: 2022-03-20 21:04:34
categories: 前端
tags:
  - JavaScript
---

# 自定义工具函数库(二) 数组相关

最终仓库：[utils: 自定义工具库](https://github.com/13535944743/utils)

以前的笔记：[JS 数组常用的方法](https://www.clzczh.top/2021/11/14/js-array-methods/)

## 1. 数组声明式系列方法

### 1.1 map 函数封装实现

> ` map()`方法创建一个新数组，其结果是该数组中的每个元素各自调用一次提供的函数后的返回值

循环，数组的每个元素都调用一次函数，并把每次循环得到的返回值都存好，循环结束后，把存好的数组返回。

```js
// map函数的封装实现
function map(arr, callback) {
  let ret = [];

  for (let i = 0; i < arr.length; i++) {
    ret.push(callback(arr[i], i));
  }

  return ret;
}
```

测试：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>map函数封装实现</title>
    <script src="./map.js"></script>
  </head>

  <body>
    <script>
      const arr = [1, 2, 3, 4, 5];
      // 自定义
      const result = map(arr, (item, index) => {
        console.log(index);
        return item + 1;
      });

      console.log(result);

      // 原生
      // const result = arr.map((item, index) => {
      //   console.log(index)
      //   return item + 1
      // })

      // console.log(result)
    </script>
  </body>
</html>
```

### 1.2 reduce 函数

> 对数组中的每个元素执行一个由您提供的**reducer**函数(升序执行)，将其结果汇总为单个返回值。

循环遍历数组，并每次都把调用函数得到的值，重新赋值给` ret`变量，然后作为下一次调用函数时的第一个参数

```js
// reduce函数的封装实现
function reduce(arr, callback, initValue) {
  let ret = initValue;
  for (let item of arr) {
    ret = callback(ret, item); // 每次调用都把最新的值重新赋给ret
  }

  return ret;
}
```

测试：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>reduce函数封装实现</title>
    <script src="./reduce.js"></script>
  </head>

  <body>
    <script>
      const arr = [1, 2, 3, 4];
      // 自定义
      const result = reduce(
        arr,
        (preValue, curValue) => preValue + curValue,
        10
      );

      console.log(result);

      // 原生
      // const result = arr.reduce(
      //   (preValue, curValue) => preValue + curValue,
      //   0)

      // console.log(result)
    </script>
  </body>
</html>
```

### 1.3 filter 函数

> 创建一个新数组, 其包含通过所提供函数实现的测试的所有元素。

循环，如果以该数组元素作为参数调用函数的返回值为` true`，则存好。

```js
// filter函数的封装实现
function filter(arr, callback) {
  let ret = [];
  for (let i = 0; i < arr.length; i++) {
    if (callback(arr[i], i)) {
      ret.push(arr[i]);
    }
  }

  return ret;
}
```

测试：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>filter函数封装实现</title>
    <script src="./filter.js"></script>
  </head>

  <body>
    <script>
      const arr = [1, 2, 3, 4, 5];
      // 自定义
      const result = filter(arr, (item, index) => index % 2 === 0);

      console.log(result);

      // 原生
      // const result = arr.filter(item => item >= 3)

      // console.log(result)
    </script>
  </body>
</html>
```

### 1.4 find 函数

> 返回数组中满足提供的测试函数的第一个元素的值。否则返回 [`undefined`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined)。

```js
// find函数的封装实现
function find(arr, callback) {
  for (let i = 0; i < arr.length; i++) {
    if (callback(arr[i], i)) {
      return arr[i];
    }
  }

  return undefined;
}
```

测试：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>find函数封装实现</title>
    <script src="./find.js"></script>
  </head>

  <body>
    <script>
      const arr = [1, 2, 3, 4, 5];
      // 自定义
      const result1 = find(arr, (item, index) => item >= 3);
      console.log(result1);

      const result2 = find(arr, (item, index) => item >= 10);
      console.log(result2);

      // 原生
      // const result = arr.find((item, index) => item >= 3)

      // console.log(result)
    </script>
  </body>
</html>
```

### 1.5 findIndex 函数

> 返回数组中满足提供的测试函数的第一个元素的**索引**。若没有找到对应元素则返回-1。

和 find()函数类似，只不过是返回满足条件的第一个元素的索引，以及没有满足的是返回-1

```js
// findIndex函数的封装实现
function findIndex(arr, callback) {
  for (let i = 0; i < arr.length; i++) {
    if (callback(arr[i], i)) {
      return i;
    }
  }

  return -1;
}
```

测试：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>findIndex函数封装实现</title>
    <script src="./findIndex.js"></script>
  </head>

  <body>
    <script>
      const arr = [1, 2, 3, 4, 5];
      // 自定义
      const result1 = findIndex(arr, (item, index) => item >= 3);
      console.log(result1);

      const result2 = findIndex(arr, (item, index) => item >= 10);
      console.log(result2);

      // 原生
      // const result = arr.findIndex((item, index) => item >= 10)

      // console.log(result)
    </script>
  </body>
</html>
```

### 1.6 every 函数

> 测试一个数组内的所有元素是否都能通过某个指定函数的测试。它返回一个布尔值。

```js
// every函数的封装实现
function every(arr, callback) {
  for (let i = 0; i < arr.length; i++) {
    if (!callback(arr[i], i)) {
      return false;
    }
  }

  return true;
}
```

测试：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>every函数封装实现</title>
    <script src="./every.js"></script>
  </head>

  <body>
    <script>
      const arr = [1, 2, 3, 4, 5];
      // 自定义
      const result1 = every(arr, (item, index) => item >= 3);
      console.log(result1);

      const result2 = every(arr, (item, index) => item < 10);
      console.log(result2);

      // 原生
      // const result = arr.every((item, index) => item < 10)

      // console.log(result)
    </script>
  </body>
</html>
```

### 1.7 some 函数

> 测试数组中是不是至少有 1 个元素通过了被提供的函数测试。它返回的是一个 Boolean 类型的值。

```js
// some函数的封装实现
function some(arr, callback) {
  for (let i = 0; i < arr.length; i++) {
    if (callback(arr[i], i)) {
      return true;
    }
  }

  return false;
}
```

测试：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>some函数封装实现</title>
    <script src="./some.js"></script>
  </head>

  <body>
    <script>
      const arr = [1, 2, 3, 4, 5];
      // 自定义
      const result1 = some(arr, (item, index) => item < 3);
      console.log(result1);

      const result2 = some(arr, (item, index) => item < 0);
      console.log(result2);

      // 原生
      // const result = arr.some((item, index) => item < 3)

      // console.log(result)
    </script>
  </body>
</html>
```

## 2. 数组去重

### 2.1 forEach() + indexOf()

```js
// 数组去重方法1：forEach() + indexOf()
function unique(arr) {
  const ret = [];

  arr.forEach((item) => {
    if (ret.indexOf(item) === -1) {
      // 如果数组中不存在当前值，则存进去
      ret.push(item);
    }
  });

  return ret;
}
```

### 2.2 forEach() + 容器(对象)

```js
// 数组去重方法2：forEach() + 容器(对象)
function unique(arr) {
  const ret = [];
  const obj = {}; // 键是数组之前出现过的元素，值是是否有出现过。
  // 出现过值是true，没出现过只是undefined

  arr.forEach((item) => {
    if (obj[item]) {
      return;
    }

    obj[item] = true;
    ret.push(item);
  });

  return ret;
}
```

### 2.3 Set + Array.from() 或 扩展运算符(...)

```js
// 数组去重方法3：Set + Array.from() 或 扩展运算符(...)
function unique(arr) {
  let set = new Set(arr);
  // return Array.from(set)      // Array.from()可以通过可迭代对象(包括数组、Set等)创建一个新的数组

  return [...set]; // 使用ES6的扩展运算符`...`
}
```

## 3. concat 函数

> 用于合并两个或多个数组。此方法不会更改现有数组，而是返回一个新数组。

```js
// concat函数的封装实现
function concat(arr, ...args) {
  const ret = [...arr];

  args.forEach((item) => {
    if (Array.isArray(item)) {
      // 如果是数组，则通过扩展运算符把数组中的元素push到新数组中
      ret.push(...item);
    } else {
      ret.push(item);
    }
  });

  return ret;
}
```

测试：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>concat函数实现</title>
    <script src="./concat.js"></script>
  </head>

  <body>
    <script>
      let arr = [1, 2, 3];

      // 自定义函数
      const result = concat(arr, [4, 5, 6], 7, 8, [9, 10]);
      console.log(result);

      // 内置方法
      // const result = arr.concat([4, 5, 6], 7, 8)
      // console.log(result)
    </script>
  </body>
</html>
```

## 4. slice 数组切片

> 返回一个新的数组对象，这一对象是一个由 `begin` 和 `end` 决定的原数组的**浅拷贝**（包括 `begin`，不包括`end`）。原始数组不会被改变。

```js
// slice函数实现
function slice(arr, begin, end) {
  if (arr.length === 0) {
    return [];
  }

  begin = begin || 0; // 如果begin没有传，则从0开始切
  if (begin >= arr.length) {
    return [];
  }

  end = end || arr.length;

  const ret = [];
  for (let i = 0; i < arr.length; i++) {
    if (i >= begin && i < end) {
      ret.push(arr[i]);
    }
  }

  return ret;
}
```

测试：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>slice函数实现</title>
    <script src="./slice.js"></script>
  </head>

  <body>
    <script>
      let arr = [1, 2, 3, 4, 5, 6, 7];

      // 自定义函数
      const result1 = slice(arr, 2, 6);
      console.log(result1);

      const result2 = slice(arr);
      console.log(result2);

      const result3 = slice(arr, 2);
      console.log(result3);

      const result4 = slice(arr, 3, 1);
      console.log(result4);

      // 内置方法
      // const result = arr.slice(2, 6) // 截取索引在[2， 6)范围内的元素
      // console.log(result)
    </script>
  </body>
</html>
```

## 5. 数组扁平化

数组扁平化是指将一个多维数组变为一维数组

**内置方法 flat**：

```js
let arr = [1, 2, [3, 4], [[5], 6], 7];

const result = arr.flat(Infinity);
console.log(result); //  [1, 2, 3, 4, 5, 6, 7]
```

### 5.1 递归

```js
// 数组扁平化(1): 通过concat和forEach实现，遇到多维数组时，通过递归调用flatten实现扁平化
function flatten(arr) {
  let ret = [];

  arr.forEach((item) => {
    if (Array.isArray(item)) {
      ret = ret.concat(flatten(item)); // 如果是数组，继续打平
    } else {
      ret = ret.concat(item);
    }
  });

  return ret;
}
```

### 5.2 扩展运算符...

```js
// 数组扁平化(2): 通过concat函数和扩展运算符...实现
function flatten(arr) {
  let ret = [...arr];

  while (ret.some((item) => Array.isArray(item))) {
    ret = [].concat(...ret);
    // 举例：arr = [1, 2, 3, 4, [5, 6]]
    // 则ret = [].concat(1, 2, 3, 4, [5, 6]) = [1, 2, 3, 4, 5, 6]
  }

  return ret;
}
```

<br>

### 5.3 toString / join + split + map

先通过` toString`或` join`把数组转成字符串，在通过` split`转换成数组，最后还需要通过` map`函数，把数组的每一个元素变回数字。(不考虑其他类型的数组的话)

```js
function flatten(arr) {
  return arr
    .toString()
    .split(",")
    .map((item) => Number(item));
  // return arr.join().split(',').map(item => Number(item))
}
```

<br>

### 5.4 reduce

遍历数组，如果是数组，则递归遍历，否则，通过` concat`拼接起来

```js
function flatten(arr) {
  return arr.reduce((pre, cur) => {
    return pre.concat(Array.isArray(cur) ? flatten(cur) : cur);
  }, []);
}
```

<br>

## 6. 数组分块

> 语法：` chunk(array, size)`
>
> 功能：将数组拆分成多个 size 大小长度的区块，每个区块组成小数组，整体组成一个二维数组
>
> 例子：[1, 2, 3, 4, 5, 6]调用 chunk(arr, 4) => [[1, 2, 3, 4], [5, 6]]

```js
// 数组分块：这里用了比较巧妙的方法。暂存分块好的数组为0时，把它push到ret数组中，然后通过数组的引用性质，给temp数组push值，从而也改变ret数组的值
function chunk(arr, size = 1) {
  let ret = [];
  let temp = []; // 暂存分块好的数组，再push到ret中

  arr.forEach((item) => {
    if (temp.length === 0) {
      ret.push(temp);
      // console.log(1, ret)   // 如需测试，请在node环境测试
    }

    temp.push(item);
    // console.log(2, ret)

    if (temp.length === size) {
      temp = [];
    }
  });

  return ret;
}

// console.log(chunk([1, 2, 3, 4, 5, 6, 7], 3))
```

## 7. 数组差集

> - 语法: difference(arr1, arr2)
> - 功能: 得到当前数组中所有不在 arr 中的元素组成的数组(不改变原数组)
> - 例子: difference([1,3,5,7], [5, 8]) ==> [1, 3, 7]

```js
// 数组差集
function difference(arr1, arr2 = []) {
  if (arr1.length === 0) {
    return [];
  }
  if (arr2.length === 0) {
    return arr1.slice();
  }

  let ret = arr1.filter((item) => !arr2.includes(item));
  return ret;
}

// console.log(difference([1, 2, 3, 4], [3, 4, 5]))
```
