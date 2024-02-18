---
title: 攀爬TS之路(四)    函数类型
categories: 前端
date: 2022-06-18 12:46:34
tags:
  - TypeScript
---

# 攀爬TS之路(四)    函数类型

## 函数类型

### 三种方式

#### 函数声明
函数会有参数，也会有返回值。所以要用TS对函数进行约束的话，我们这个块都得限制。

```ts
function sum(a: number, b: number): number {
    return a + b;
}


console.log(sum(1, 2))
```

我们学过JS的话，应该会知道：JS的参数个数不符合函数定义时的，也不会报错。但是，在TS中，**输入多的或少的参数都是不允许的**
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/515310aed5e34a53be014dcabe71c0c4~tplv-k3u1fbpfcp-zoom-1.image)

#### 函数表达式

```ts
const sum: (a: number, b: number) => number = function(a, b) {
    return a + b;
}
```

这里一看，就会很混乱，又有箭头在，又好像不是箭头函数，下面就来分析一波。
首先，我们的`=`右边就是一般的函数表达式用法。
然后，我们在TS中,`=>`可以用来表示函数的定义，左边是输入类型，右边是输出类型。

```ts
sum: (a: number, b: number) => number   // 这一部分就是函数的类型，两个参数都要是number，返回值也得是number
```

TS的函数表达式搭配箭头函数一开始可能会觉得很怪。

```ts
const sum: (a: number, b: number) => number = (a, b) => a + b
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181246104.png)

#### 接口
函数的定义也能通过结果来实现。用接口来实现基本和对象的定义一样，就不多说废话了。

```ts
interface ISum {
    (a: number, b: number): number
}

const sum: ISum = (a, b) => a + b

console.log(sum(11, 22))    
```

### 可选参数
可选参数和对象中的可选属性一样，都是通过`?:`来定义。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97d3bcd8dfe74869838ae24a634f58a9~tplv-k3u1fbpfcp-zoom-1.image)

在函数中使用需要注意：**必选参数不能在可选参数后面**
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a516ee479b4140909966148ff45384c9~tplv-k3u1fbpfcp-zoom-1.image)

### 重载
重载就是让一个函数能够实现接收不同数量或类型的参数时，做不同的处理。

比如：一个函数当参数是数字时，返回参数乘积，当参数是字符串时，返回字符串拼接结果。

实现：

```ts
function myFunc(a: number | string, b: number | string): number | string | void {
    if (typeof a === 'number' && typeof b === 'number') {
        return a * b
    } else if (typeof a === 'string' && typeof b === 'string') {
        return a + b
    }
}

const x1 = myFunc(10, 20)
const x2 = myFunc('Hello ', 'CLZ')

console.log(x1)
console.log(x2)
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2c3da4b9709940acbb648c5a72ac6c3d~tplv-k3u1fbpfcp-zoom-1.image)

乍一看，这不是没有重载的必要吗？
重载能让我们能够得到具体的类型。

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181246101.png)
没有重载，得到的返回值的类型就会是联合类型，很混乱，也不能对函数的返回值进行类型定义。

重载函数就能解决上面说的问题。
重载函数也不是很难实现，只需要在实际实现函数之前多次定义函数(定义时只是定义好输入和输出的类型)
![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181246960.png)
