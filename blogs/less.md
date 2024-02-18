---
title: Less笔记
date: 2021-12-05 13:20:38
keywords: Less
categories: 前端
tags:
  - Less
---

# Less 笔记

_Less_ 是一门 CSS 预处理语言,它扩展了 CSS 语言,增加了变量、Mixin、函数等特性。_Less_ 可以运行在 Node 或浏览器端。

开始之前，先介绍一个 vscode 插件，可以实现根据 less 编译生成对应的 css

![](https://pic.imgdb.cn/item/61a7850c2ab3f51d91ea0502.jpg)

## 1. 变量

通过@变量名定义，也是通过@变量名使用，不是用"="赋值，而是和 css 的样式一样用":"赋值

```less
@width: 100px;

div {
  width: @width;
}
```

编译为：

```css
div {
  width: 100px;
}
```

## 2. 混合

混合是一种将一组属性从一个规则集混入另一个规则集的方法。

```less
.position {
  left: 0;
  right: 0;
}

.box div {
  position: fixed;
  .position();
}
```

编译为:

```css

```

## 3. 嵌套

<b style="color: red">个人最常用的 Less 语法。</b>非常方便，模仿 HTML 的组织结构，调试的时候会感觉清晰明了。

```less
.box {
  color: #fff;
  div {
    background-color: pink;
    span {
      font-size: 20px;
    }
  }
}
```

编译后：

```css
.box {
  color: #fff;
}
.box div {
  background-color: pink;
}
.box div span {
  font-size: 20px;
}
```

可以搭配"&"运算符使用，”&“表示当前选择器的父级

```less
a {
  color: red;
  &:hover {
    text-decoration: none;
  }
}
```

编译后：

```css
a {
  color: red;
}
a:hover {
  text-decoration: none;
}
```

**@规则嵌套**

@ 规则（例如 `@media` 或 `@supports`）可以与选择器以相同的方式进行嵌套。@ 规则会被放在前面，同一规则集中的其它元素的相对顺序保持不变。这叫做冒泡

```less
div {
  width: 400px;

  @media (min-width: 768px) {
    width: 600px;
  }

  @media (min-width: 1280px) {
    width: 800px;
  }
}
```

编译后：

```css
div {
  width: 400px;
}
@media (min-width: 768px) {
  div {
    width: 600px;
  }
}
@media (min-width: 1280px) {
  div {
    width: 800px;
  }
}
```

## 4. 运算

**结果会以最左侧的有单位的操作数的单位为准**

+、-部分

```less
@width: 10px - 5em;
@height: 10em - 5px;
@lineHeight: 10 - 5px + 1em;

div {
  width: @width;
  height: @height;
  line-height: @lineHeight;
}
```

编译后：

```css
div {
  width: 5px;
  height: 5em;
  line-height: 6px;
}
```

\*、/部分：<b style="color: red">除法需要使用()包住，否则无法正常进行运算</b>

原因：进入 4.0 版本后， 除法运算符如果在括号外面，不执行除法运算，在小括号内可以看做是除法运算

```less
@width: 10px * 2em;
@height: 10em * 2px;
@lineHeight: (10 * 5px / 2em);
@color: (#224488 / 2);

div {
  width: @width;
  height: @height;
  line-height: @lineHeight;
  color: @color;
}
```

编译后：

```css
div {
  width: 20px;
  height: 20em;
  line-height: 25px;
  color: #112244;
}
```

## 5. 转义

转义允许使用任何字符串作为属性或变量值。任何`~"anything"`形式的内容都会按原样输出

```less
@min728: ~"(min-width: 768px)";
div {
  @media @min728 {
    width: 200px;
  }
}
```

编译后：

```css
@media (min-width: 768px) {
  div {
    width: 200px;
  }
}
```

## 6. 函数

例子：percentage()把小数转换为百分比, length()得到数组长度

```less
@width: 0.5;
@list: "banana", "tomato", "potato", "peach";
@n: length(@list);

.class {
  width: percentage(@width);
  height: @n * 200px;
}
```

编译后：

```css
.class {
  width: 50%;
  height: 800px;
}
```

## 7. 映射

```less
#colors() {
  primary: blue;
  secondary: green;
}

.button {
  color: #colors[primary];
  border: 1px solid #colors[secondary];
}
```

编译后：

```css
.button {
  color: blue;
  border: 1px solid green;
}
```

## 8. 作用域

会先在本地查找有无变量，如果找不到，则会去到父级作用域中查找

```less
@color: red;
.box {
  @color: blue;
  div {
    @color: purple;
    color: @color;
  }
}
```

编译后：

```css
.box div {
  color: purple;
}
```

找不到:

```less
@color: red;
.box {
  div {
    color: @color;
  }
}
```

编译后：

```css
.box div {
  color: red;
}
```

## 9. 注释

和 js 一样，有两种形式的注释方式，`//`和`/**/`

其中，`/**/`形式的注释编译后，也会出现在生成的 css 中，而`//`形式的则不会出现在生成的 css 中

```less
/* 测试1 */
// 测试2
@color: red;
.box {
  div {
    color: @color;
  }
}
```

编译后：

```css
/* 测试1 */
.box div {
  color: red;
}
```

## 10. 导入

如果导入的文件是`.less`扩展名，则可以把扩展名去掉

要导入的 test.less 文件

```less
div {
  color: red;
}
```

index.less 文件

```less
@import "./test";
```

编译后：

```css
div {
  color: red;
}
```

学习链接：[Less 快速入门 | Less.js 中文文档 - Less 中文网 (bootcss.com)](https://less.bootcss.com/)
