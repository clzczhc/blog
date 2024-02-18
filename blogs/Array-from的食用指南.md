---
title: Array.from的食用指南
categories: 前端
date: 2022-04-17 12:07:41
tags:
  - JavaScript

---

# Array.from的食用指南

## 将伪数组转换成数组

伪数组：有若干索引属性的任意对象以及一个length属性

```js
const fakeArr = {
  0: 'red',
  b: 'blue',
  2: 'purple',
  w: 'white',
  length: 4
}

console.log(Array.from(fakeArr))  // ['red', undefined, 'purple', undefined]
```

伪数组的属性名应该是索引，所以上面的例子中，属性名是字母的都得不到结果，因为没有该索引位置上的属性，所以最终的结果是` undefined`。



```js
const fakeArr = {
  0: 'red',
  1: 'blue',
  2: 'purple',
  3: 'white',
  4: 'black',
  length: 4
}

console.log(Array.from(fakeArr))  //  ['red', 'blue', 'purple', 'white']
```

其次，如果如果伪数组的属性有很多(个数比` length`属性还要多)的话，就会直截取一段` length`长度的。如上面例子中，` fakeArr`对象中实际有5个属性(除掉` length`)，但是最终通过` Array.from`转换成的真数组的长度为4。



## 将字符串转换成数组

```js
const mystr = 'Hello, CLZ!'
console.log(Array.from(mystr))  // ['H', 'e', 'l', 'l', 'o', ',', ' ', 'C', 'L', 'Z', '!']
```



## 将可迭代对象转换成数组

### map

```js
const mymap = new Map()
mymap.set('name', 'clz')
mymap.set('age', 21)

console.log(mymap)
console.log(Array.from(mymap))
```



![image-20220415004308593](https://s2.loli.net/2022/04/23/9hDvzS8WJjnMNTR.png)



### set

```js
const myset = new Set()

myset.add(1)
myset.add(2)
myset.add(1)
myset.add(3)

console.log(myset)              // Set(3) {1, 2, 3}
console.log(Array.from(myset))  // [1, 2, 3]
```



## 配合Set使用实现数组去重

首先，利用的是` Set`结构不允许重复的特性，然后利用` Array.from`可以将**可迭代对象**转换成数组，而` Set`恰恰也是可迭代对象。

```js
function unique(arr) {
  return Array.from(new Set(arr))
}

console.log(unique([1, 2, 3, 2, 1, 6, 6, 7, 5]))  // [1, 2, 3, 6, 7, 5]
```



## 浅克隆

因为数组也是可迭代对象，所以也可以将数组转换成数组，而这就实现了克隆

```js
function shallowClone(arr) {
  return Array.from(arr)
}

const arr = [1, 2, 3, 4, 5]
const arrCopy = shallowClone(arr)

console.log(arr)
console.log(arrCopy)

arrCopy[2] = 999
console.log('%c%s', 'color: red;font-size: 24px;', '=====================')

console.log(arr)
console.log(arrCopy)
```

![image-20220415092319749](https://s2.loli.net/2022/04/23/ab9NOnq6uotfwT2.png)



另外，`Array.from`是浅克隆，即如果数组里面有对象，那么只是复制对象的引用而已。所以修改克隆的对象，也会修改到原来的对象。直接给对象赋新值倒是不会修改到原来的对象因为这个时候是直接把地址给修改了。

```js
function shallowClone(arr) {
  return Array.from(arr)
}

const arr2 = [
  {
    name: 'clz',
    age: 21
  },
  {
    name: 'czh',
    age: 1
  }
]

const arrCopy2 = shallowClone(arr2)

arrCopy2[1].age = 999
console.log(arr2)
console.log(arrCopy2)
```

![image-20220415092444023](https://s2.loli.net/2022/04/23/jTUzdwHQJZka1lt.png)



## 接收第二个映射函数参数

可以直接修改数组的每一项的值，而不再需要` Array.from().map()`，因为` Array.from().map()`会创建出一个中间数组，而这个中间数组你没办法使用，却会占内存。

```js
const a1 = [1, 2, 3, 4];
const a2 = Array.from(a1, x => x ** 2);
console.log(a2); // [1, 4, 9, 16]

const a3 = Array.from(a1).map(x => x ** 2)    // 用这个方法来实现的话，会创建一个访问不到的中间数组
console.log(a3)   // [1, 4, 9, 16]
```



## 接收第三个映射函数参数

第三个参数可以指定第二个映射函数参数中` this`的值。这么一说，`Array.from`既有类似` map`的功能，还有类似` bind`的功能。

```js
const a1 = [1, 2, 3, 4];

const a2 = Array.from(a1, function (x) {
  return x * this.value
}, {
  value: 3
})

console.log(a1)   //  [1, 2, 3, 4]
console.log(a2)   //  [3, 6, 9, 12]
```



参考链接：

* https://blog.csdn.net/yangaoyuan1999/article/details/119993661
