---
title: HTML基础
date: 2021-08-19 14:15:29
categories: "前端"
tags:
  - HTML
---

## HTML 基础

HTML 小复习(主要是自己不常用的知识点，语义化标签)

网页三大元素：

HTML：网页的基本结构

CSS:网页的展示效果

JS：网页的功能与行为

### HTML 简介

HTML(HyperText Markup Language, 超文本标记语言)，用于构建网页基本结构及其内容的标记语言

超文本：文本中包含指向其他文本的链接

标记语言：将文本以及文本相关的其他信息结合起来，展现出关于文档结构和数据处理细节的电脑文字编码。如 HTML、XML、KML、Markdown 等。

### HTML 结构

1. HTML 文档包含多个 HTML 元素,元素具备不同的特性
2. HTML 元素 = 开始标签 + 结束标签 + 元素内容

`<p>test</p> ` 3. 部分元素是单标签元素。如 img、input、br

`<img src="images/angel.gif" alt="小天使">` 4. HTML 元素标签不区分大小写,即 `<p></p> 和<P></P>等价`,但是建议小写 5. 元素可以嵌套在其他元素中间 6. 元素可以拥有属性，属性包含有元素的额外信息，如 img 标签的 alt 属性可以用于指定图片的替换文字，即当无法正常显示图片时会显示出来的文字。

#### HTML 固定结构

```

    <!DOCTYPE html>
    <html lang="en">

    <head>
    	<meta charset="UTF-8">
    	<title>Document</title>
    </head>

    <body>
    	<div>
    		<p class="test">123</p>
    		<img src="images/angel.gif" alt="小天使">
    	</div>
    </body>

    </html>

    1. <!DOCTYPE html>:HTML文档最前面的位置，加上后会按W3C的HTML5标准来解析渲染页面
    2. <html>:根元素，包含整个页面的内容
    3. <head>:对用户不可见，包含面向搜索引擎的关键字、页面描述、字符编码声明、CSS样式等。
    4. <body>：包含能够被用户访问到的内容，包含文本、图像、视频等。

```

#### HTML 页面结构

    1. <meta charset="UTF-8">:定义文档字符编码
    2. <meta name="keywords" content="HTML">：关键字，即用搜索引擎搜索时可凭借关键字搜索到
    3. <meta name="description" content = "HTML基础">：页面描述

![](https://img-blog.csdnimg.cn/img_convert/b7429bf908fac3ce7d6d7f42d75eb2b0.png)

![](https://img-blog.csdnimg.cn/img_convert/a16f920ce9adb604a38cc097cb86dc31.png)

    4. <meta name="viewport" content="width=device-width, initial-scale=1.0">:视口标签，主要用于移动端
    5. <title>Document</title>:页面的标题，显示在浏览器标签页上
    6. <style>:CSS样式
    7. <meta "http-equiv="expires" content="31 Dec 2021">：http头部，可以向浏览器发送一些信息，如网页到期时间
    8. <link rel="shortcut icon" href="favicon.ico" />：标签页上的图标

![](https://img-blog.csdnimg.cn/img_convert/1bba743aefa9643f812700f732f1a70d.png)

    9. <link rel="stylesheet" href="css/menu.css">链接到样式表
    10. <script src="js/index.js"></script>可执行脚本，链接到js文件，也可直接在标签里写

### 常用元素

#### 块级元素

1. 占据父元素的整行，块级元素独占一行
2. 能容纳其他块级元素和行内元素（内联元素）
3. 可以控制宽高、行高、边距、边框等改变尺寸
4. 常见块级元素：div、p、h1-h6、ul、ol、dl、table、form、blockquote、address

#### 行内元素（内联元素）

1. 只占据对应标签边框所占据的空间，不独占一行
2. 只能容纳文本或其他内联元素
3. 只能通过修改水平边距、边框或行高来改变尺寸
4. 常见行内元素有：a、span、br(br 会让后面的元素从另一行开始，但它还是属于上一行)、i、em、strong、label、code、cite

#### 行内块级元素

1. 元素在行内排列，不会独占一行
2. 可以控制宽高、垂直边距、边框来改变尺寸
3. 常见行内块级元素有：img、input、td

### 语义化标签

根据内容的结构，选择合适的标签构建出便于开发者阅读的可维护性更高的代码结构，同时能够让机器更好地解析。

样例展示：

![](https://pic.imgdb.cn/item/611ccd864907e2d39c689b47.jpg)

图片出处：[html 语义化标签 例子,HTML5 语义化](https://blog.csdn.net/weixin_32496547/article/details/117719327)

#### header 标签

1. 展示介绍性信息
2. 通常包含一组介绍性或辅助导航的元素，如标题、Logo、搜索框、作者名称等
3. 不能放在 footer 标签、address 标签和另一个 header 标签内部

例子：

```

    <header>
    	<h1>HTML</h1>
    	<p> <time datetime="2021-08-15"></time> </p>
    </header>

```

#### nav 标签

1. 在当前文档中提供导航链接，如菜单、目录、索引等
2. 用来放一些热门的链接，不常用的链接一般放在 footer 标签里，而 footer 标签放在底部

```

	<nav>
		<ol>
			<li><a href="#">HTML</a></li>
			<li><a href="#">CSS</a></li>
			<li><a href="#">JS</a></li>
		</ol>
	</nav>

```

#### article 标签

1. 定义独立的内容
2. 内容本身必须是有意义的且必须是独立于文档的其余部分，如论坛帖子、新闻文章、博客、用户提交的评论、交互式组件等

```

    <article>
      <h1>Internet Explorer 9</h1>
      <p> Windows Internet Explorer 9(缩写为 IE9 )在2011年3月14日21:00 发布。</p>
    </article>

```

#### section 标签

1. 按主题将内容
2. 分组，通常会有标题
3. section 标签通常出现在文档的大纲中
4. 不要把 section 作为普通容器使用，例如，用来梅花片段样式时，用 div 更合适
5. 如果元素里面是独立的内容，可以单独存在，更适合用 article

如果只是针对一个块内容做样式化，article 和 section 二者并无区别。

section 元素用于对网站或应用程序中页面上的内容进行分块，section 元素的作用是对页面上的内容进行分块，或者说对文章进行分段；一个 section 元素通常由内容及其标题组成，通常不推荐为那些没有标题的内容使用 section 元素。<b style = "color: red">引用自下面的链接</b>

[H5 中 section 和 article 和 div 的区别](https://blog.csdn.net/qq_22855325/article/details/72877285)

#### aside 标签

主要有两种用法

1. 包含在 article 元素中作为主要内容的附属部分，其中的内容可以是与文章有关的相关资料、名词解释等。

```


	<article>
		<h1>...</h1>
		<p>...</p>
		<aside>...</aside>
	</article>

```

2. 在 article 元素之外作为页面或站点的附属信息部分。如侧边栏，其中的内容可以是友情链接、博客中的其他文章列表、广告等。

#### footer 标签

1. 描述了文档的底部区域
2. 通常包含文档的作者，著作权信息，链接的使用条款，联系信息等

例子：

```

    <footer>
      <p>Posted by: Hege Refsnes</p>
      <p><time pubdate datetime="2012-03-01"></time></p>
    </footer>

```

![](https://pic.imgdb.cn/item/611ccd194907e2d39c66d6cb.jpg)

### picture 元素

picture 元素允许我们在不同设备上显示不同的图片，一般用于响应式

picture 元素有多个 source 元素和一个 img 元素，每个 source 元素匹配不同的设备并引用不同的图像源，如果没有匹配的，就选择 img 元素中的图像。

img 是放在最后一个 source 元素之后的

```

    <picture>
      <source media="(min-width: 650px)" srcset="demo1.jpg">
      <source media="(min-width: 465px)" srcset="demo2.jpg">
      <img src="img_girl.jpg">
    </picture>

```
