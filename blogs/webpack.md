---
title: webpack
date: 2021-10-01 21:12:05
categories: "前端"
tags:
  - Webpack
---

# 前端工程化和 webpack

前端开发四大要点：

1. **模块化**：js、css、资源的模块化
2. **组件化**：UI 结构、样式、行为可复用
3. **规范化**：目录结构、编码、接口、文档规范化、git 分支管理
4. **自动化**：自动化构建、自动部署、自动化测试

## 1. 前端工程化

**前端工程化**：在企业级的前端项目开发中，把前端开发所需的工具、技术、经验等进行规范化、标准化。

早期的前端工程化解决方案：

- grunt
- gulp

目前主流的前端工程化解决方案：[webpack]([webpack 中文文档 | webpack 中文网 (webpackjs.com)](https://www.webpackjs.com/))

## 2. webpack 的基本使用

**webpack**是前端项目工程化的具体解决方案。

主要功能：提供友好的**前端模块化开发**支持，以及**代码压缩混淆**，**处理浏览器端 Javascript 的兼容性**，**性能优化**等功能。

优点：**提高了前端开发效率和项目的可维护性**

例子：

创建列表隔行换色项目

步骤：

1. 新建项目空白目录（路径英文），运行` npm init -y`命令，初始化包管理配置文件 package.json

   ![](https://pic.imgdb.cn/item/61473bdd2ab3f51d9129e6f5.jpg)

2. 新建 src 源代码目录

3. 新建 src/index.html 首页和 src/index.js 文件

4. 初始化首页基本的结构

   ![](https://pic.imgdb.cn/item/61473e572ab3f51d912eb768.jpg)

5. 运行` npm install jquery -S`命令，安装 jQuery,其中-S 是--save 的缩写

   ![](https://pic.imgdb.cn/item/61473f4c2ab3f51d91307175.jpg)

6. 编写 index.js，实现隔行换色效果

   ```javascript
   import $ from "jquery"; /*导入jquery，用$符号接*/

   $(function () {
     $("ul li:odd").css("background", "pink");
     $("ul li:even").css("background", "purple");
   });
   ```

   报错：

   ![](https://pic.imgdb.cn/item/614741a22ab3f51d9134bfed.jpg)

   原因：import 方法较高级，浏览器执行会出错,可使用 webpack 解决问题

7. 在项目中安装 webpack

   安装 webpack 相关的两个包` npm install webpack webpack-cli -D`（-D 是--save-dev 的缩写）

   ![](https://pic.imgdb.cn/item/6147436b2ab3f51d91383879.jpg)

8. 配置 webpack

   1. 在项目根目录下，创建名为 webpack.config.js 的 webpack 配置文件，并初始化配置

      ```javascript
      module.exports = {
        mode: "development", //mode用来指定模式，可以是development(开发模式)或production(生产模式)
      };
      ```

      ![](https://pic.imgdb.cn/item/614745542ab3f51d913c1e41.jpg)

   2. 在 package.json 的 scripts 节点下，新增 dev 脚本

      ```json
      "scripts": {
          "dev": "webpack"
       }
      //dev脚本名字可变，后面的webpack是命令名，不可变，script节点下的脚本可以通过npm run执行，如npm run dev
      ```

9. 执行` npm run dev`命令，启动 webpack 进行项目的打包构建

   ![](https://pic.imgdb.cn/item/614747862ab3f51d91408073.jpg)

10. 更换使用的 js 文件为新生成的 js 文件

    ```html
    <script src="../dist/main.js"></script>
    ```

    ![image-20210919222517017](https://s2.loli.net/2022/01/23/SJUBCnszLdmr1ZQ.png)

配置 webpack 时 mode 节点的可选值有两个：

1. development

   - **开发环境**
   - **不会**对打包生成的文件进行**代码压缩**和**性能优化**
   - 打包**速度快**， 适合在开发阶段使用

2. production

   - **生产环境**
   - 会**对打包生成的文件进行**代码压缩**和**性能优化\*\*
   - 打包**速度很慢**， 适合在项目发布阶段使用

   ![](https://pic.imgdb.cn/item/6147fdf62ab3f51d917ac9e7.jpg)

   webpack.config.js 是 webpack 的配置文件。webpack 在真正开始打包构建之前，会**先读取这个配置文件**，然后根据给定的配置，对项目进行打包。

   webpack4.x 和 5.x 的版本中：

   1. 默认的打包入口文件为 src/index.js
   2. 默认的输出文件路径为 dist/main.js

   找不到入口文件会报错，如更改 src 文件夹和更改 index.js 文件名

   ![](https://pic.imgdb.cn/item/6147ffc82ab3f51d917d6373.jpg)

   可以更改通过 webpack 的配置文件来自定义打包的入口和出口。通过**entry 节点**指定**打包的入口**，通过**output 节点**指定**打包的出口**

   ```js
   const path = require("path"); //导入node.js中专门操作路径的模块
   module.exports = {
     mode: "development", //mode用来指定模式，可以是development(开发模式)或production(生产模式)

     entry: path.join(__dirname, "./src/myindex.js"), //打包入口文件路径
     output: {
       path: path.join(__dirname, "./dist/mymain.js"), //打包的出口路径
       filename: "mymain.js", //输出文件名称
     },
   };
   ```

   ![](https://pic.imgdb.cn/item/614802ab2ab3f51d918171aa.jpg)

   <b style="color: red">问题：更改 myindex.js，页面用的还是打包的版本，需要再次执行`npm run dev`命令</b>

## 3. webpack 插件

### 3.1 webpack-dev-server

每当修改了源代码，webpack 会自动进行项目的打包和构建

- 安装 webpack-dev-server，`npm install webpack-dev-server@3.11.2 -D`，-D 表示只在开发阶段会用到，这里练习时，不加版本号报错

- 配置 webpack-dev-server：修改 package.json 中的 script 节点的 dev 命令为"webpack serve"

  ```json
  "scripts": {
      "dev": "webpack serve"
    }
  ```

- 再次执行` npm run dev`命令，重新进行项目的打包![](https://pic.imgdb.cn/item/614806e12ab3f51d91870a28.jpg)

  命令没有结束，会一直监听源代码有没有变化，每当保存源代码，都会自动打包

  ![](https://pic.imgdb.cn/item/614808012ab3f51d91889be0.jpg)

  <b style="color: red">注意：这里又会出现问题，自动打包后，vscode，右键打开 index.html 文件会发现，没有变化</b>。因为 webpack-dev-server 会启动一个**实时打包的 http 服务器**，即无法通过 file 协议查看打包效果，需要通过 http 协议查看效果

- 在浏览器中访问 http://localhost:8080 地址，查看自动打包效果

  问题还是没有解决：样式还是没有实时变化。

  ![](https://pic.imgdb.cn/item/614816732ab3f51d919d71ee.jpg)

  原因：

  配置了 webpack-dev-server 后，打包生成的文件并没有放在物理磁盘上，而是放到了**内存**中，可以在 http://localhost:8080/mymain.js（后面是生成的文件名）中查看真正生成的 js 文件。

  为什么要放在内存中呢？

  因为可能会需要频繁保存源代码，需要频繁读写文件，放在内存中，可以提高实时打包输出的性能，因内存比物理磁盘快很多。

- 这样子的话，引入 js 文件的路径就得变成内存中的位置才对了

  <script src="/mymain.js"></script>

- 之后，每次更改源代码，会实时刷新，可以实时查看效果

### 3.2 html-webpack-plugin

html-webpack-plugin 是\*\*webpack 中的 HTML 插件，通过此插件可以复制 html 文件放到其他位置（内存中）

- 安装 html-webpack-plugin 插件，` npm install html-webpack-plugin@5.3.2 -D`

- 配置 html-webpack-plugin，(在 webpack.config.js 中)

  ```js
  //1. 导入HTML插件，得到一个构造函数
  const HtmlPlugin = require("html-webpack-plugin");

  //2. 创建HTML插件的实例对象
  const htmlPlugin = new HtmlPlugin({
    template: "./src/index.html", //指定要复制的文件路径
    filename: "./index.html", //指定生成的文件的路径
  });

  module.exports = {
    mode: "development",

    //3. 通过plugins节点，使htmlPlugin插件生效
    plugins: [htmlPlugin],
    //plugins：插件的数组，webpack运行时会加载并调用这些插件
  };
  ```

  **这里只有配置 html-webpack-plugin 的内容，为了看起来更清晰**

- 重新执行` npm run dev`命令，打开 http://localhost:8080/，可以直接查看效果，也是会实时更新

  通过插件复制的 index.html 页面，**被放到了内存中**

  HTML 插件在生成复制的 index.html 页面时，会自动引入打包的 js 文件（即不需要自己引入 js 文件）

  ![](https://pic.imgdb.cn/item/61483d642ab3f51d91e05328.jpg)

### 3.3 devServer 节点

在 webpack.config.js 配置文件中，可以通过 devServer 节点进行其他配置，如实现初次打包时，自动打开浏览器等

```js
devServer: {
        open: true,
        host: "127.0.0.1", //使用的主机地址
        port: 80, //实时打包使用的端口号
    }
```

## 4. loader

webpack 默认只能打包处理以.js 后缀为结尾的模块。其他不是以.js 后缀为结尾的模块 webpack 默认处理不了，**需要调用 loader 加载器才可以正常打包**。

loader 加载器的作用：协助 webpack 打包处理特定的文件模块

- css-loader：可以打包处理.css 相关文件
- less-loader：可以打包处理.less 相关的文件
- babel-loader：可以打包处理 webpack 无法处理的高级 JS 语法

例子：

### 4.1 打包处理 css 文件

1. 编写 CSS 文件

2. myindex.js 文件导入 css

   ```js
   import $ from "jquery"; /*导入jquery，用$符号接*/

   //导入样式，在webpack中，一切都是模块
   import "./css/index.css";

   $(function () {
     $("ul li:odd").css("background", "red");
     $("ul li:even").css("background", "pink");
   });
   ```

3. 安装处理 css 文件的 loader，` npm i style-loader@3.0.0 css-loader@5.2.6 -D`

4. 在 webpack 的配置文件中的 module->rules 数组中，添加 loader 规则

   ```js
   module: {
     //所有第三方文件模板的匹配规则
     rules: [
       //对应文件的匹配规则
       {
         test: /\.css$/,
         //test表示匹配的文件类型（后缀名）

         use: ["style-loader", "css-loader"],
         //use表示对应要调用的loader,多个loader的调用顺序是从后往前调用，
         //即先调用css-loader，再调用style-loader，前面没有loader后给会webpack，webpack把style-loader处理的结果，合并到/dist/mymain.js中，生成最终打包好的文件
       },
     ];
   }
   ```

5. npm run dev，<b style="color: red">注意，这里如果 index.html 导入了 css 文件，myindex.js 文件也导入 css 文件，会报错</b>

### 4.2 打包处理 less 文件

1. 编写 less 文件

2. myindex.js 文件导入 less

3. 安装处理 less 文件的 loader，` npm i less-loader@10.0.1 -D`

4. 添加 loader 规则

   ```js
   module: {
     //所有第三方文件模板的匹配规则
     rules: [
       //对应文件的匹配规则
       {
         test: /\.css$/,
         use: ["style-loader", "css-loader"],
       },
       {
         test: /\.less$/,
         use: ["style-loader", "css-loader", "less-loader"],
       },
     ];
   }
   ```

### 4.3 打包处理与 url 路径相关的文件

1. 通过 myindex.js 导入图片

   ```js
   import $ from "jquery"; /*导入jquery，用$符号接*/

   //导入样式，在webpack中，一切都是模块
   import "./css/index.css";
   import "./css/myindex.less";

   import logo from "./images/logo.png";

   $(function () {
     $("img").attr("src", logo);

     $("ul li:odd").css("background", "red");
     $("ul li:even").css("background", "pink");
   });
   ```

2. 运行命令，` npm i url-loader@4.1.1 file-loader@6.2.0 -D`

3. 添加 loader 规则

   ```js
   module: {
     //所有第三方文件模板的匹配规则
     rules: [
       //对应文件的匹配规则
       {
         test: /\.jpg|png|gif$/,
         use: "url-loader",
       },
     ];
   }
   ```

   上面的 use 中，后面可以增加参数

   如：

   ```js
   module: {
     //所有第三方文件模板的匹配规则
     rules: [
       //对应文件的匹配规则
       {
         test: /\.jpg|png|gif$/,
         use: "url-loader?limit=300",
       },
     ];
   }
   ```

   - limie 用来指定图片格式的大小，单位是字节
   - 只有<=limit 大小的图片，才会被转成 base64 格式的图片

   练习用图片大小：337 字节

   limit=300 时，输出 logo

   ![](https://pic.imgdb.cn/item/61485acb2ab3f51d91108a8f.jpg)

   limit=400 时，输出 logo

   ![](https://pic.imgdb.cn/item/61485b5d2ab3f51d911146e2.jpg)

## 5. 打包发布

项目开发完成之后，需要使用 webpack 对项目进行打包发布。原因：

1. 开发环境下，打包生成的文件**存在于内存中**，无法获取到最终生成的文件
2. 开发环境下，打包生成的文件**不会出现代码压缩和性能优化**

配置 webpack 的打包发布

在 package.json 文件的 script 节点下，增加新的命令：

```json
  "scripts": {
    "dev": "webpack serve",
     //开发环境中，运行dev命令(npm run dev)
    "build": "webpack --mode production"
      //项目发布时，运行build命令(npm run build)，--mode用来指定webpack的运行模式。会覆盖webpack.config.js中的model选项
  }
```

执行 npm run dev 命令后，会发现生成的 js 文件、图片文件（只有 base64 格式的图片会生成）和 index.html 文件都直接放在 dist 文件夹下。

### 5.1 把 js 文件统一放到生成的 js 目录中

在 webpack.config.js 的 output 节点中，进行配置

```js
output: {
        path: path.join(__dirname, "./dist"), //打包的出口路径
        filename: "js/mymain.js" //输出文件名称
    }
```

### 5.2 把图片文件统一放到生成的 images 目录下

在 webpack.config.js 配置文件中的 module 下的 rules，对应的 url 路径相关的文件下的 use 后增加参数，位置具体参考 4.3

```js
{
    test: /\.jpg|png|gif$/,
    use: "url-loader?limit=300&outputPath=images"
}
```

outputPath 选项可以指定图片文件的输出路径

没有及时删除 dist 再重新 npm run build 会出现以下下问题

![](https://pic.imgdb.cn/item/6148907c2ab3f51d91660de2.jpg)

### 5.3 自动清理 dist 目录下的旧文件

为了在每次打包发布时**自动清理 dist 目录下的旧文件**，可以安装 clean-webpack-plugin 插件

1. 安装

```
npm install --save-dev clean-webpack-plugin
```

2. 配置（在 webpack.config.js 中）

   ![](https://pic.imgdb.cn/item/614892a72ab3f51d916a3df4.jpg)

## 6. Souce Map

前端项目在投入生产环境之前，都要对 Javascript 源代码进行压缩混淆，减小文件体积，提高文件加载效率。

对压缩混淆之后的代码除错很困难：

- 变量会被替换成没有任何语义的名称，如 a, b, c 等
- 空行和注释被剔除

Source Map 时一个信息文件，里面存着位置信息。Source Map 文件中存着压缩混淆后的代码对应变化前的位置。

有了它，出错时会直接显示原始代码，而不是转换后的代码，方便了程序员的调试。

### 6.1 默认 Source Map 的问题

在开发环境下，webpack 默认启用了 Source Map 功能。当程序出错时，可以直接在控制台显示错误行的位置，并定位到具体的源代码。

默认生成的 Source Map 记录的是生成后的代码的位置，会导致报错时的行数与源代码的行数不一致。

![](https://pic.imgdb.cn/item/6148966b2ab3f51d91711419.jpg)

### 6.2 解决默认 Source Map 的问题

开发环境下，在 webpack.config.js 中添加以下配置，就可以实现运行时报错的行数和源代码的行数保持一致

![](https://pic.imgdb.cn/item/614897432ab3f51d9172ab98.jpg)

生产环境中，如果省略 devtool 选项，那么生成的文件中不包含 Source Map。这样可以防止源代码通过 Source Map 的形式暴露出去。

![](https://pic.imgdb.cn/item/614898c42ab3f51d9175a293.jpg)

**只定位行数，不暴露源码**：在生产环境下，只想知道报错的地方在源码的具体行数，而且不想暴露源码，将 devtool 的值设置为**nosources-source-map**。

![](https://pic.imgdb.cn/item/614899f92ab3f51d917801c4.jpg)

**定位行数，暴露源码**：将 devtool 的值设置为**source-map**。了解即可，<b style="color:red">极其不安全</b>

总结：

![](https://pic.imgdb.cn/item/61489a962ab3f51d91792fbc.jpg)

学习链接：

[黑马程序员 Vue 全套视频教程](https://www.bilibili.com/video/BV1zq4y1p7ga)
