---
title: 工作中的那点事之 杂项
date: 2025-03-22 11:02:05
categories: "前端"
tags:
  - 工作中的那点事
  - docusaurus
---

# 工作中的那点事之 docusaurus

## 前言

> 记录一下之前开发的时候的一些关键步骤、代码

版本：3.7.0

## 国际化

配置文件添加配置：
docusaurus.config.ts

```js
export default {
  i18n: {
    defaultLocale: "zh-CN",
    locales: ["en", "zh-CN"],
    localeConfigs: {
      en: {
        label: "English",
        direction: "ltr",
        path: "en",
      },
      "zh-CN": {
        label: "简体中文",
        direction: "ltr",
      },
    },
  },
  themeConfig: {
    // ...
    navbar: {
      // ...
      items: [
        {
          type: "localeDropdown", //添加选择语言的下拉菜单
          position: "right",
        },
      ],
    },
  },
};
```

### 生成其它语言的文件结构

通过下面的命令生成其他语言的文件结构

```
yarn write-translations --locale en
```

![](https://www.clzczh.top/CLZ_img/images/20250322095101.png)

> i18n 下对应语言的文件夹名称由上面配置的`path`决定，也就是说`en`对应的文件夹甚至可以配置成`zh`。

### 添加文档的翻译文件

添加文档的翻译文件，则放到`docusaurus-plugin-content-docs`文件夹下，分别根据生成的 json 文件创建对应版本的翻译文件夹。如果没有分版本的话，那就放到 current 文件夹下。

![](https://www.clzczh.top/CLZ_img/images/20250322095506.png)

> 配置了多语言，但是部分文档没有翻译的话，则会直接使用默认语言。比如上面`docs`文件夹下还有其他`md`文件，`en`下没有，则会直接用`docs`的内容。

### 查看效果

本地环境无法直接查看多语言的情况，`yarn start`查看默认语言，`yarn run start --locale en`查看其他语言

打包后，查看效果：
![](https://www.clzczh.top/CLZ_img/images/20250322101350.png)
![](https://www.clzczh.top/CLZ_img/images/20250322101421.png)

## 资源引入路径添加 cdn 前缀

**这部分没有用最新版本的`docusaurus`测试，版本是`2.4.3`**

### 编写 webpack 插件

custom-webpack.js

```js
const path = require("path");
const resolve = path.resolve;

module.exports = function (context, options) {
  return {
    name: "custom-webpack",
    configureWebpack(config, isServer, utils) {
      return {
        resolve: {
          alias: {
            "@": resolve("/src"),
          },
        },
        output: {
          publicPath: process.env.PUBLIC_PATH || "/",
        },
      };
    },
  };
};
```

docusaurus.config.js

```js
const customWebpack = require("../plugins/custom-webpack");
export default {
  plugins: [customWebpack],
};
```

通过 webpack 插件可以将大部分静态资源的路径添加`cdn`前缀，但是有几个 css、js 文件不受控制。

### 添加配置`srrTemplate`

上面提到的“有几个 css、js 文件不受控制”，可以通过`srrTemplate`配置解决。

> 搞了好久，还简单看了源码的流程，才发现配置本身就提供`ssrTemplate`。https://github.com/facebook/docusaurus/discussions/4123。有个discussion讨论了好久，实际上就可以通过这个来实现。

template.js

```js
const publicPath = process.env.PUBLIC_PATH || "/";

module.exports = `
<!DOCTYPE html>
<html <%~ it.htmlAttributes %>>
  <head>
    <meta charset="UTF-8">
    <meta name="generator" content="Docusaurus v<%= it.version %>">
    <% it.metaAttributes.forEach((metaAttribute) => { %>
      <%~ metaAttribute %>
    <% }); %>
    <%~ it.headTags %>
    <% it.stylesheets.forEach((stylesheet) => { %>
      <link rel="stylesheet" href="${publicPath}<%= stylesheet %>" />
    <% }); %>
    <% it.scripts.forEach((script) => { %>
      <script src="${publicPath}<%= script %>" defer></script>
    <% }); %>
  </head>
  <body <%~ it.bodyAttributes %>>
    <%~ it.preBodyTags %>
    <div id="__docusaurus"><%~ it.appHtml %></div>
    <%~ it.postBodyTags %>
  </body>
</html>
`;
```

docusaurus.config.js

```js
const ssrTemplate = require("./template");

export default {
  ssrTemplate,
};
```

## 分版

### 生成版本相关文件、文件夹

`yarn docusaurus docs:version 2.0`
![](https://www.clzczh.top/CLZ_img/images/20250322104257.png)

会基于当前版本`docs`生成对应版本，多语言也会生成。不过没有相关的 json 文件，需要重新执行一次`yarn write-translations --locale en`

### 添加配置

docusaurus

```ts
export default {
  themeConfig: {
    // ...
    navbar: {
      // ...
      items: [
        {
          type: "docsVersionDropdown",
          position: "right",
        },
      ],
    },
  },
  presets: [
    [
      "classic",
      {
        docs: {
          // ...
          lastVersion: "current",
          versions: {
            current: {
              label: "latest",
              path: "",
            },
          },
        },
      },
    ],
  ],
};
```

### 效果

![](https://www.clzczh.top/CLZ_img/images/20250322110123.png)

![](https://www.clzczh.top/CLZ_img/images/20250322110151.png)
