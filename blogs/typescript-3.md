---
title: 攀爬TS之路(三)    数组类型、元组类型
categories: 前端
date: 2022-06-18 12:45:19
tags:
  - TypeScript
---

# 攀爬TS之路(三)    数组类型、元组类型

## 数组类型

数组类型有多种定义方式。

### 普通法
这个方法基本上和其他静态语言的使用差不多
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ccb3e2a0bd7243898fd47f59164f371f~tplv-k3u1fbpfcp-zoom-1.image)

数组使用联合类型(这个看的教程没有这种用法，有问题可以评论交流)
![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181245448.png)

### 数组泛型
使用数组泛型(`Array<type>`)来定义数组。

```ts
let nums: Array<number | string> = [1, 2, 3]

nums.push(2)
nums.push('3')

console.log(nums)       //  [1, 2, 3, 2, '3']

nums.push(true)     // 这里报错，因为定义的数组类型里不包括`boolean`类型
```

### 接口
数组就是一个特殊的对象，它的键是数字，且是从0开始。所以我们也可以使用接口来表示数组。

```ts
interface IArray {
    [index: number]: number | string
}

const arr: IArray = [1, 2, 'hello']

console.log(arr)
```

使用接口表示数组有很大问题：不能调用数组的方法
![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181245606.png)

没想到好的解决方案，有想法的可以评论一下(虽然不建议用这个)

## 元组类型
元组在赋值时，需要提供元组类型中指定的项。

```ts
const tuple: [number, string, number | string] = [1, '2', 'hello']  // 正常

// const tuple: [number, string, number | string] = [1, '2']   // 报错：不能将类型“[number, string]”分配给类型“[number, string, string | number]”。源具有 2 个元素，但目标需要 3 个。

// const tuple: [number, string, number | string] = [1, '2', 'hello', 4]   // 不能将类型“[number, string, string, number]”分配给类型“[number, string, string | number]”。源具有 4 个元素，但目标仅允许 3 个。
```

这么一看，就像是一个固定大小和元素类型的数组。
但是，因为TS是JS的超集，所以元组能够使用数组的方法，即我们可以通过数组的方法让该元组不再固定大小。(这里说实在有点迷，赋值的时候元组大小固定，调方法又能让元组大小不固定)

```ts
let tuple: [number, string, number | string]

tuple = [1, 'hello', 3]

tuple.push('123')
console.log(tuple)  //  [1, 'hello', 3, '123']
```

当我们添加越界的元素时，类型会被限制成元组中每个类型的联合类型。

```ts
let tuple: [number, boolean]

tuple = [1, true]

tuple.push(true) //  允许越界

tuple.push('123')   // 报错：允许越界。但是越界的元素需要是元组中每个类型的联合类型
console.log(tuple)  
```
