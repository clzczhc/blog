---
title: Webpack搭建简单的TypeScript脚手架
categories: 前端
date: 2022-06-18 12:51:29
tags:
  - TypeScript
  - Webpack
---

# Webpack搭建简单的TypeScript脚手架

## 前言
> 这里的脚手架只是指能更方便学习TypeScript的基础工具
> 简单入门了一下Typescript(可能还没入门)，学习TypeScript并不能直接运行查看结果，需要`tsc xxx.ts`将TS编译出JS才能执行，这样子很明显不是很方便。

虽然我们也可以在TypeScript中文网的练习平台写，直接看对比编译出来的JS代码，但是实际看代码运行结果还是需要点击运行按钮，去到新页面，再打开控制台。

所以为了很方便地学习TS，搭建一个简单的TypeScript脚手架很有必要

## 步骤

### 项目初始化
`npm init -y`：对项目进行初始化操作对包进行管理。(会生成默认的`package.json`文件)

添加`src`目录，后续的代码在`src`目录下进行编写

### 安装webpack

`npm install webpack webpack-cli`

添加Webpack配置文件`webpack.config.js`，设置入口文件、出口文件地址。
```js
const path = require('path')

module.exports = {
    // 开发模式
    mode: 'development',

    // 入口文件是src目录下的`index.js`文件
    entry: path.join(__dirname, 'src', 'index.js'),
    
     
    output: {
        // 把所有依赖的模块合并输出到一个 index.js 文件
        filename: 'index.js',

        // 输出文件都放到 dist 目录下
        path: path.join(__dirname, 'dist')
    }
}
```

### 初次测试
编写一下`index.js`文件，测试一下前面的配置是否正确。
```js
console.log('赤蓝紫')
```

执行命令`npx webpack`
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2840f6380f4b45d788d234a64b4f2767~tplv-k3u1fbpfcp-zoom-1.image)

执行编译生成的文件，能得到正确的结果就表示前面的步骤正确了。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eee2a95b5d0d4a9888f1a4d24a57a507~tplv-k3u1fbpfcp-zoom-1.image)

### 生成html
上面我们已经能够使用Webpack编译打包js代码了，但是生成的是js文件，还得去执行它。所以接下来我们需要能够开启一个服务。开启服务之前得先让它能够生成html文件。

1. 安装依赖
`npm install html-webpack-plugin`

2. 修改配置，引入并使用插件
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d68ce145b6bd4a62b133cd28811ab497~tplv-k3u1fbpfcp-zoom-1.image)

3. 执行`npx webpack`
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f317369984bb4005a8c83cb70decc331~tplv-k3u1fbpfcp-zoom-1.image)

### 开启服务器
1. 安装`webpack-dev-server`：`npm install webpack-dev-server`
2. 执行`npx webpack serve`
3. 打开http://localhost:8080/

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/060925f181a54875a24a55ad88a529ce~tplv-k3u1fbpfcp-zoom-1.image)


### 处理TS文件
现在我们的脚手架还是不能处理TS文件的。

index.js
```js
import './myts.ts'
```

myts.ts
```ts
let age: number = 21
console.log(age)
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c64caf42202844dd903dfd1f13d6b030~tplv-k3u1fbpfcp-zoom-1.image)

处理TS文件其实也不难，只需要两个步骤就行：
1. 安装`ts-loader`，`npm install ts-loader`
2. 修改Webpack配置文件`webpack.config.js`，增加`module`节点
```js
module: {
    rules: [
        {
            // ts后缀名的文件会使用ts-loader
            test: /\.ts$/,
            use: ["ts-loader"]
        }
    ]
}
```
3. 增加TS配置文件，空文件也行，只是一定要有


再次执行`npx webpack serve`
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ddc46ad1ea64f5b9cff7d84936a0e84~tplv-k3u1fbpfcp-zoom-1.image)

然后，还可稍微修改一下`package.json`文件，设置`npx webpack serve`命令为更常用的`npm run dev`
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e432893f879c416491fa4d0188b8fbde~tplv-k3u1fbpfcp-zoom-1.image)

简单的TS脚手架这样子就结束了。之后就能更方便的学习TS了。

完整版Webpack配置献上：
```js
const path = require("path")
const HtmlWebpackPlugin = require("html-webpack-plugin")

module.exports = {
    mode: "development",
    entry: path.join(__dirname, "src", "index.js"),
    output: {
        filename: "index.js",
        path: path.join(__dirname, "dist")
    },
    plugins: [new HtmlWebpackPlugin()],
    module: {
        rules: [
            {
                // ts后缀名的文件会使用ts-loader
                test: /\.ts$/,
                use: ["ts-loader"]
            }
        ]
    }
}
```
