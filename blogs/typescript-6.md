---
title: 攀爬TS之路(六)    类型别名、字面量类型、枚举
categories: 前端
date: 2022-06-18 12:47:43
tags:
  - TypeScript
---

# 攀爬TS之路(六)    类型别名、字面量类型、枚举

## 类型别名
类型别名就是给一个类型起一个新名字。使用关键字`type`。

```ts
type Name = string


const myname1: Name = 'clz'
const myname2: Name = 123
```
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4416a0dc1fd649be9868a9bbc41b14c5~tplv-k3u1fbpfcp-zoom-1.image)

上面的例子中，使用了类型别名，所以后续可以直接使用类型别名`Name`来当成`string`使用。

如果给比较复杂的类型使用类型别名，后续使用就会很方便。
```ts
type SumType = (a: number, b: number) => number

const sum: SumType = (a, b) => a + b

console.log(sum(11, 22))
console.log(sum(11, ''))    // 报错：类型“string”的参数不能赋给类型“number”的参数。
```

## 字面量类型
可以使用字面量类型来约束取值只能为特定的字面量。
```ts
type Name = 'clz' | 123 | true | { name: 'clz' }

const myname1: Name = 'clz'
const myname2: Name = 123
const myname3: Name = true
const myname4: Name = {
    name: 'clz'
}

const myname5: Name = 'ccc'
const myname6: Name = 124
const myname7: Name = false
const myname8: Name = {
    name: 'czh'
}
```
![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181248354.png)

**`null`和`undefined`不受限制**
```ts
type Name = 'clz' | 123 | true | { name: 'clz' }

const myname1: Name = null
const myname2: Name = undefined
```



## 枚举

枚举一般用来表示一组常量，比如一周的七天，方向有东南西北等。

### 基本使用

使用方法很简单：
```ts
enum Direction {
    East,
    South,
    West,
    North
}

console.log(Direction['East'])     // 0
console.log(Direction['South'])    // 1
console.log(Direction['West'])     // 2
console.log(Direction['North'])    // 3
```

枚举成员会被赋值从`0`开始递增数字，所以上面的例子会依次打印0、1、2、3。
另外，还会对枚举值到枚举名进行反向映射，如枚举成员`East`的值是0，那么`Direction[0]`的值就是`East`

```ts
console.log(Direction['East'])     // 0
console.log(Direction[0])          // East
```

那么，这是怎么实现的呢？
先打印`Direction`瞧瞧。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/803a015557c84c6195c05a6016d9e213~tplv-k3u1fbpfcp-zoom-1.image)

发现这个枚举对象的键和值都是该对象的属性。接下来当然得看看编译成JS后究竟是怎么实现的。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fcc386fca2e64b11bf2c3a8f3d466b87~tplv-k3u1fbpfcp-zoom-1.image)

发现还挺简单。就是赋值的时候，把另一个赋值表达式当成键来赋值，这样子就会拿那个表达式的结果来当成键来使用。

### 手动赋值
```ts
enum Direction {
    East = 4,
    South = 1,
    West,
    North
}

console.log(Direction)
```

手动赋值的时候，枚举项的值将不再是从0开始递增了，而是会接着上一个枚举项递增。比如上面的例子中，`South`的值是1，而后面的`West`没有手动赋值，所以它的值是2，同理，`North`的值是3。

**注意**：非手动赋值的枚举项和手动赋值的重复的时候，后面的会把前面的给覆盖掉。

```ts
enum Direction {
    East = 2,
    South = 1,
    West,
    North
}

console.log(Direction[2])   // West
console.log(Direction)      // 键为2时的结果是West。键为2，值为East的被覆盖掉了
```

### 常数枚举
常数枚举就是使用`const enum`定义的枚举类型。它和普通枚举不同，它会在编译阶段被删除。

所以下面这段代码编译出来是什么东西都没有。
```ts
const enum Direction {
    East,
    South,
    West,
    North,
}
```
![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181248319.png)

```ts
console.log([Direction.East, Direction.South, Direction.West, Direction.North])
```
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a1a88bbc7bc54017887bf4c39f525d4d~tplv-k3u1fbpfcp-zoom-1.image)

当我们使用时，编译的结果后面会有**该枚举项的键**作为注释。

### 其他使用方式

#### 枚举项是小数或负数
手动赋值的枚举项还可以是小数或负数，递增步长仍然是1

```ts
enum Direction {
    East = -4,
    South = -1.5,
    West,
    North
}

console.log(Direction)     
```
![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181248176.png)

#### 枚举项不是数字
**手动赋值的枚举项可以不是数字**

```ts
enum Direction {
    East = "E",
    South = "S",
    West = "W",
    North = "N"
}

console.log(Direction)  
```
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/87dcf47cbf744859a0e4809f53f63964~tplv-k3u1fbpfcp-zoom-1.image)

这个时候没有反向映射了，如果枚举项不是数字，但是还是想要有反向映射的话。需要使用类型断言来让tsc无视类型检查，并且全部都要并且只能断言为`any`。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/521234f6125f4767ac3d9db0074f9711~tplv-k3u1fbpfcp-zoom-1.image)

```ts
enum Direction {
    East = "E" as any,
    South = "S" as any,
    West = "W" as any,
    North = "N" as any
}

console.log(Direction)     
```
![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181248373.png)

**手动赋值的枚举项不是数字时，后面不能有非手动赋值项**
![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202206181248475.png)
