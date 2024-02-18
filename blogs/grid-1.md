---
title: Grid布局 容器属性(一) `grid-template`系列属性
categories: 前端
date: 2022-07-05 20:39:22
tags:
    - CSS
    - 布局
---



# Grid布局 容器属性(一) `grid-template`系列属性

## 前言

如果学会语法了，想要类似刷题增加一点印象的话，可以去[GRID GARDEN](http://cssgridgarden.com/)玩一下游戏，不过比较简单。

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

## 容器属性

### `display`属性

设置`grid`布局。

` display: grid;`

```html
<div class="box">
    <div></div>
    <div></div>
    <div></div>
</div>
```

```css
.box {
    display: grid;
    width: 300px;
    height: 300px;
}
```

![image-20220616223659841](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0141122b66d74317943a3bac73be811a~tplv-k3u1fbpfcp-zoom-1.image)

现在的效果是，子元素会平分行。

### `grid-template`系列属性

* `grid-template-rows`：定义每一列的行高
* `grid-template-columns`：定义每一列的列宽

```css
grid-template-rows: 50px 100px;
grid-template-columns: 50px 100px 150px;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9421ae729a6549d9bf6b1b31106b8163~tplv-k3u1fbpfcp-zoom-1.image)
第一行50px，第二行100px；第一列50px，第二列100px，第三列150px。

**注意**：

* 如果只定义行高，没有定义到的行，会平分剩余高度。(剩余高度为0，则后续的行高都是0)
  
  ```css
  grid-template-rows: 50px 100px;
  ```
  
  ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d535ac7c022d4378a8fa6077ff3227bd~tplv-k3u1fbpfcp-zoom-1.image)

* 如果自定义列宽，也是平分剩余高度。
  
  ```css
  grid-template-columns: 50px 100px 150px;
  ```
  
  ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ba994e11e1b4cb283c09bb4f8054879~tplv-k3u1fbpfcp-zoom-1.image)

#### `repeat()`

我们还可以使用`repest()`来简写重复值。第一个参数是重复的次数，第二个是要重复的值。

```css
grid-template-rows: repeat(2, 50%);
grid-template-columns: repeat(3, 33.3%);
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc22e67b8e034c03b4aeb36cb43af2d7~tplv-k3u1fbpfcp-zoom-1.image)

重复的值也可以是一个模式。

```css
grid-template-columns: repeat(2, 20px 40px 60px);
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f8103d72455e4dd18e2c4997686c1523~tplv-k3u1fbpfcp-zoom-1.image)

1、2、3列是20px、40px、60px，4、5、6列还是20px、40px、60px

另外，当我们直接使用`repeat()`溢出容器也是不会自动换行的。

```css
grid-template-columns: repeat(4, 100px);
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ffd0e6caa287423db231c1455cd02bba~tplv-k3u1fbpfcp-zoom-1.image)

这时候，我们可以使用`auto-fill`关键字，可以实现容纳尽可能多的单元格。

```css
grid-template-columns: repeat(auto-fill, 100px);
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/472b98fd6e5f46f5837733cdc6fa71ad~tplv-k3u1fbpfcp-zoom-1.image)

#### `fr`关键字

`fr`是`fraction`的缩写，代表片段。如果两列的宽度分别是`1fr`和`2fr`，那么第二列是第一列的两倍。

```css
grid-template-rows: 50% 50%;
grid-template-columns: 1fr 3fr;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f81850483d514f9997a93ceffc729888~tplv-k3u1fbpfcp-zoom-1.image)

上面两个图可以发现后两个元素消失了，这是因为我们只定义了两行，两列，没有剩余高度了，所以后两个元素的高度是0。

`fr`关键字还可以和其他单位混用。

```css
grid-template-rows: 50% 50%;
grid-template-columns: 20px 1fr 2fr;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4cbbae933e8847229d0cfda4f35edba1~tplv-k3u1fbpfcp-zoom-1.image)

如果只有一个`fr`单位，那么此时会占满剩余空间

```css
grid-template-rows: 50% 50%;
grid-template-columns: 20px 1fr;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c3e4b19b5bb940619d230333da4fcd82~tplv-k3u1fbpfcp-zoom-1.image)

#### `auto`关键字

`auto`关键字表示占满剩余空间。
所以只有一个`fr`单位的例子中，也可以使用`auto`替换，能得到同样的结果。

```css
grid-template-rows: 50% 50%;
grid-template-columns: 20px auto;
```

那么`fr`和`auto`有什么区别呢？
`fr`表示占据剩余空间的份数，所以可以有比例关系，
`auto`是分配空余空间(即占满剩余空间)。

如果`auto`遇到`fr`，会被压缩成元素真实大小，`fr`会占满剩余空间。(小弟遇到大哥)

```css
grid-template-columns: auto 50px 1fr;
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b5e758421c664737a7405421e9ea560a~tplv-k3u1fbpfcp-zoom-1.image)

#### `minmax()`

CSS函数`minmax()`定义了一个长宽范围的闭区间， 与CSS网格布局一起使用。两个参数，最小值和最大值。

### `grid-template-areas`属性

`grid-template-areas`属性没有归到上面那里是因为它有一点点特殊。它用于定义区域(划分区域)，它定义的区域需要项目(子元素)使用才会有效果。

```html
<div class="box">
    <!-- 实际上，使用划分区域这种做法的话，排版顺序可以随意 -->
    <header>header</header>
    <nav>nav</nav>
    <main>main</main>
    <footer>footer</footer>
</div>
```

```css
.box {
    display: grid;
    /* 定义区域(划分区域) */
    grid-template-areas:
        "header header"
        "nav main"
        "nav footer";
}

header {
    /* 真正使用区域(注意不是字符串) */
    grid-area: header;
    background-color: pink;
}

nav {
    grid-area: nav;
    background-color: purple;
}

main {
    grid-area: main;
    background-color: skyblue;
}

footer {
    grid-area: footer;
    background-color: orange;
}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/effbbbbb2db14091aeb4c5dd63519df1~tplv-k3u1fbpfcp-zoom-1.image)

如果有连续的行的区域一样的话，那么会合并成只有一行。

```css
grid-template-areas:
                "header header"
                "nav main"
                "nav div"
                "nav footer";
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f53536744814551bfb82efaffbbcb37~tplv-k3u1fbpfcp-zoom-1.image)

```css
grid-template-areas:
                "header header"
                "nav main"
                "nav footer"
                "nav footer";
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4160c6d05500442fa26ae63f668a00ff~tplv-k3u1fbpfcp-zoom-1.image)
上面的代码中，`footer`占据了两个区域，但是实际上只占了一个，这是因为第三行和第四行的区域完全相同，所以合并了。

**如果有区域不需要利用，使用点`.`来表示。**

```css
grid-template-areas:
                "header header header"
                "nav . main"
                "footer footer footer"
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2e974d79bfd5491fab6164f7ef5c7da3~tplv-k3u1fbpfcp-zoom-1.image)

划分区域需要注意两点：

* 不能当墙头草(跨行的同时跨列)
* 不能太贪心(同时拿两份不紧贴的)

墙头草：

```css
grid-template-areas:
    "header header header"
    "footer nav main"
    "footer footer footer"
```

贪心：

```css
grid-template-areas:
    "header header header"
    "nav . main"
    "main footer footer"
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5c9c7bd699d14cc3a8f973b87bac3b53~tplv-k3u1fbpfcp-zoom-1.image)
结果都会堆在一个位置。

点`.`不受这两点限制。

```css
grid-template-areas:
    "header header ."
    "nav . main"
    "footer . ."
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/28555658129044dd8602d013f2af3ec3~tplv-k3u1fbpfcp-zoom-1.image)

参考链接：

* [CSS Grid 网格布局教程](http://www.ruanyifeng.com/blog/2019/03/grid-layout-tutorial.html)
* [MDN](https://developer.mozilla.org/zh-CN/)
* [Grid布局中的auto和fr](http://www.qiutianaimeili.com/html/page/2020/03/2050e4ee4h3hjqc.html)
