---
title: 前端笔试之JavaScript ACM模式
categories: 前端
date: 2022-08-11 12:01:40
tags:
    - 面试(含笔试)
    - JavaScript
---

# 前端笔试之JavaScript ACM模式

## 前言

相信很多小伙伴刷算法题，都是用力扣。但是，力扣是核心代码模式，是不需要处理输入、输出的，只需要直接返回值就行。笔试、面试的时候，不一定是核心代码模式，也可能是ACM模式。如果没有了解过JavaScript的输入输出可能就寄了。(本人就试过突然ACM模式笔试，后面不知道怎么输入输出，直接重操旧业，用C++来做题😂)

> 祝看到这篇文章并且在找工作的，顺顺利利拿到心仪的`offer`。(包括本人🐶)

> JavaScript的ACM模式会有两种：`V8模式`、`Nodejs模式`。

## V8模式

### 输出

> console.log();

这个就没啥好说的了，学JavaScript还不认识这个的话，就太尴尬了。

### 读取一行输入(`read_line()`)

> 最多读取个字符，当还未达到1024个时如果遇到回车或结束符，提前结束。也有可能是`readline()`。牛客ACM模式就是`readline()`。

$\color{red}最重要的一个输入方法$，可以通过该方法得到所有情况的输入，后面讲的其他输入方法，有可能没有。比如牛客的ACM模式，使用后面的输入，都会报错：`xxx is not defined`

> `read_line()`返回输入的一行，字符串形式。需要通过`split`、`parseInt`等方法来得到真正的输入。(这里确实比C++那些要麻烦得多)

#### 练习

题目来源：[OJ在线编程常见输入输出练习场J](https://ac.nowcoder.com/acm/contest/5657)

$\color{red}题目来源是牛客，但是练习是在赛码上练的，因为牛客提供的输入API少很多。$

##### 1. A+B(1)

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207201145549.png)

```js
let line;
while(line = read_line()) {
    let nums = line.split(' ');

    let a = +nums[0];    // 将字符串转为数字
    let b = +nums[1];

    console.log(a + b);
}
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202208061711049.png)

##### 2. A+B(2)

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207201146382.png)

```js
let t;

while(t = +read_line()) {
  while(t--) {
    let line = read_line();

    let [a, b] = line.split(' ').map(Number);

    console.log(a + b);
  }
}
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202208061722099.png)

##### 3.  A+B(4)

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207201207926.png)

```js
let line;
while(line = read_line()) {
  let [n, ...nums] = line.split(' ').map(Number);

  if(n === 0) {
      break;
  }

  let sum = 0;

  for(let i = 0; i < n; i++) {
      sum += nums[i];
  }

  console.log(sum);
}
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202208061728227.png)

##### 4. A+B(5)

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202207201204912.png)

```js
let t;

while(t = +read_line()) {
  console.log()
  while(t--) {
    let [n, ...nums] = read_line().split(' ').map(Number)

    let sum = 0;
    for(let i = 0; i < n; i++) {
        sum += nums[i];
    }

    console.log(sum);
  }
}
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202208061736978.png)

### 读取一个整数(`readInt()`)

> 根据平台的不同，有可能不提供该接口，比如牛客网。

#### 练习

A+B(1)

```js
let a;
let b;

while((a = readInt()) && (b = readInt())) {
    console.log(a + b);
}
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202208061742068.png)

正所谓，**术业有专攻**，很明显，有的时候直接用`readInt()`会比`read_line()`方便很多。(当然前提是你做题的平台有提供这个接口才行)

### 读取一个浮点型(`readDouble`)

> 和`readInt()`基本一致。

```js
let a;
let b;

while((a = readDouble()) && (b = readDouble())) {
    console.log(a + b);
}
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202208061748802.png)

最多读取n个字符，当没有达到n个时，如果遇到回车或结束符，会提前结束。

### 读取n个字符(`gets(n)`)

因为`read_line()`只能读取1024个字符，所以如果输入长度大于1024个字符的字符串的话，就轮到`gets(n)`函数大显神通了。

```js
let n;

while(n = +read_line()) {
  let str = gets(n);
  console.log(str + ' - clz');
}
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202208061909479.png)



> > 如果题目备注字符串的长度范围为1到10000。
> > 
> > 那么读取输入应该是`gets(10000)`

## Nodejs模式

Nodejs输入主要有三大步骤：

1. 引入`readline`模块

2. 调用`readline.createInterface()`，创建一个`readline的接口实例`

3. 监听`line`事件，事件处理函数的参数就是`输入的行`

```js
const readline = require('readline');

const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
});
rl.on('line', function (line) {
    // 
});
```

### 练习

```js
const readline = require('readline');

const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
});
rl.on('line', function (line) {
  let nums = line.split(' ');

  if(nums && nums.length == 2) {
    // 这里加上这个条件是因为在赛码平台上，输入结束还会触发一次，line是空行
    // 吐个槽：牛客输入结束不会再触发一次。真服了，每个平台都有自己的一套规则

    let a = +nums[0];
    let b = +nums[1];

    console.log(a + b);
  }
});
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202208061856924.png)

## 小贴士

输出指定位数使用`toFixed(n)`函数（四舍五入）

```js
let a = 1 / 6;

console.log(a.toFixed(3));    // 0.167
```

## 参考

[ACMcoder OJ](https://labfiles.acmcoder.com/ojhtml/index.html#/?id=%e8%be%93%e5%85%a5api)
