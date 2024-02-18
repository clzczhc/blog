---
title: Webpack笔记
date: 2022-02-06 14:44:22
keywords: Webpack
categories: "前端"
tags:
  - 青训营
  - Webpack
---

# Webpack 笔记

参加字节跳动的青训营时写的笔记。这部分是范文杰老师讲的课。

插一嘴：范文杰老师的公众号**Tecvan**有很多干活，可以关注一下。(下面的部分有好多都有很有用的扩展链接，偷懒，就直接把老师的公众号贴出来)

## 1. 简介

![image-20220123215526028](https://s2.loli.net/2022/02/06/IkXns8yYKHgZrzp.png)

Webpack 本质上是一种前端资源编译、打包工具。

功能：

- 多份资源文件打包成一个 Bundle
- 支持 Less、Babel、Eslint、TypeScript
- 支持模块化处理 css、图片等资源文件
- 支持 HMR(热更新)
- 支持 Tree-shaking
- 支持 SourceMap
- ,,,

## 2. 使用 Webpack

### 2.1 使用步骤

1. 安装，` npm install webpack webpack-cli -D`

2. 编辑配置文件(webpack.config.js)

   ```js
   const path = require("path");

   module.exports = {
     entry: path.join(__dirname, "src", "index.js"),
     mode: "development",
     output: {
       filename: "bundle.js",
       path: path.join(__dirname, "dist"),
     },
   };
   ```

3. 执行编译命令，` npx webpack`

### 2.2 核心流程

1. **入口处理**：从入口文件开始启动编译流程
2. **依赖解析**：根据` require`或` import`等语句找到依赖资源
3. **资源解析**：根据` module`配置，调用资源加载器，把 css、less、png 等非标准 JS 资源转译成 JS 内容(Webpack 只认识 JS)
4. **资源合并打包**：将转译后的资源合并打包为可以直接在浏览器运行的 JS 文件，包括代码混淆、代码压缩等操作。

<b style="color: red">递归调用 2、3，直到所有资源处理完毕。因为依赖资源可能也会依赖其他资源。</b>

### 2.3 分类

Webpack 的使用基本都围绕**配置**展开，而配置大致可分为两类：

- **流程类**：作用于流程中，直接影响打包效果的配置项
- **工具类**：主流程之外，提供更多工程化能力的配置项

![image-20220124174806526](https://s2.loli.net/2022/02/06/bpg3fRc2SeoEBXz.png)

### 2.4 练习

#### 2.4.1 简单使用

<b style="color: red">模块在上一层。(为了不影响观感)</b>

1. 编写代码

![image-20220124174656480](https://s2.loli.net/2022/02/06/6tYolBpqW85Niab.png)

2. 编辑 webpack 配置文件

   ```js
   const path = require("path");

   module.exports = {
     entry: path.join(__dirname, "src", "index.js"),
     mode: "development",
     output: {
       filename: "bundle.js",
       path: path.join(__dirname, "dist"),
     },
   };
   ```

3. 执行` npx webpack`

   ![image-20220124175243382](https://s2.loli.net/2022/02/06/Vf2e5YPjGB6QHNc.png)

#### 2.4.2 处理 CSS

![image-20220124180035396](https://s2.loli.net/2022/02/06/hdKWLnOr4SicsN7.png)

按 2.4.1 的步骤直接执行，会发现出错

![image-20220124180151225](https://s2.loli.net/2022/02/06/qfrm5LzBx7YMtCD.png)

<b style="color: red">原因：Webpack 只认识 JS，所以需要使用 Loader 将对应非 JS 资源转译成 JS</b>

1. 安装 Loader

   ` npm install css-loader style-loader -D`

2. 添加` module`，使用 Loader 处理 CSS 文件

   ```js
   const path = require("path");

   module.exports = {
     entry: path.join(__dirname, "src", "index.js"),
     mode: "development",
     output: {
       filename: "bundle.js",
       path: path.join(__dirname, "dist"),
     },
     module: {
       rules: [
         {
           test: /\.css$/,
           use: ["style-loader", "css-loader"],
         },
       ],
     },
   };
   ```

3. 执行` npx webpack`，成功

   ![image-20220124180951673](https://s2.loli.net/2022/02/06/k2VSJy5rPZ8jpYA.png)

#### 2.4.3 使用 Babel

Babel 用于将使用 ES6 语法编写的 JS 代码转换为向后兼容的 JavaScript 语法，以便能够运行在当前和旧版本的浏览器或其他环境中。

1. 安装依赖。

   ` npm install @babel/core @babel/preset-env babel-loader`

2. 配置。

   ```js
   const path = require("path");

   module.exports = {
     entry: path.join(__dirname, "src", "index.js"),
     mode: "development",
     output: {
       filename: "bundle.js",
       path: path.join(__dirname, "dist"),
     },
     module: {
       rules: [
         {
           test: /\.js$/,
           use: [
             {
               loader: "babel-loader",
               options: {
                 presets: [["@babel/preset-env"]],
               },
             },
           ],
         },
       ],
     },
   };
   ```

3. ` npx webpack`

   **源代码**：

   ```js
   let add = (a, b) => a + b;
   ```

   **转换后的**：

   ![image-20220124191225084](https://s2.loli.net/2022/02/06/H3SOZeTbsQWc1af.png)

#### 2.4.4 生成 HTML

1. 安装依赖

   ` npm install html-webpack-plugin -D`

2. 配置。因为这个不是 loader，而是 plugin(插件)，所以配置需要引入插件，然后在` plugins`中实例化插件

   ```js
   const path = require("path");
   const HtmlWebpackPlugin = require("html-webpack-plugin");

   module.exports = {
     entry: path.join(__dirname, "src", "index.js"),
     mode: "development",
     output: {
       filename: "bundle.js",
       path: path.join(__dirname, "dist"),
     },
     plugins: [new HtmlWebpackPlugin()],
   };
   ```

3. `npx webpack`

   ![image-20220124192212302](https://s2.loli.net/2022/02/06/dI1ilLcGVvgxU2k.png)

#### 2.4.5 HMR(热更新)

热更新简单来说就是可以在应用运行的时候,不需要刷新页面,就可以实时查看到修改的效果

1. 安装依赖

   ` npm install webpack-dev-server -D`

2. 开启 HMR

![image-20220124193400240](https://s2.loli.net/2022/02/06/YUXrOxTcFPyupEk.png)

3. 启动 Webpack

` npx webpack serve`

![动画](https://s2.loli.net/2022/02/06/uIROe6HBP1vUSVa.gif)

![image-20220124194101445](https://s2.loli.net/2022/02/06/lsMdryi2bhXZnI6.png)

#### 2.4.6 Tree-Shaking

**Tree-shaking**：树摇，用于删除 Dead Code

Dead Code：

- 代码没有被用到
- 代码只读不写

Dead Code 例子：

![image-20220124232306850](https://s2.loli.net/2022/02/06/ngbMeHsGUJpLSBQ.png)

没有使用 Tree-Shaking 时：

![image-20220124232435821](https://s2.loli.net/2022/02/06/YHcrp6QUniOGWo3.png)

修改配置文件，开启 Tree-Shaking：

![image-20220124233227128](https://s2.loli.net/2022/02/06/cCYBrLUlqyxI7fk.png)

结果：

![image-20220124233301693](https://s2.loli.net/2022/02/06/XBPMrJuknAVjYFD.png)

### 2.5 更多功能

- 缓存
- Sourcemap
- 性能监控
- 日志
- 代码压缩
- 分包

## 3. Loader

作用：**将资源转译成标准 JS**。(因为 Webpack 只认识 JS)

### 3.1 Loader 使用示例

使用 Loader(处理 less)：

1. 安装 Loader

   ` npm install style-loader css-loader less-loader -D`

2. 添加` module`处理

   ```js
   const path = require("path");

   module.exports = {
     entry: path.join(__dirname, "src", "index.js"),
     mode: "development",
     output: {
       filename: "bundle.js",
       path: path.join(__dirname, "dist"),
     },
     module: {
       rules: [
         {
           test: /\.less$/,
           use: ["style-loader", "css-loader", "less-loader"],
         },
       ],
     },
   };
   ```

分析：

- less-loader：实现 less -> css 的转换
- css-loader：将 css 转换成 module.eports = "${css}"的形式，符合 JS 的语法(直截了当来说的话，就是把 css 用 js 表示)
- style-loader：将 css 模块包进 require 语句，并在运行时调用 injectStyle 等函数<b style="color: red">将内容注入到页面的 style 标签</b>

### 3.2 Loader 其他特性

![image-20220124235232348](https://s2.loli.net/2022/02/06/Vh7MUkZBvEDjNwT.png)

- **链式执行**(过程的输出刚好是下一个过程的输入)
- **支持异步执行**
- 分 normal、pitch 两种模式(<b style="color: red">这部分没有什么概念</b>)

### 3.3 编写简单 Loader

#### 3.3.1 Loader 形式

```js
module.exports = function (source, sourceMap?, data?) {
  // ?代表参数可有可无
  // source是loader的输入。可能是文件内容或上一个loader处理的结果

  // ......

  return source; // 此时的source是指经loader处理后的source(不处理则是原来的source，如eslint-loader)
};
```

#### 3.3.2 开始动手

**目录结构**

![image-20220125154719915](https://s2.loli.net/2022/02/06/ekhqvYgz78Topy1.png)

**index.js**

```js
let t1 = "Hello";
```

**自编简单 loader**

```js
module.exports = function (source) {
  console.log(source);
  source += ` let t2 = ' World';
              console.log(t1 + t2);`;
  return source;
};
```

**配置文件**

```js
const path = require("path");

module.exports = {
  entry: path.join(__dirname, "src", "index.js"),
  mode: "production",
  output: {
    filename: "bundle.js",
    path: path.join(__dirname, "dist"),
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        use: path.join(__dirname, "loader", "myloader.js"),
      },
    ],
  },
};
```

![image-20220125154908116](https://s2.loli.net/2022/02/06/rzc7MKgAXqlEebV.png)

![image-20220125154931895](https://s2.loli.net/2022/02/06/HLW1AXUMh3u9QNI.png)

### 3.4 常见 Loader

![image-20220125155015086](https://s2.loli.net/2022/01/25/s2kMWbN3cOEwtrf.png)

## 4. Plugin(插件)

使用如 2.4.4 所示

写插件，暂时无能为力，先把课件上的干货放出来(方便以后查看)

![image-20220125155439723](https://s2.loli.net/2022/01/25/v9gXo8siF7eDmct.png)

![image-20220125155456498](https://s2.loli.net/2022/01/25/xMPJvrSybXOgkBz.png)

## 5. 更多

在老师的公众号可能没有的(老师分享的)

[Awesome Webpack4+ 优秀学习资源](https://github.com/Tecvan-fe/awesome-webpack-4plus)

[深入浅出 Webpack](https://webpack.wuhaolin.cn/)

[Webpack 5 知识体系](https://gitmind.cn/app/doc/fac1c196e29b8f9052239f16cff7d4c7)
