---
title: rem适配布局
date: 2021-09-27 09:12:18
categories: "前端"
tags:
	- 布局
---

# rem 适配布局

## rem 单位

rem 是根 em(root em)的缩写，是相对于根元素(html 元素)的字体大小。

rem 作用于非根元素字体大小时，相对于根元素字体大小；rem 作用于根元素字体大小时，相对于其初始字体的大小。

比如，根元素（html）设置 font-size = 12px;非根元素设置 width: 2rem;则换成 px 表示就是 24px。

整个页面只有一个 html，通过修改 html 的文字大小，可以很好的控制页面中元素的大小。

## 媒体查询

### 介绍

媒体查询(Media Query)是 CSS3 新语法。

1. 使用@media 查询，可以针对不同的媒体类型定义不同的样式；
2. @media 可以针对不同的屏幕尺寸设置不同的样式；
3. 重置浏览器大小的过程中，页面也会根据浏览器的宽度和高度重新渲染页面；
4. 苹果手机、Android 手机平板等设备都用得到多媒体查询。

### 语法规范

```
    @media mediatype and|not|only (media feature) {
    	CSS-Code;
    }

```

1. 用 media 开头
2. media type 媒体类型
3. 关键字 and、not、only
4. media feature 媒体特性必须有小括号包含

#### media type 查询类型

将不同的终端设备划分成不同的类型。称为媒体类型。

1. all：用于所有设备
2. print：用于打印机和打印预览
3. screen：用于电脑屏幕、平板、手机等

#### 关键字

关键字将媒体类型和媒体特性连接起来作为媒体查询的条件。

1. and：相当于"且"的意思，即当媒体类型和媒体特性都符合条件才起作用；
2. not：相当于"非"的意思，排除某个媒体类型，可以省略
3. only：指定某个特定的媒体类型，可以省略

#### 媒体特性

每个媒体类型都具有不同的特性，根据不同媒体类型特性来设置不同的展示风格。

常用：width，max-width，min-width

例子：

```
    <!DOCTYPE html>
    <html lang="zh-CN">

    <head>
    	<meta charset="UTF-8">
    	<meta http-equiv="X-UA-Compatible" content="IE=edge">
    	<meta name="viewport" content="width=device-width, initial-scale=1.0">
    	<title>Document</title>
    </head>
    <style>
    	@media screen and (max-width: 800px) {
    		body {
    			background-color: pink;
    		}
    	}

    	@media screen and (max-width: 500px) {
    		body {
    			background-color: purple;
    		}
    	}
    </style>

    <body>
    </body>

    </html>

```

当页面宽度小于等于 500px 时，页面背景色为紫色；当页面宽度大于 500px 小于等于 800px 时，页面背景色为粉色。

多个媒体特性示例：

```
    @media screen and (min-width: 540px) and (max-width: 969px) {
    	body {
    			background-color: green;
    	}
    }

```

### 引入资源

当样式比较多时，我们可以针对不同的媒体使用不同的样式表。

原理：直接在 link 中判断设备的尺寸，然后引用不同的 css 文件。

#### 语法规范

`<link rel="stylesheet" href="mystylesheet.css" media="media and|not|only (media feature)">`

示例：

```
    <!DOCTYPE html>
    <html lang="zh-CN">

    <head>
    	<meta charset="UTF-8">
    	<meta http-equiv="X-UA-Compatible" content="IE=edge">
    	<meta name="viewport" content="width=device-width, initial-scale=1.0">
    	<title>Document</title>
    </head>
    <link rel="stylesheet" href="css/style320.css" media="screen and (min-width: 320px)">
    <link rel="stylesheet" href="css/style640.css" media="screen and (min-width: 640px)">

    <body>
    	<div>1</div>
    	<div>2</div>
    </body>

    </html>

```

## less

### CSS 弊端

CSS 是非程序是语言，没有变量、函数、作用域等概念。

1. 需要大量看似没有逻辑的代码，CSS 冗余度较高
2. 不方便维护及扩展，不利于复用
3. 没有良好的计算能力

### 介绍

Less（Leaner Style Sheets 的缩写）是一门 CSS 扩展语言，成为 CSS 预处理器。

常见的 CSS 预处理器： Sass、Less、Stylus

Less 作为 CSS 一种形式上的扩展，没有减少 CSS 的功能，而是在现有的 CSS 语法上，加入程序式语言的特性。

引入了变量、Mixin（混入）、运算以及函数等功能。大大简化了 CSS 编写，并且降低了 CSS 的维护成本，可以实现用更少的代码完成更多的事。

总结：Less 是一门 CSS 预处理 1 语言，它扩展了 CSS 的动态特性。

[Less 官网](http://lesscss.cn/)

### 使用

过程：

1. 新建后缀名为 less 的文件，书写 less 语句
2. less 编译生成 css 文件
3. 引入 CSS 文件

#### Less 变量

没有固定的值，可以改变的。颜色和数值经常使用。

`@变量名:值;`

<b>变量命名规范</b>

1. 必须以@为前缀
2. 不能包含特殊字符
3. 不能以数字开头
4. 区分大小写

使用示例：

![](https://pic.imgdb.cn/item/6102825a5132923bf80bbd53.jpg)

#### Less 编译

Less 包含一套自定义的语法和一个解析器，用户根据这些语法定义自己的样式规则，这些规则最终会通过解析器，编译生成对应的 CSS 文件。

VSCode 插件：Easy LESS 插件可以实现

安装插件后，重新加载 VSCode，之后只要保存 less 文件，会自动生成对应的 CSS 文件。

#### Less 嵌套

实例：
CSS：

```
    .x .y {
      background-color: #fff;
    }


```

Less:

```
    .x {
      .y {
    	background-color: #fff;
      }
    }

```

生成的 CSS 样式和上面的是一样的

<br>

<p style = "color: red">如果遇到交集|伪类|伪元素选择器：<br />如果内层选择器前面没有&符号，则被解析为父选择器的后代；如果有，责备解析为父元素自身或父元素的伪类。</p>

CSS:

```
    .header a:hover {
      color: blue;
    }
    .header a::before {
      content: "";
    }

```

Less:

```
.header a {
  &:hover {
color: blue;
  }
}
.header a {
  &::before {
content: "";
  }
}

```

#### Less 运算

和其他语言运算大同小异

1. 运算符左右两侧各自添加一个空格；
2. 多个数参与运算，如果只有一个数有单位，则最后就是以这个单位为准；
3. 多个数参与运算，如果多个数有单位，则最后就是以第一个单位为准；

用上"/"的式子可能会不起作用，甚至会报错，需要用小括号包住整个式子或者除法的式子。

另外，Less 注释为 `//注释内容`,并且不会出现在对应的 CSS 中。

Less:

```css
@border: 5px;
@size: 50px;
div {
  width: 200px + 50;
  height: 200px - 50;
  border: @border solid #ccc;
  margin: 100px auto;
}
p {
  width: (82rem / @size + 50px);
  height: 82 / 50rem;
  // 没有实现运算, height: 30 + 82 / 50rem; 会报错
  // 改成 height: (30 + 82 / 50rem)
  // 或 height: 30 + (82 / 50rem);
  font-size: 82 + 15px + 3rem;
}
```

生成的 CSS:

```css
div {
  width: 250px;
  height: 150px;
  border: 5px solid #ccc;
  margin: 100px auto;
}
p {
  width: 51.64rem;
  height: 82 / 50rem;
  font-size: 100px;
}
```

##

## rem 适配方案

### 目标

让一些不能等比自适应的元素，当设备尺寸发生改变时,等比例适配当前设备。

### 实现

1. 使用媒体查询根据不同设备按比例设置 html 的字体大小
2. 页面元素使用 rem 做单位。这样的话，当 html 字体大小变化（即不同设备）时，元素尺寸也会发生变化，从而达到等比例缩放的适配。

### rem 实际开发适配方案

1. 首先选一套标准尺寸 750 为准
2. 动态设置 html 标签 font-size 大小
3. 元素大小取值方法

① 页面元素的 rem 值=页面元素值（px）/(屏幕宽度/划分的分数)
② 屏幕宽度/划分的份数就是 html&nbsp;&nbsp;font-size 大小
③ 页面元素的 rem 值=页面元素值（px）/html&nbsp;&nbsp;font-size 大小

@import 导入的 css 文件名：可以把一个样式文件导入到另一个样式文件中。

```
    @import "common";
```

和 link 相似，区别：link 是把一个样式文件引入到 html 页面里面。

示例：

![](https://pic.imgdb.cn/item/6107ef055132923bf88d211c.jpg)

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <style>
    @media screen and (min-width: 320px) {
      html {
        font-size: 21.33px;
      }
    }

    @media screen and (min-width: 750px) {
      html {
        font-size: 50px;
      }
    }

    div {
      width: 2rem;
      height: 2rem;
      background-color: pink;
    }
  </style>

  <body>
    <div></div>
  </body>
</html>
```

rem 适配方案 1：

1. less
2. 媒体查询
3. rem
4. 插件 easy less

rem 适配方案 2：

1. flexible.js
2. rem
3. 插件 cssrem

参考：
pink 老师前端入门教程
