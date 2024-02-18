---
title: 响应式布局
date: 2021-09-25 09:12:18
categories: "前端"
tags:
	- 布局
---

# 响应式布局

## 原理

使用媒体查询针对不同宽度的设备进行布局和样式的设置，从而适配不同设备。

| 设备               |    尺寸区间     |
| ------------------ | :-------------: |
| 手机               |     <768px      |
| 平板               | [768px, 992px)  |
| 桌面显示器         | [992px, 1200px) |
| 大桌面显示器(电脑) |    >=1200px     |

## 响应式布局容器

响应式布局需要一个父级作为布局容器，让子级元素实现变化效果

<b style="color: red">原理</b>：在不同屏幕下，通过媒体查询来改变布局容器的大小，再改变里面子元素的排列方式和大小，从而实现在不同大小的屏幕下，看到不同的页面布局和样式。

常用的响应式尺寸划分：

| 设备               |    尺寸区间     | 宽度设置 |
| ------------------ | :-------------: | :------: |
| 手机               |     <768px      |   100%   |
| 平板               | [768px, 992px)  |  750px   |
| 桌面显示器         | [992px, 1200px) |  970px   |
| 大桌面显示器(电脑) |    >=1200px     |  1170px  |

除了手机的宽度设置是 100%外，其他设备的宽度设置都会比设备的尺寸区间最小值小一点，原因是留空一点，不占满屏幕，然后容器可以居中显示。

例子：

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Myself</title>
    <style>
      * {
        margin: 0;
        padding: 0;
      }

      .container {
        height: 200px;
        background-color: pink;
        margin: 0 auto;
      }

      /*媒体查询，根据设备的宽度设置容器的宽度*/
      @media screen and (max-width: 767px) {
        .container {
          width: 100%;
        }
      }

      @media screen and (min-width: 768px) {
        .container {
          width: 750px;
        }
      }

      @media screen and (min-width: 992px) {
        .container {
          width: 970px;
        }
      }

      @media screen and (min-width: 1200px) {
        .container {
          width: 1170px;
        }
      }
    </style>
  </head>

  <body>
    <div class="container"></div>
  </body>
</html>
```

## Bootstrap

Bootstrap 是最受欢迎的 HTML、CSS 和 JS 框架，用于开发响应式布局、移动设备优先的 WEB 项目。

### 使用步骤

1. [下载 Bootstrap](https://v3.bootcss.com/)

2. 把会用到的文件夹放到要用的站点文件夹下

   ![](https://pic.imgdb.cn/item/614451aa2ab3f51d91d17745.jpg)

   另外，要防止低版本 ie 没办法用 h5、css3 的新东西，导致出问题，html 骨架需要加点料。

   ```html
   <meta charset="UTF-8" />

   <!-- 要求当前网页使用IE浏览器最高版本的内核来渲染 -->
   <meta http-equiv="X-UA-Compatible" content="IE=edge" />

   <!-- 视口标签的设置：视口的宽度和设备一致，默认的缩放比例和PC端一致，用户不能自行缩放 -->
   <meta
     name="viewport"
     content="width=device-width, initial-scale=1.0, user-scalable=0"
   />

   <!--[if lt IE 9]>
     <script src="https://oss.maxcdn.com/html5shiv/3.7.2/html5shiv.min.js"></script>
     <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
   <![endif]-->

   <title>Myself</title>
   ```

   <b style="color: red">注意！这里 if 这一段是注释，但是，注释的部分只是说浏览器不渲染，不显示被注释的代码，但是，浏览器还是回去读注释的代码的（刷新想法）</b>

   ```html
   <!-- 解决ie9以下浏览器对html5新增标签的不识别，并导致CSS不起作用的问题 -->
   <script src="https://oss.maxcdn.com/html5shiv/3.7.2/html5shiv.min.js"></script>

   <!-- 解决ie9以下浏览器对css3 Media Query的不识别 -->
   <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
   ```

3. 引入要用的 css 文件等

   ```html
   <link
     rel="stylesheet"
     href="../bootstrap-3.4.1-dist/css/bootstrap.min.css"
   />
   ```

4. 在[全局 CSS 样式](https://v3.bootcss.com/css/)中选要用的东西，复制对应标签

   ```html
   <button type="button" class="btn btn-danger">（危险）Danger</button>
   ```

5. 根据自己需要修改

   ```html
   <style>
     .container {
       width: 750px;
       margin: 0 auto;
     }

     .blue {
       background-color: skyblue !important;
     }
   </style>

   <body>
     <div class="container">
       <button type="button" class="btn btn-danger blue">登录</button>
     </div>
   </body>
   ```

组件使用例子：

```html
<!-- 字体图标 -->
<div class="container">
  <span class="glyphicon glyphicon-ok"></span>
</div>

<!-- 巨幕 -->
<div class="jumbotron">
  <h1>Hello, world!</h1>
  <p>One plus one equals two!</p>
  <p><a class="btn btn-primary btn-lg" href="#" role="button">Learn more</a></p>
</div>
```

### Bootstrap 布局容器

Bootstrap 预定义了两个 container 容器

1. **container 类**
   - 响应式布局的容器，固定宽度
   - 大屏(电脑)(>=1200px)：宽度固定为 1170px
   - 中屏(桌面显示器)(>=992px)：宽度固定为 970px
   - 小屏(平板)(>=768px)：宽度固定为 750px
   - 超小屏(手机)(<768px)：宽度固定为 100%

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />

    <!-- 要求当前网页使用IE浏览器最高版本的内核来渲染 -->
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />

    <!-- 视口标签的设置：视口的宽度和设备一致，默认的缩放比例和PC端一致，用户不能自行缩放 -->
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1.0, user-scalable=0"
    />

    <!--[if lt IE 9]>
      <script src="https://oss.maxcdn.com/html5shiv/3.7.2/html5shiv.min.js"></script>
      <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
    <![endif]-->

    <title>Myself</title>
    <style>
      .container {
        height: 200px;
        background-color: pink;
      }
    </style>
    <link
      rel="stylesheet"
      href="../bootstrap-3.4.1-dist/css/bootstrap.min.css"
    />
  </head>

  <body>
    <div class="container"></div>
  </body>
</html>
```

上面的例子等价于响应式布局容器的例子，简单来说就是，有大佬已经把它封装好了，可以直接用

2. **container-fluid 类**
   - 流式布局容器，100%宽度
   - 占据全部视口(viewport)的容器
   - 适合于单独做移动端开发

### 栅格系统

栅格系统是将页面布局划分为等宽的列，然后通过列数的定义来模块化页面布局。

Bootstrap 提供了一套响应式、移动设备优先的流动栅格系统，会把 container 分为 12 列。

栅格系统通过一系列的行(row)和列(column)的组合来创建页面布局。

规则：

- 行(row)必须要放在 container 布局容器里面

- 要实现列的平均划分，需要给类添加<b style="color: red">类前缀</b>

| 设备               |    尺寸区间     | 宽度设置 |  类前缀  |
| ------------------ | :-------------: | :------: | :------: |
| 手机               |     <768px      |   100%   | .col-xs- |
| 平板               | [768px, 992px)  |  750px   | .col-sm- |
| 桌面显示器         | [992px, 1200px) |  970px   | .col-md- |
| 大桌面显示器(电脑) |    >=1200px     |  1170px  | .col-lg- |

- xs(extra small)：超小；sm(small)：小；md(medium)：中等；lg(large)：大
- 列的总和大于 12 的话，多余的列会另起一行排列
- 每一列默认有左右 15 像素的 padding
- 可以同时为一列指定多个设备的类名，例如`class="col-md-4 col-sm-6"`

例子

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />

    <!-- 要求当前网页使用IE浏览器最高版本的内核来渲染 -->
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />

    <!-- 视口标签的设置：视口的宽度和设备一致，默认的缩放比例和PC端一致，用户不能自行缩放 -->
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1.0, user-scalable=0"
    />

    <!--[if lt IE 9]>
      <script src="https://oss.maxcdn.com/html5shiv/3.7.2/html5shiv.min.js"></script>
      <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
    <![endif]-->

    <title>Myself</title>
    <style>
      .container {
        height: 200px;
      }

      [class^="col"] {
        border: 1px solid #ccc;
      }

      .container > div:nth-child(1) div {
        background-color: pink;
      }
    </style>
    <link
      rel="stylesheet"
      href="../bootstrap-3.4.1-dist/css/bootstrap.min.css"
    />
  </head>

  <body>
    <div class="container">
      <div class="row">
        <div class="col-lg-3 col-md-4 col-sm-6 col-xs-12">1</div>
        <div class="col-lg-3 col-md-4 col-sm-6 col-xs-12">2</div>
        <div class="col-lg-3 col-md-4 col-sm-6 col-xs-12">3</div>
        <div class="col-lg-3 col-md-4 col-sm-6 col-xs-12">4</div>
      </div>

      <div class="row">
        <div class="col-lg-6">1</div>
        <div class="col-lg-1">2</div>
        <div class="col-lg-1">3</div>
        <div class="col-lg-4">4</div>
      </div>

      <!-- 总份数小于12 -->
      <div class="row">
        <div class="col-lg-3">1</div>
        <div class="col-lg-3">2</div>
        <div class="col-lg-3">3</div>
        <div class="col-lg-1">4</div>
      </div>
      <br />

      <!-- 总份数大于12 -->
      <div class="row">
        <div class="col-lg-5">1</div>
        <div class="col-lg-5">2</div>
        <div class="col-lg-3">3</div>
        <div class="col-lg-3">4</div>
      </div>
    </div>
  </body>
</html>
```

### 列嵌套

栅格系统可以将一个列再分成若干个小列。

例子：

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />

    <!-- 要求当前网页使用IE浏览器最高版本的内核来渲染 -->
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />

    <!-- 视口标签的设置：视口的宽度和设备一致，默认的缩放比例和PC端一致，用户不能自行缩放 -->
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1.0, user-scalable=0"
    />

    <!--[if lt IE 9]>
      <script src="https://oss.maxcdn.com/html5shiv/3.7.2/html5shiv.min.js"></script>
      <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
    <![endif]-->

    <title>Myself</title>
    <style>
      .container {
        height: 200px;
      }

      [class^="col"] {
        border: 1px solid #ccc;
      }

      .container .row div {
        background-color: pink;
      }
    </style>
    <link
      rel="stylesheet"
      href="../bootstrap-3.4.1-dist/css/bootstrap.min.css"
    />
  </head>

  <body>
    <div class="container">
      <div class="row">
        <div class="col-md-3">
          <!-- 直接把列分成小列，会出现小列无法铺满原来的列，因为小列的父元素会有padding值 -->
          <div class="col-md-6" style="background-color: red">11</div>
          <div class="col-md-6" style="background-color: red">12</div>
        </div>
        <div class="col-md-3">2</div>
        <div class="col-md-3">3</div>
        <div class="col-md-3">4</div>
      </div>

      <div class="row">
        <div class="col-md-3">
          <!-- 把列分成行后，再把行分成小列，可以实现取消父元素的padding值 -->
          <div class="row">
            <div class="col-md-6" style="background-color: red">11</div>
            <div class="col-md-6" style="background-color: red">12</div>
          </div>
        </div>
        <div class="col-md-3">2</div>
        <div class="col-md-3">3</div>
        <div class="col-md-3">4</div>
      </div>
    </div>
  </body>
</html>
```

### 列偏移

使用`类前缀-offset-*类`可以将列向右侧偏移，这些类实际是通过\*选择器为当前元素增加了左边距（margin）

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />

    <!-- 要求当前网页使用IE浏览器最高版本的内核来渲染 -->
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />

    <!-- 视口标签的设置：视口的宽度和设备一致，默认的缩放比例和PC端一致，用户不能自行缩放 -->
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1.0, user-scalable=0"
    />

    <!--[if lt IE 9]>
      <script src="https://oss.maxcdn.com/html5shiv/3.7.2/html5shiv.min.js"></script>
      <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
    <![endif]-->

    <title>Myself</title>
    <style>
      .container {
        height: 200px;
      }

      [class^="col"] {
        border: 1px solid #ccc;
      }

      .container .row div {
        background-color: pink;
      }
    </style>
    <link
      rel="stylesheet"
      href="../bootstrap-3.4.1-dist/css/bootstrap.min.css"
    />
  </head>

  <body>
    <div class="container">
      <div class="row">
        <!-- 两个div，空出中间一块，只需要右边的盒子偏移 12 - 左盒子占的份数 - 右盒子占的份数即可 -->
        <div class="col-md-4">左</div>
        <div class="col-md-4 col-md-offset-4">右</div>

        <!-- 一个盒子占中间位置，只需给这个盒子偏移 (12 - 盒子占的份数) / 2即可 -->
        <div class="col-md-6 col-md-offset-3">中</div>
      </div>
    </div>
  </body>
</html>
```

### 列排序

使用`类前缀-push-*和类前缀-pull-*`可以改变列的顺序（<b style="color: red">往左边是 pull，往右边是 push,写错的话得不到预期的结果</b>）

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />

    <!-- 要求当前网页使用IE浏览器最高版本的内核来渲染 -->
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />

    <!-- 视口标签的设置：视口的宽度和设备一致，默认的缩放比例和PC端一致，用户不能自行缩放 -->
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1.0, user-scalable=0"
    />

    <!--[if lt IE 9]>
      <script src="https://oss.maxcdn.com/html5shiv/3.7.2/html5shiv.min.js"></script>
      <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
    <![endif]-->

    <title>Myself</title>
    <style>
      .container {
        height: 200px;
      }

      [class^="col"] {
        border: 1px solid #ccc;
      }

      .container .row div {
        background-color: pink;
      }
    </style>
    <link
      rel="stylesheet"
      href="../bootstrap-3.4.1-dist/css/bootstrap.min.css"
    />
  </head>

  <body>
    <div class="container">
      <div class="row">
        <!-- 两个div，空出中间一块，只需要右边的盒子偏移 12 - 左盒子占的份数 - 右盒子占的份数即可 -->
        <div class="col-md-4 ">左</div>
        <div class="col-md-5 col-md-offset-3">右</div>
        <!-- 想要把左右盒子互换位置，可以pull(拉)右边的盒子过来，拉的份数为左盒子的份数 + 右盒子的偏移份数
            push(推)左边的盒子过去，推的份数为右盒子的份数 + 右盒子的偏移份数 -->
        <div class="col-md-4 col-md-push-8">左</div>
        <div class="col-md-5 col-md-offset-3 col-md-pull-7">右</div>
      </div>
    </div>
  </body>
</html>
```

### 隐藏和显示内容

![](https://pic.imgdb.cn/item/6145f81a2ab3f51d91fed067.jpg)

和上面相反的是 visible-xs, visible-sm, visible-md, visible-lg，显示内容

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />

    <!-- 要求当前网页使用IE浏览器最高版本的内核来渲染 -->
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />

    <!-- 视口标签的设置：视口的宽度和设备一致，默认的缩放比例和PC端一致，用户不能自行缩放 -->
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1.0, user-scalable=0"
    />

    <!--[if lt IE 9]>
      <script src="https://oss.maxcdn.com/html5shiv/3.7.2/html5shiv.min.js"></script>
      <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
    <![endif]-->

    <title>Myself</title>
    <style>
      .container {
        height: 200px;
      }

      [class^="col"] {
        border: 1px solid #ccc;
      }

      .container .row div {
        background-color: pink;
      }

      span {
        font-size: 20px;
        color: #fff;
      }
    </style>
    <link
      rel="stylesheet"
      href="../bootstrap-3.4.1-dist/css/bootstrap.min.css"
    />
  </head>

  <body>
    <div class="container">
      <div class="row">
        <div class="col-xs-4">
          1
          <!-- 超小屏（手机）才会显示 -->
          <span class="visible-xs">Hello! 手机</span>
        </div>

        <!-- 小屏和大屏会隐藏 -->
        <div class="col-xs-4 hidden-sm hidden-lg">2</div>

        <div class="col-xs-4">3</div>
      </div>
    </div>
  </body>
</html>
```

参考：
pink 老师前端入门教程
