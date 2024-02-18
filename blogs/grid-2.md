---
title: Grid布局 容器属性(二)
categories: 前端
date: 2022-07-05 20:40:19
tags:
    - CSS
    - 布局
---



# Grid布局 容器属性(二)

## 基础代码

HTML(`box`的子元素可能会增加、减少)

```html
<div class="box">
    <div></div>
    <div></div>
    <div></div>
    <div></div>
    <div></div>
    <div></div>
</div>
```

CSS

```css
.box {
    width: 300px;
    height: 300px;
}

.box div:nth-child(odd) {
    background-color: pink;
}

.box div:nth-child(even) {
    background-color: purple;
}
```

### `row-gap`、`column-gap`属性

`row-gap`设置行间距，`column-gap`设置列间距。

```css
grid-template-columns: repeat(3, 1fr);
row-gap: 10px;
column-gap: 20px;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c4a3ee034c134aeb8e213c34c9bf482b~tplv-k3u1fbpfcp-zoom-1.image)

### `gap`属性

`gap`属性是`row-gap`和`column-gap`的简写形式。

```css
gap: <row-gap> <column-gap>;
```

```css
gap: 10px 20px;
```

结果和上图一样。

如果只有一个值，那么行间距、列间距都是该值。

```css
gap: 20px;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1571d6fa80e34368a2f75eb6a5cc9a40~tplv-k3u1fbpfcp-zoom-1.image)

### `grid-auto-flow`属性

`grid-auto-flow`属性指定在网格中被自动布局的元素怎样排列。默认值是`row`，即`先行后列`。

```css
grid-template-rows: 50% 50%;
grid-template-columns: repeat(3, 1fr);
grid-auto-flow: row;    /* 这里有没有都是一样的结果 */
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3a9c12f285284d079ff5ed17d3036d35~tplv-k3u1fbpfcp-zoom-1.image)

设置成`column`的话，就会按`先列后行`的顺序来排列。

```css
grid-template-rows: 50% 50%;
grid-template-columns: repeat(3, 1fr);
grid-auto-flow: column;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e4c22f6c117445959c4aa945a7f3dad3~tplv-k3u1fbpfcp-zoom-1.image)

下面还需要讲一下设置`row`或`column`的同时添加`dense`的情况。加了`dense`表示尽可能紧密填满。

```css
grid-template-rows: repeat(3, 1fr);
grid-template-columns: repeat(3, 1fr);
grid-auto-flow: row;
```

```css
.box div:nth-child(3) {
    /* 后续将项目属性时会细讲。 */
    /* auto: 表示该项目对网格项目没有任何贡献。实际没有它也行。暂时找不到必须要的理由 */
    /* span: 表示跨越，即占多少个格 */
    grid-column: auto / span 2;
}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e0e399150354c7f966f3d70b480c94e~tplv-k3u1fbpfcp-zoom-1.image)

如果需要紧密填满的话，只需要将`grid-auto-flow`属性变成`row dense`即可。之后4就会往上移，补空位，5、6也依次补上去。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/072c6d45c5804fcaa0d78a74ff0fed87~tplv-k3u1fbpfcp-zoom-1.image)

### 单元格内容排列

和单元格排列有关的主要有两个属性。

* `justify-items`：设置单元格内容的水平位置
* `align-items`：设置单元格内容的垂直位置

它们的取值都是一样的：

* `start`: 对齐单元格的起点
* `end`: 对齐单元格的终点
* `center`：单元格内容居中
* `stretch`: 拉伸占满单元格(默认值)

#### `justify-items`属性

上面已经简单介绍过了，其实和`flex`的差不太多，接下来来一下实例加深一下印象。

box元素的CSS基础代码加一下下面的内容。(方便体验)

```css
grid-template-rows: repeat(2, 20%);
grid-template-columns: repeat(3, 20%);
background-color: skyblue;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fcb5d061a6734d349030e3e0e926eab4~tplv-k3u1fbpfcp-zoom-1.image)

**`stretch`**: 效果和上图一样。因为默认值就是`stretch`

```css
justify-items: stretch;
```

**`start`**：

```css
justify-items: start;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1bf571d713c442429e27f33675995e60~tplv-k3u1fbpfcp-zoom-1.image)
注意：不再是`stretch`之后，单元格内容的大小就不会是单元格本身的大小了，而是真正内容的大小。例如，上面的例子中，没有设置宽度，真正内容大小就是文字的大小。

**`end`**：

```css
justify-items: end;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70cef98571ca433aa2ae279022c7ef6d~tplv-k3u1fbpfcp-zoom-1.image)

**`center`**：

```css
justify-items: center;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ede98852a490477fa20ae8a1b57113eb~tplv-k3u1fbpfcp-zoom-1.image)

#### `align-items`属性

**`start`**:

```css
align-items: start;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e7e94f23234f49cfbf70ae99ea0583bd~tplv-k3u1fbpfcp-zoom-1.image)

**`end`**:

```css
align-items: end;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d3d498829ae6470393b4ac799e8a6452~tplv-k3u1fbpfcp-zoom-1.image)

**`center`**:

```css
align-items: center;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/51578d5573cb4e04b4a8667cbd7667be~tplv-k3u1fbpfcp-zoom-1.image)

#### `place-items`属性

`place-items`属性是`align-items`和`justify-items`的简写形式。

语法：

```css
place-items: <align-items> <justify-items>;
```

示例：

```css
place-items: start center;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b30b62a94dea40e8b28213adb0dc7714~tplv-k3u1fbpfcp-zoom-1.image)
水平方向居上，垂直方向居中。

需要水平垂直居中只需要值都设置为`center`即可，**如果省略第二个值，则第二个值会等于第一个值**。也就是说水平垂直居中只需要`place-items: center;`即可。

### 整体内容排列

box元素的CSS基础代码还是要加一下下面的内容。(方便体验)

```css
grid-template-rows: repeat(2, 20%);
grid-template-columns: repeat(3, 20%);
background-color: skyblue;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/720b992735d64dc9a6a2a31caffe3d31~tplv-k3u1fbpfcp-zoom-1.image)

和单元格排列有关的主要有两个属性。

* `justify-content`：设置整体内容的水平位置
* `align-items`：设置整体内容的垂直位置

它们的取值都是一样的：

* `start`、`end`、`center`、`stretch`和单元格排列部分的一样，只是对齐的不再是单元格，而是容器了。
* `space-around`：每个项目两侧的间隔相等，项目之间的间隔会比容器边框的间隔大一倍。
* `space-between`：项目与项目的间隔相等，项目与容器边框之间没有间隔
* `space-evenly`：项目与项目、项目与容器边框之间的间隔都相等。

#### `justify-content`属性

简单举几个实例，基本看看结果+看看概念就懂了(真正懂需要开发时经常使用)
`center`:

```css
justify-content: center;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8f5ff314dc54468a90ba0e0ccb0de199~tplv-k3u1fbpfcp-zoom-1.image)

`space-around`:

```css
justify-content: space-around;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2364907b9d8a48b59627641e529aef62~tplv-k3u1fbpfcp-zoom-1.image)

`space-between`:

```css
justify-content: space-between;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c709503dd5f4e478e749e8a63ad9107~tplv-k3u1fbpfcp-zoom-1.image)

`space-evenly`:

```css
justify-content: space-evenly;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c5c5f6baa3a4ccd80ed495b1ad61ca2~tplv-k3u1fbpfcp-zoom-1.image)

#### `align-content`属性

和`justify-content`属性一样，只是从水平方向变成了垂直方向。

#### `place-content`属性

`place-content`属性是`align-content`和`justify-content`的简写形式。

语法：

```css
place-content: <align-content> <justify-content>;
```

示例：

```css
place-content: space-between center;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/08fdf0b11b6d495cb195e65e651ade60~tplv-k3u1fbpfcp-zoom-1.image)

#### `start`和`stretch`的区别

用上面的例子测试，如果使用`start`和`stretch`,会发现它们的结果一样。

这是因为我们的项目大小已经固定好了，如果变成`auto`的话，就能看出`start`和`stretch`的区别了。

`stretch`: 会拉伸占满容器

```css
grid-template-rows: repeat(2, 20%);
grid-template-columns: repeat(3, auto);
justify-content: stretch;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e885391d3454d8dbb432af227cf536b~tplv-k3u1fbpfcp-zoom-1.image)

`start`：真正内容的大小，不会拉伸。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/37a5def12ea44a6e9b419bec4f4fc8ff~tplv-k3u1fbpfcp-zoom-1.image)

参考链接：

* [CSS Grid 网格布局教程](http://www.ruanyifeng.com/blog/2019/03/grid-layout-tutorial.html)
* [MDN](https://developer.mozilla.org/zh-CN/)
