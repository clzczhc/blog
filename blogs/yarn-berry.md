---
title: yarn 设置 yarn-berry 启用缓存
categories: 前端
date: 2024-04-13 23:30:00
tags:
  - yarn
---

# yarn 设置 yarn-berry 启用缓存

## 前言

> 公司项目用到了 yarn-berry 来启用缓存，极大地减少了流水线的时间。使用缓存后，就不再需要到网上下载依赖，而是使用缓存的依赖。而缓存的依赖是一些压缩包，安装的时候，把它解压到`node_module`。甚至不需要联网都可以安装依赖

## yarn-berry

简单来说就是，yarn-berry 是高版本的 yarn，而 yarn 默认是 1.x 版本的。而 yarn-berry 对缓存有更好的支持。（本人实际上 yarn 1.x 的缓存没启用成功，很麻烦。并且既然 yarn-berry 支持的更好，就没必要搞了）

> - [yarn 1.x](https://github.com/yarnpkg/yarn)
> - [yarn-berry](https://github.com/yarnpkg/berry)

### 启用

可以看，yarn-berry 提供的文档，[yarn-berry](https://yarnpkg.com/migration/guide)

![](https://www.clzczh.top/CLZ_img/images/202404132353320.png)
没启用 yarn-berry 时，yarn 的版本

> `yarn set version berry`，执行一下命令，启用`yarn-berry`。`berry`实际上也可以换成 yarn 的版本号。
> ![](https://www.clzczh.top/CLZ_img/images/202404132354310.png)

命令执行后，package.json 会添加字段：

```json
"packageManager": "yarn@4.1.1"
```

并且，在根目录下会生成一个.yarn 文件夹跟 yarn 的配置文件`.yarnrc.yml`。

如果`.yarnrc.yml`并没有`enableGlobalCache`这个字段，那么添加上`enableGlobalCache: false`。这样子缓存就是在项目内，而不是电脑其他地方。

添加新的依赖时，会在`.yarn`文件夹中生成一个`cache`文件夹，里面的内容是以来的压缩包。
![](https://www.clzczh.top/CLZ_img/images/202404140008741.png)

有意思的一点是，并不会生成`node_modules`，而是会有一个`.pnp.cjs`文件。这就是 yarn-berry 的[零安装](https://yarnpkg.com/features/caching#zero-installs)。

但是，会有一些问题：
![](https://www.clzczh.top/CLZ_img/images/202404140016201.png)
依赖能够正常引入，但是执行下面代码会报错：Cannot find module 'lodash'

```js
const lodash = require("lodash");
console.log(lodash.cloneDeep);
```

因为 yarn 的这个 Pnp 模式是 yarn 的，并不是 node 本身就支持的。所以执行命令需要换成`yarn node index.js`。
![](https://www.clzczh.top/CLZ_img/images/202404140019064.png)

可以查看官方提供的一些[注意事项](https://yarnpkg.com/migration/pnp#what-to-look-for)

### node_modules 模式

pnp 模式会有一些问题，所以有可能会需要回归最开始的 node_modules 模式。
`.yarnrc.yml`添加字段：`nodeLinker: node-modules`。

> `.yarnrc.yml`的字段可以查看[官方手册](https://www.yarnpkg.cn/configuration/yarnrc)

添加后，再重新`yarn`就可以添加`node_modules`了，并且还会帮忙删除`.pnp.cjs`文件。

把.yarn 文件夹也添加到远程仓库中，后面安装缓存中有的依赖就只需要解压即可。(直接 yarn 就会解压)

> 有一些特殊情况还能通过这种方式把特殊依赖带上，比如公司内部依赖在外网无法安装。（一些 github 项目需要用到公司内部的依赖的话，之前就是有个项目在 github 上分为公开、私有仓库两部分，私有仓库需要用到公司的依赖）

### .yarnrc.yml

```yml
enableGlobalCache: false

nodeLinker: node-modules

yarnPath: .yarn/releases/yarn-4.1.1.cjs
```

## 参考

- https://blog.csdn.net/fenglailea/article/details/121467992(yarn2部分)

- https://yarnpkg.com/

- https://www.yarnpkg.cn/configuration/yarnrc
