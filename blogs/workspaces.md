---
title: npm、yarn 使用 workspaces
categories: 前端
date: 2024-04-13 18:08:00
tags:
  - npm
  - yarn
---

# npm、yarn 使用 workspaces

## 前言

使用 workspaces 可以进行包的管理。简单来说就是一个项目中，可以分成多个子包，像是 UI 层，业务层等。使用 workspaces 进行分包的话，可以实现@clz/ui 是 ui 层的包，@clz/model 是数据层的包。

## 使用

1. package.json 添加`workspaces`字段。一个是包的根路径，另外的则是子包。

```json
"workspaces": [
    "packages/*",
    "packages/@clz/a"
  ],
```

2. 添加子包
   ![](https://www.clzczh.top/CLZ_img/images/202404131735884.png)

分好 workspaces 后，就可以使用 workspaces 了。例如下面的代码就是在 b workspace 中引入 a workspace 的内容。

`packages/a/index.js`

```js
export const a = 123;
```

`packages/b/index.js`

```js
import { a } from "@clz/a";
console.log(a);
```

> 分好 workspaces 后，还需要执行一下`npm install`操作，把子包添加到 node_modules 中。然后需要执行子包的命令的话，需要携带-w 参数，`npm run xxx -w @clz/a`这种形式去执行子包命令。

`根package.json`

```json
{
  "name": "workspaces",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "type": "module",
  "workspaces": ["packages/*", "packages/@clz/a"],
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev:b": "npm run dev -w @clz/b"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

`@clz/b package.json`

```json
{
  "name": "@clz/b",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "type": "module",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "node ./index.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

执行命令
![](https://www.clzczh.top/CLZ_img/images/202404131742537.png)

## 给子包添加、移除依赖

给子包添加、移除依赖根上面执行子包命令类似，都是通过添加参数`-w`来实现。

```
npm install xxx -w @clz/a
npm uninstall xxx -w @clz/a
npm update xxx -w @clz/a
```

> 子包的依赖在根 package.json 中并不一定会有。但是`npm install`的时候会安装上。

顺便附上 yarn 版本的。(跟 npm 类似，只是`-w`换成`workspace`而已)

```
yarn workspace @clz/a dev
yarn workspace @clz/a add xxx
yarn workspace @clz/a remove xxx
```

> yarn 开启 workspaces 需要 package.json 添加`"private": "true"`。
> 不然会报错。
> ![](https://www.clzczh.top/CLZ_img/images/202404131753867.png)

## 快捷修复路径问题

![](https://www.clzczh.top/CLZ_img/images/202404131758478.png)
快捷修复的方式引入并不是`@clz/a`这种子包的形式。

这个时候可以改一下 ts 的配置。

```
添加ts配置。可以避免绝对路径引入
{
    "compilerOptions": {
        "baseUrl": "./",
        "paths": {
            "@clz/a": [
                "packages/a"
            ]
        }
    }
}
```

![](https://www.clzczh.top/CLZ_img/images/202404131803601.png)
