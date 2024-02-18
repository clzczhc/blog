---
title: JS手撕(六) trim、模板字符串、千分位分隔符
categories: 前端
date: 2022-12-24 12:44:38
tags:
    - JavaScript
    - 手撕
---

# JS手撕(六)    trim、模板字符串、千分位分隔符

## trim

去掉字符串两边的空格。

```js
function myTrim(str) {
  const reg = /^\s+|\s+$/g;
  return str.replace(reg, '');
}
```

核心就是`/^\s+|\s+$/`这一段正则表达式，它会匹配字符串前后的空格，然后通过`replace()`把匹配到的部分替换成空串。

- `^`：匹配输入字符串的开始位置

- `\s`：匹配任何空白字符。包括空格符、制表符、回车符、换行符等

- `|`：`a | b`匹配`a`或 `b`

- `$`：匹配输入字符串的结束位置

测试：

```js
const str1 = '   12 3';
const str2 = '12 3   ';
const str3 = '   12 3   ';
const str4 = '12 3';

console.log(myTrim(str1));  // 12 3
console.log(myTrim(str2));  // 12 3
console.log(myTrim(str3));  // 12 3
console.log(myTrim(str4));  // 12 3
```

## 模板字符串

手动实现Vue中的模板字符串。(简单版)

原理就是通过正则表达式匹配`{{ name }}`，然后把它替换成我们传进去的对象的`name`属性值。

### 递归实现

```js
function render(templateStr, data) {
  const reg = /\{\{\s*(\w+)\s*\}\}/;    // \{：匹配{，\用于转义

  if (reg.test(templateStr)) {

    // 挑出{{}}里面的属性，如name、age
    const key = templateStr.match(reg)[1];

    // replace方法不会修改原字符串
    templateStr = templateStr.replace(reg, data[key]);

    // 递归下去，直到没有{{}}为止
    return render(templateStr, data);
  } else {
    return templateStr;
  }
}
```

上面是通过递归来实现的，每一次只替换一次模板字符串，直到没有模板字符串时，才直接返回字符串。

```js
const templateStr = `
  名字： {{ name }}
  年龄： {{ age }}
`;

const data = {
  name: 'clz',
  age: 21
};

console.log(render(templateStr, data));
```

### replace方法的函数参数

`replace`方法的第二个参数可以是函数。

函数的参数如下图所示：

从上图就可以知道，我们可以通过`p1`参数来换取模板字符串中的属性名。

并且该函数返回的值就会替换掉匹配到的字符串。

```js
function render(templateStr, data) {
  const reg = /\{\{\s*(\w+)\s*\}\}/g;    // 不用递归的方法的话，需要加g，变成全局匹配

  return templateStr.replace(reg, (match, p1) => {
    return data[p1];
  })
}
```

## 实现千分位分隔符

这一段是[战场小包](https://juejin.cn/post/7033275515880341512)的代码。个人感觉很优雅，所以就学习了一下原理(有误请指正)

```js
var str = "100000000000",
    reg = /(?=(\B\d{3})+$)/g;
str.replace(reg, ",")
```

`\d{3}`：匹配三个数字

`\B`：匹配非单词边界。用我个人的理解就是不匹配开头部分的字符串。

那么`?=`这部分是什么意思呢？

下面是引用《正则表达式中?=和?:和?!》的部分。(链接在底部)

> `exp1(?=exp2)`：前瞻，查找exp2前面的exp1
> 
> `(?<=exp2)exp1`：后顾，查找exp2后面的exp1
> 
> `exp1(?!exp2)`：负前瞻，查找后面不是exp2的exp1
> 
> `(?<!exp2)exp1`：负后顾，查找前面不是exp2的exp1
> 
> 举例：
> 
> "中国人".replace(/(?<=中国)人/, "rr") // 匹配中国人中的人，将其替换为rr，结果为 中国rr
> "法国人".replace(/(?<=中国)人/, "rr") // 结果为 法国人，因为人前面不是中国，所以无法匹配到

回到正题，测试一下完整的正则表达式的效果。

发现没看到它匹配到什么了。

在看一下上面引用的部分，发现上面没有`exp1`，加一下`exp1`看看。

没有`exp1`的时候其实也可以说是匹配上了一个空串，所以当调用`replace`把匹配的内容变成`,`的时候，就有点见缝插针一样多出了一些分隔符。

最后，再来看一下完整的代码及结果

## 参考

[2021年前端各大公司都考了那些手写题(附带代码) - 掘金](https://juejin.cn/post/7033275515880341512)

[死磕 36 个 JS 手写题（搞懂后，提升真的大） - 掘金](https://juejin.cn/post/6946022649768181774)

[GitHub - qianlongo/fe-handwriting: 手写各种js Promise、apply、call、bind、new、deepClone....](https://github.com/qianlongo/fe-handwriting)

[正则表达式中?=和?:和?!的理解_这个昵称没有被占用吧的博客-CSDN博客_正则表达式?!](https://blog.csdn.net/csm0912/article/details/81206848)
