---
title: 工作的一些问题记录（一）
categories: 前端
date: 2023-12-30 16:18:48
tags:
    - docusaurus
    - git
---

# 工作的一些问题记录（一）

## TypeScript

### tsconfig.js配置路径别名

```json
{
  "compilerOptions": {
    "module": "Node16",
    "moduleResolution": "Node16",
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

## keyof typeof工作原理

> 1. typeof可以获取一个值的类型，typeof obj会返回obj的类型。
>    
>    ```ts
>    const obj = {
>      name: 'clz',
>      age: 999
>    };
>    
>    // ObjType: { name: string, age: number }
>    type ObjType = typeof obj;  
>    ```
> 
> 2. `keyof`可以获取一个类型的所有属性名的联合类型。
>    
>    ```ts
>    const obj = {
>      name: 'clz',
>      age: 999
>    };
>    
>    type ObjType = typeof obj;
>    
>    // Objkeys: "name" | "age"
>    type ObjKeys = keyof ObjType;
>    ```
> 
> 3. `keyof`只能对类型使用，而不能对值使用。
>    
>    ![](https://www.clzczh.top/CLZ_img/images/202312262241201.png)

## Docusaurus

### 配置别名

`docusaurus.config.js`

```javascript
const customWebpack = require('./plugins/custom-webpack');

/** @type {import('@docusaurus/types').Config} */
// 上面的类注释内容可以引入类型。因为配置文件是js，默认是没有类型提示的提示
const config = {
  // ...
  plugins: [customWebpack]
}
```

`plugins/custom-webpack/index.js`

```js
const path = require('path');
const resolve = dir => path.resolve(__dirname, dir);

module.exports = function (context, options) {

  return {
    name: "custom-webpack",
    configureWebpack(config, isServer, utils) {

      // windows直接return不生效，但是mac会生效。具体原理不清楚
      config.resolve.alias = {
        ...config.resolve.alias,
        '@': resolve('../../src')
      };

      console.log(config.resolve.alias);

      return {
        resolve: {
          alias: {
            '@': resolve('../../src')
          }
        }
      }
    }
  };
};
```

添加好webpack的alias配置后只是功能能正常使用了，但是还是会类型报错：

这时候就要在`tsconfig.json`中设置路径别名，来解决类型报错：

```json
{
  // This file is not used in compilation. It is here just for a nice editor experience.
  "extends": "@tsconfig/docusaurus/tsconfig.json",
  "compilerOptions": {
    "module": "Node16",
    "moduleResolution": "Node16",
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

### docusaurus无法使用window

> 使用`ExecutionEnvironment.canUseDOM`来判断，只有能使用DOM的时候才会执行需要用到`window`部分的代码。

```javascript
import ExecutionEnvironment from '@docusaurus/ExecutionEnvironment';

if (ExecutionEnvironment.canUseDOM) {   
  useEffect(() => {
  }, [window.location.href]);
}
```

[executionenvironment](https://docusaurus.io/zh-CN/docs/docusaurus-core#executionenvironment)

### dev跟pro两套配置

不同的部分放在`customFields`中。然后`dev`跟`pro`的配置就只有`customFields`这部分不同，可以将相同部分配置抽离。

![](https://www.clzczh.top/CLZ_img/images/202312241852473.png)

![](https://www.clzczh.top/CLZ_img/images/202312241852511.png)

![](https://www.clzczh.top/CLZ_img/images/202312241853527.png)

## git

## cherry-pick

> 可以将某个分支的某次提交复制到当前分支。
> 
> 应用场景：有两条分支`dev`和`master`，分别对应测试环境跟正式环境。测试环境加了几个功能，要求只上线某个功能，其他的后续再上线。（比如测试还没来的测其他的各种原因）

b分支

```bash
git log
commit 1f76a5c7e69de351d1fa9a2549efc00a6cd8ce38 (HEAD -> b)
```

main分支

```bash
git log
commit c522ee68085778e5a2ed793128aa6469ba9a49d0 (HEAD -> main)
```

main分支执行`git cherry-pick 1f76a5c7e69de351d1fa9a2549efc00a6cd8ce38`

```bash
git log
commit 1c82a097ca42e60b8d811267810b5fdd6b5c93c8 (HEAD -> main) 
# 从b分支复制过来的
```

### git查看配置，并清除指定配置

查看配置

```bash
git config --global -l
```

![](https://www.clzczh.top/CLZ_img/images/202312241900576.png)

清除指定配置

```bash
git config --global --unset user.nickname
```

### github ssh无法连接

[在 HTTPS 端口使用 SSH - GitHub 文档](https://docs.github.com/zh/authentication/troubleshooting-ssh/using-ssh-over-the-https-port)

## 包管理器

### 将本地插件安装到node_modules中

> 依赖不再是版本号，而是`file:`前缀，指明是本地。并且要是是文件夹。

`package.json`

```javascript
{
  "dependencies": {
    "custom-webpack": "file:plugins/custom-webpack"
  }, 
}
```

这样就可以很方便地引入插件了。完全不需要管层级问题

```js
const customWebpack = require('custom-webpack');s 
```

## H5

### 伪元素写多行内容

```html
<div class="box">
  <div class="content">content</div>
</div>
```

```css
.box {
  width: 200px;
  height: 200px;
  background-color: pink;
} 

.content::before {
  content: '你好呀！\A赤蓝紫\A';
  color: red;
  white-space: pre;
}
```

伪元素可以通过`\A`换行，前提是设置`white-space`为`pre`

![](https://www.clzczh.top/CLZ_img/images/202312241644178.png)

### svg设置成父元素来设置颜色

```html
<div class="svg-box">
  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 500 250">
    <path d="m 0 0 L 0 250 L 20 250 A 1 1 0 0 1 478 250 L 500 250 L 500 0" fill="currentColor" />
  </svg>
</div>
```

```css
.svg-box {
  color: red;
}
```

![](https://www.clzczh.top/CLZ_img/images/202312241643252.png)

> 关键：fill="currentColor"

### 绝对定位：让子元素的层级比元素低

设置子元素的`z-index`为负数，并且父元素不能设置`z-index`。设置会导致失效。

```html
<div class="box">
  <div class="son"></div>
</div>
```

```css
.box {
  position: relative;
  width: 200px;
  height: 200px;
  background: rgba(0, 0, 0, .3);
}　

.son {
  position: absolute;
  z-index: -2;
  width: 50px;
  height: 50px;
  background-color: pink;
}
```

![](https://www.clzczh.top/CLZ_img/images/202312241657226.png)

### 调试`mouseenter`事件后添加的样式

如果是比较小的一些变化的话，可以通过查看类名等之类的变化来定位元素。能定位就能先定位，再把鼠标移过去看具体的样式变化。

但是如果是`mouseenter`的时候添加的一些dom元素，就没有办法定位了，因为没触发的时候都不在dom树上。

没什么特别好的办法。只有一个小妙招，有一丢丢麻烦。

> 通过Snipaste截图，强制从浏览器失焦，这时候再点击控制台（不要点击网页）就能找到了

### img懒加载

设置`loading`属性为`lazy`。这样子非首屏的图片的优先级就会比较低，并且还有预加载功能。

```js
<img src="./imgs/1x.jpg" alt="" loading="lazy">
```

示例：

```html
<div>
  <img src="./imgs/1x.jpg" alt="" loading="lazy">
</div>
<div>
  <img src="./imgs/2x.jpg" alt="" loading="lazy">
</div>
<div>
  <img src="./imgs/3x.jpg" alt="" loading="lazy">
</div>
```

```css
div {
  height: 200vh;
}
img {
  height: 100%;
}
```

![](https://www.clzczh.top/CLZ_img/images/202312241755176.png)

## background层级

> `background`支持多背景，写在前面的在最上面，写在最后面的在最下面。
> 
> **背景色不能单独作为一层，但是渐变可以**。所以如果需要背景色在背景图片的上层，从而实现蒙版效果的话，可以使用渐变初始值跟渐变结束值一样的渐变层。

```css
div {
  height: 200px;
  background: -webkit-linear-gradient(top, rgba(0, 0, 0, 0.7), rgba(0, 0, 0, 0.7)), url("./imgs/1x.jpg")
}
```

[多个背景的应用 - CSS：层叠样式表 | MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_backgrounds_and_borders/Using_multiple_backgrounds)

[background的层级观念 - 掘金](https://juejin.cn/post/6844903777586118664)

### font-weight: 500 windows不生效

[font-weight 设置导致在 Windows 上网页不显示加粗效果 | ZRONG's BLOG](https://blog.zengrong.net/post/font-weight-500/)

## 杂

### 真机调试

[如何用Safari联调Hybrid APP-CSDN博客](https://blog.csdn.net/weixin_33972649/article/details/85968266)

[Chrome DevTools 远程调试安卓网页](https://zhuanlan.zhihu.com/p/554102971)

> PS：小米手机开启开发者选项需要额外操作才能有开发者选项：设置 -> 我的设备 -> 全部参数 -> 点击7次MIUI版本。点击完成后，才能在更多设置里看到开发者选项。

### 图片操作库

> tinypng（质量较好，每月有限制，单张也有最大5M的限制）
> 
> jimp
> sharp
> gm
