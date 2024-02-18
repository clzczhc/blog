---
title: HTML、CSS温故而知新
date: 2022-01-17 16:51:53
keywords: 前端 HTML CSS 青训营
categories: "前端"
tags:
  - 青训营
  - HTML
  - CSS
---

# HTML、CSS 温故而知新

参加字节跳动的青训营时写的笔记。这部分是韩广军老师讲的课。

**前端**：

![image-20220117162624413](https://s2.loli.net/2022/01/17/QxptcwgIJ9RW4bh.png)

前端需要关注的东西：

- **功能**
- 美观
- 安全
- 兼容
- 体验
- 性能
- 无障碍

## 1. HTML

用于创建网页的标准标记语言

### 1.1 HTML 语法

- 标签和属性不区分大小写，但是推荐小写
- 部分空标签可以不闭合，如 input、meta
- 属性值推荐使用**双引号包裹**
- 属性值为 true 时，可以省略属性值，如 required、readonly

### 1.2 HTML 标签

**h1-h6**：h1 一级标题，h6 六级标题

**ol**(有序列表)：

```html
<ol>
  <li>A</li>
  <li>B</li>
  <li>C</li>
</ol>
```

**ul**(无序列表)：

```html
<ul>
  <li>1</li>
  <li>2</li>
  <li>3</li>
</ul>
```

**dl**(定义列表)：

```html
<h3>西游记</h3>
<dl>
  <dt>作者</dt>
  <dd>吴承恩</dd>

  <dt>创作年代</dt>
  <dd>明代</dd>

  <dt>借用人</dt>
  <dd>clz</dd>
  <dd>czh</dd>
</dl>
```

dt：标题， dd：具体描述， dt 和 dd 是多对多的关系

**a**(链接)：

- href：链接的地址

- target="\_blank"：以新标签的形式打开

**img**：

- alt：当加载失败或不加载图片时的替代文字

**input**：

- type="range"：输入范围

- type="number"：输入数字，可以添加 min，max 设置范围

- type="date"：输入日期

- type="checkbox"：多选按钮

- type="radio"：单选按钮，通过 name 的属性值实现互斥

**textarea**：多行文本框

**引用**：

- blockquote：块级引用(长引用， 如引用一段话)

- cite：短引用(如书名)

- q：短引用(具体内容)

**强调**：

strong：粗体强调标签，强调，表示内容的重要性

em：斜体强调标签，更强烈的强调，表示内容的强调点

### 1.3 语义化

​ HTML 中的元素、属性及属性值都拥有某种含义，如有序列表用 ol，无序列表用 ul.

![image-20220117162740803](https://s2.loli.net/2022/01/17/fkz5BJ8tZxCVcXv.png)

语义化好处：

1. 了解每个标签和属性的含义
2. 思考什么标签最适合描述这个内容
3. 不使用可视化工具生成

### 1.4 src 和 href 的区别

​ src 指向的内容会嵌入到文档当前标签所在的位置，而 href 是用于建立这个这个标签与外部资源之间的关系

## 2. CSS:

用来定义页面元素的样式(如文字的大小、颜色)

### 2.1 使用 css 的三种形式

- 外链

  ```html
  <link rel="stylesheet" href="./index.css" />
  ```

- 嵌入

  ```html
  <style>
    p {
      color: red;
    }
  </style>
  ```

- 内联

  ```html
  <p style="color: red">test</p>
  ```

![image-20220117162822660](https://s2.loli.net/2022/01/17/aAnj92c6o7g1r3t.png)

### 2.2 选择器

[css 选择器](https://13535944743.github.io/2021/10/15/css-selector/)

![image-20220117162857197](https://s2.loli.net/2022/01/17/vSikdbXoV6TZ89t.png)

**选择器的特异度**：选择器的特异度高的会覆盖特异度低的样式

```css
nav a {
  color: red;
}

a {
  color: pink;
}
/*结果会是红色*/
```

` #nav .list li a:link`：

| id  | (伪)类 | 标签 |
| --- | ------ | ---- |
| 1   | 2      | 2    |

` .box ul.links a`：

| id  | (伪)类 | 标签 |
| --- | ------ | ---- |
| 0   | 2      | 2    |

### 2.3 字体

#### 2.3.1 字体族 font-family

![image-20220117162938942](https://s2.loli.net/2022/01/17/6BJQEheD2MIuNUs.png)

font-family 使用建议:

- 字体列表最后加上通用字体族
- 英文字体放在中文字体前面

#### 2.3.2 字体大小 font-size

- 关键字：small、medium、large
- 长度：px、em
- 百分比：相对于父元素字体大小

#### 2.3.3 字体粗细 font-weight

font-weight: 100-900

normal(400), bold(700)

#### 2.3.4 行高 line-height

用于设置多行元素的空间量

![image-20220117163300668](https://s2.loli.net/2022/01/17/D5zL6wIMqthrbAT.png)

如果 line-height 的值没有单位，则是 font-size\*line-height 的值

#### 2.3.5 简写

` font: style weight size/height family`

例子：

```css
h1 {
  font: bold 16px/2 Arial, Helvetica;
}
p {
  font: 16px serif;
}
```

### 2.4 继承

[CSS 属性取值过程](https://cdpn.io/webzhao/debug/xxXyzRd)

某些属性会自动继承父元素的计算值，除非显式指定一个值。

```html
<div style="color: red">
  <span>123</span>
  <span>456</span>
  <span style="color: blue">789</span>
</div>
```

在 CSS 中以 text-、font-、line- 开头的属性都是可以继承的

显示继承：inherit

```css
* {
  color: inherit;
}

html {
  color: red;
}

.special {
  color: blue;
}
```

### 2.5 盒模型

- 标准盒模型：width 指 content 的宽度(即内容的宽度)，`box-sizing`为 content-box
- 怪异盒模型(IE 盒模型)：width 指 content 的宽度 + 左右 padding 值 + 左右 border 值，`box-sizing`为 border-box

![img](https://s2.loli.net/2022/01/17/cJDzEYHrXCdViva.png)

![image-20220117123750994](https://s2.loli.net/2022/01/17/MPuIEOHewyDAd3N.png)

上两张图片来源：https://www.jianshu.com/p/7dadcc458410

### 2.6 块级元素与行级元素的区别

| 块级                 | 行级                            |
| -------------------- | ------------------------------- |
| 不和其他盒子并列摆放 | 可以和其他行级盒子一起放到一行  |
| 适应所有的盒模型属性 | 盒模型中的 width、height 不适用 |

### 2.7 行级排版上下文(IFC)和块级排版上下文(BFC)

#### 2.7.1 行级排版上下文(IFC)

- Inline Formatting Context
- **只包含行级盒子**的容器会创建一个 IFC
- IFC 内的排版规则
  - 盒子在一行内平行摆放
  - 一行放不下时，换行显示
  - text-align 决定一行内盒子的水平对齐
  - vertical-align 决定一个盒子在行内的垂直对齐
  - 避开浮动(float)元素

#### 2.7.2 块级排版上下文(BFC)

- BlockFormatting Context
- 某些容器会创建一个 BFC
  - 根元素
  - 浮动、绝对定位、inline-block
  - Flex 子项和 Grid 子项
  - overflow 值不为 visible 的块盒
  - display: flow-root;

### 2.8 Flex 布局

[Flex 布局 | 赤蓝紫 (13535944743.github.io)](https://13535944743.github.io/2021/07/29/Flex-Layout/)

之前学习时写的笔记。

### 2.9 Grid 布局

- display: grid 使元素生成一个块级的 Grid 容器
- 使用 grid-template 相关属性将容器划分为网格
- 设置每一个子项占哪些行/列

暂时只是初略了解，之后还是得正式学
