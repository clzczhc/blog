---
title: TypeScript笔记
date: 2022-02-09 15:53:24
keywords: typescript TS TypeScript
categories: "前端"
tags:
  - 青训营
  - TypeScript
---

# TypeScript 笔记

参加字节跳动的青训营时写的笔记。这部分是林皇老师讲的课。(过年偷懒，项目爆肝后，重新整理笔记)

**个人博客**(欢迎光临)：[TypeScript 笔记]()

## 1. 简介

- **静态类型**
  - 可读性增强：基于语法解析 TS Doc，IDE 增强
  - 可维护性增强：在编译阶段暴露大部分的错误(类型匹配错误、拼写错误等)
- **JS 的超集**
  - 包含兼容所有 JS 特性
  - 支持渐进式引入和升级，支持与 JS 共存

![image-20220124100758987](https://s2.loli.net/2022/01/24/Oq5kaANnRpWg4Pu.png)

动态类型：数据类型不是在编译阶段决定的，而是在运行阶段决定的

静态类型：数据类型是在编译期间或运行之前确定的，即编写代码时需要定义变量的类型。

强类型：变量指定了数据类型之后，如果不经过强制类型转换，那么该变量永远是这个数据类型

弱类型：数据类型可以被忽略，一个变量可以赋予不同数据类型的值。即如果给整型变量 a 赋值字符串，则 a 变成字符串类型。

[更多](https://www.cnblogs.com/zy1987/p/3784753.html)

## 2. 基本语法

### 2.1 基础数据类型

**JS**：

```js
/* 字符串 */
const s = "Hello";

/* 数字 */
const num = 1;

/* 布尔值 */
const b = true;

/* null */
const n = null;

/* undefined */
const u = undefined;
```

**TS**：

```typescript
/* 字符串 */
const s: string = "Hello";

/* 数字 */
const num: number = 1;

/* 布尔值 */
const b: boolean = true;

/* null */
const n: null = null;

/* undefined */
const u: undefined = undefined;
```

### 2.2 对象类型

```typescript
interface IPeople {
  readonly id: number; // 只读属性
  name: string;
  sex: "male" | "female";
  age: number;
  hobby?: string; // 可选属性
  [key: string]: any; // 对象可以有任意属性，键是字符串类型的，值是任意类型的
}
```

![image-20220126130542691](https://s2.loli.net/2022/01/26/KXsNIl9v12ROkw5.png)

![image-20220126130327610](https://s2.loli.net/2022/01/26/ABfOmuJXQi4IpCZ.png)

![image-20220126130918252](https://s2.loli.net/2022/01/26/mqoEcT5Q3uKSWNv.png)

### 2.3 函数类型

```typescript
/* 1 */
function add(x: number, y: number): number {
  return x + y;
}

/* 2 */
const add: (x: number, y: number) => number = (x, y) => x + y;

/* 3 */
interface IAdd {
  (x: number, y: number): number;
}
const add: IAdd = (x, y) => x + y;
```

#### 2.3.1 函数重载

```typescript
function getDate(type: "string", timestamp?: string): string;
function getDate(type: "date", timestamp?: string): Date;
function getDate(type: "string" | "date", timestamp?: string): Date | string {
  const date = new Date(timestamp);
  return type === "string" ? date.toLocaleDateString() : date;
}

const x = getDate("date"); // x: Date
const y = getDate("string", "2022-02-01"); // y: string
```

上面的函数的目的就是第一个参数是` date`时，返回 Date 对象；是` string`的话，则返回时间戳字符串。

没有上面两个定义函数的情况：

![image-20220205182147044](https://s2.loli.net/2022/02/05/qxJn1BGjMik3lAL.png)

![image-20220124103048922](https://s2.loli.net/2022/01/24/B8jeEh7nMd2alok.png)

### 2.4 数组类型

```typescript
// 1. 类型 + []表示
type IArr1 = number[]; // type关键字定义了IArr1的别名类型

// 2. 泛型表示
type IArr2 = Array<string | number | Record<string, number>>; // Record：对象类型的简化写法

// 3. 元组表示
type IArr3 = [number, string, number];

// 4. 接口表示
interface IArr4 {
  [key: number]: any;
}
```

```typescript
const arr1: IArr1 = [1, 1, 2];
const arr2: IArr2 = [1, "hello", { age: 21 }];
const arr3: IArr3 = [1, "hello", 1];
const arr4: IArr4 = ["string", () => null, []];
```

### 2.5 泛型

泛型：不提前指定具体类型，而是在使用时才指定类型

#### 2.5.1 泛型函数

```typescript
type IGetArr = <T>(target: T) => T[];
```

```typescript
function getRepeatArr(target) {
  return new Array(10).fill(target);
}

type IGetArr = <T>(target: T) => T[];

const GetArr: IGetArr = getRepeatArr;

const stringArr = GetArr("Hello");
const numArr = GetArr(10);
```

![image-20220205215955096](https://s2.loli.net/2022/02/05/BcdV7AfF6yPEuk4.png)

#### 2.5.2 泛型接口

```typescript
interface IX<T, U> {
  key: T;
  val: U;
}
```

#### 2.5.3 泛型类

```typescript
class IX<T> {
  key: T;
}
```

#### 2.5.4 泛型别名

```typescript
type ITypeArr<T> = Array<T>;
```

#### 2.5.5 泛型约束

```typescript
type IGetRepeatStringArr = <T extends string>(target: T) => T[]; // 限制泛型必须符合字符串(demo，实际上使用应该不会只限制成字符串这种)
```

![image-20220205220434379](https://s2.loli.net/2022/02/05/Q5FSaX9o2qbGyON.png)

#### 2.5.6 泛型参数默认

```typescript
type IGetRepeatArr<T = number> = (target: T) => T[];
```

![image-20220205220835362](https://s2.loli.net/2022/02/05/7vglhHGkBdNxfyV.png)

### 2.6 其他类型

```typescript
// 空类型，表示勿赋值
type IEmptyFunction = () => void;

// 任意类型，是所有类型的子类型
type IAnyArr = any;

// 枚举类型：支持枚举值到枚举名的正反映射
enum EnumExample {
  add = "+",
  mul = "*",
}

EnumExample["add"] === "+";
EnumExample["+"] === "add";

// 指定固定值
type IOddNum = 1 | 3 | 5 | 7 | 9; // IOddNum必须是1、3、5、7、9中的数
```

## 3. 高级类型

### 3.1 联合/交叉类型

首先，假设一个情景，你有收藏书籍的兴趣，但是只收藏历史书和故事书，而且历史书需要记录历史范围，而故事书则是需要记录主题。

![image-20220206164054320](https://s2.loli.net/2022/02/06/Sq3fNnFa1KeyHE9.png)

为书籍列表编写类型(如下图所示)：可以发现类型声明繁琐，存在较多变量

![image-20220206164140324](https://s2.loli.net/2022/02/06/2cCJBaNAyW4Kbfq.png)

<b style="color: red">通过联合/交叉类型可以实现优化</b>

- **联合类型**：IA | IB，表示一个值可以是 IA 类型或 IB 类型
- **交叉类型**：IA & IB，多种类型叠加成一种类型，包含了所需的所有类型的特性

优化后：

![image-20220206164516232](https://s2.loli.net/2022/02/06/7PMQzpVkjwORFhn.png)

### 3.2 类型保护和类型守卫

#### 3.2.1 类型保护

```typescript
function reverse(target: string | Array<any>) {
  // 作用：把字符串或数组反转

  if (typeof target === "string") {
    // typeof类型保护
    return target.split("").reverse().join("");
  }

  if (target instanceof Object) {
    // instance类型保护(只考虑字符串和数组两种情况)
    return terget.reverse();
  }
}
```

联合/交叉类型中的书籍只有历史和故事两种类型，所以可以实现自动类型推断

```typescript
function logBook(book: IBookList) {
  if (book.type === "history") {
    // 联合类型 + 类型保护 = 自动类型推断
    console.log(book.range);
  } else {
    console.log(book.theme);
  }
}
```

#### 3.2.2 类型守卫

<b style="color: red">访问联合类型时，仅能访问联合类型中的交集部分。</b>所以下面的例子会报错

```typescript
interface IA {
  a: 1;
  a1: 2;
}

interface IB {
  b: 1;
  b1: 2;
}

function log(arg: IA | IB) {
  // 报错，因为类型IA | IB上不存在属性a、a1、b、b1。访问联合类型时，仅能访问联合类型中的交集部分。
  if (arg.a) {
    console.log(arg.a1);
  } else {
    console.log(arg.b1);
  }
}
```

通过类型守卫则可以解决问题。

```typescript
interface IA {
  a: 1;
  a1: 2;
}

interface IB {
  b: 1;
  b1: 2;
}

// 类型守卫：定义一个函数，返回值是一个类型谓词
function getIsIA(arg: IA | IB): arg is IA {
  return !!(arg as IA).a; // as是类型断言的语法。(arg as IA).a表示存在a，则一定是IA。
  // !!是用于转换成为boolen类型
}

function log(arg: IA | IB) {
  if (getIsIA(arg)) {
    console.log(arg.a1);
  } else {
    console.log(arg.b1);
  }
}
```
