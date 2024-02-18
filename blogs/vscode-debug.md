---
title: VSCode前端调试的几种场景
categories: 前端
date: 2023-03-22 11:04:13
tags:
    - 调试
---

# VSCode前端调试的几种场景

## 前言

> 看了神光的前端调试秘籍通关，以及查询一些资料后总结的。这本小册真的强烈推荐，非常有用，非常有用。

VSCode实现调试主要就是靠的编写`lauch.json`文件来实现。下面就来看看几种场景。

## 普通页面

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <script>
    function add(a, b) {
      return a + b;
    }

    debugger;

    add(1, 2);
  </script>
</body>
</html>
```

创建`lauch.json`文件，选择Web应用。

![](https://www.clzczh.top/CLZ_img/images/202303032053537.png)

![](https://www.clzczh.top/CLZ_img/images/202303032110075.png)

之后点击上图左上角的绿色按钮即可打开页面，并且进入打好的断点。

![](https://www.clzczh.top/CLZ_img/images/202303032117277.png)

如果是开启web服务，则不是通过`file`属性，而是变成`url`和`webRoot`。（这里的`webRoot`不太清楚具体含义，没找到合理的说法，而且删掉也可以正常使用）

```json
{
  "name": "Launch Chrome",
  "request": "launch",
  "type": "chrome",
  "url": "http://localhost:5500",
  "webRoot": "${workspaceFolder}"
},
```

顺便解释一下，调试的几种操作：

![](https://www.clzczh.top/CLZ_img/images/202303032331370.png)

- 继续/暂停：可以用来从一个断点跳到另一个断点

- 单步跳过：不会进入函数内部

- 单步调试：会进入函数内部

- 单步跳出：跳出当前函数

- 重启：重新开启调试

- 停止：结束调试

## Node.js调试

Node.js的调试基本和前面也是一样的，只是添加配置选择`Node.js：启动程序`。

```json
{
  "name": "Launch Program",
  "program": "${workspaceFolder}/index.js",
  "request": "launch",
  "skipFiles": [
    "<node_internals>/**"
  ],
  "type": "node"
}
```

> - `program`: 调试入口文件地址
> 
> - `skipFiles`：跳过node核心模块。如果想要跳过`node_modules`则可以添加`"${workspaceFolder}/node_modules/**"`

## 调试脚本启动

有用过`Vue`的应该都知道，启动项目是通过`npm run dev`这种形式启动的，而这种脚本启动也是可以调试滴。

### 小谈`npm run dev`

首先，看一下`package.json`的`script`字段。

![](https://www.clzczh.top/CLZ_img/images/202303042006991.png)

也就是说，当我们执行`npm run dev`的时候，实际上是相当于执行`vite`。那我们直接使用`vite`的话，会怎样呢？

![](https://www.clzczh.top/CLZ_img/images/202303042008247.png)

这时候，因为操作系统里并没有`vite`这个指令，所以会报错。那么为啥运行`npm run dev`能成功呢？

这是因为当我们`npm install`时，会在`node_modules/.bin`文件夹下创建好了`vite`的可执行文件。所以，`npm run dev`真正执行的其实是`node_modules/.bin`文件夹下的可执行文件。

顺带一提，`npx ***`也是执行的`.bin`文件夹下的可执行文件，所以执行`npx vite`也能得到同样的结果。

了解更多：[三面面试官：运行 npm run xxx 的时候发生了什么？ - 掘金](https://juejin.cn/post/7078924628525056007)

### 调试

知道真正执行的位置后，想调试自然就得去看看啦，不然都没法打断点。

![](https://www.clzczh.top/CLZ_img/images/202303042016369.png)

继续跟着这个路径去打断点。

![](https://www.clzczh.top/CLZ_img/images/202303042017157.png)

接着就是添加调试配置。调试这种脚本启动，需要选择的配置类型就是`Node.js: 通过npm启动`。

```json
{
  "name": "Launch via NPM",
  "request": "launch",
  "runtimeArgs": [
    "run-script",
    "dev"
  ],
  "runtimeExecutable": "npm",
  "skipFiles": [
    "<node_internals>/**"
  ],
  "type": "node"
}
```

![](https://www.clzczh.top/CLZ_img/images/202303042022716.png)

接着，调试启动，就断往我们刚刚打断点的地方了。

![](https://www.clzczh.top/CLZ_img/images/202303042024066.png)

## 调试上线页面

使用Vue开发时，当我们部署打包项目用于上线时，代码会被打包混淆。这种情况下，这种情况下就很难调试。

比如：

![](https://www.clzczh.top/CLZ_img/images/202303052047812.png)

![](https://www.clzczh.top/CLZ_img/images/202303052136132.png)

这时候，就需要用到`sourcemap`来辅助调试了。`sourcemap`其实就是一个信息文件，存着一些位置信息，即转换后的代码的每个位置，所对应的转换前的位置。

更多可以查看：[JavaScript Source Map 详解 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2013/01/javascript_source_map.html)

那么怎么生成`sourcemap`文件呢？修改一下配置信息就可以了，下面用还是用Vue3的项目来搞一波。

![](https://www.clzczh.top/CLZ_img/images/202303052212544.png)

修改后，再`build`就能看到`sourcemap`文件了

![](https://www.clzczh.top/CLZ_img/images/202303052214157.png)

可以看到，对应生成的js文件会关联上`sourcemap`

之后再添加`sourcemap`文件就行了。

![](https://www.clzczh.top/CLZ_img/images/202303052203988.png)

添加后，**不刷新**，再到控制台报错，就是正常的报错了。

![](https://www.clzczh.top/CLZ_img/images/202303052204214.png)

顺带一提：还可以将`sourcemap`的是值设置为：`inline`。这样子就不会生成`sourcemap`文件。而是`js`文件最后直接携带着`sourcemap`信息。

![](https://www.clzczh.top/CLZ_img/images/202303052222701.png)

而将值设置为`hidden`，就会生成`sourcemap`，但是并不会关联`sourcemap`。这样子，把`sourcemap`文件上传到错误管理平台，就能够后续报错时，及时定位错误位置对应的源码。

## 调试Vue源码

![](https://www.clzczh.top/CLZ_img/images/202303052302658.gif)

可以看得出来，我们调试时，看到的实际上是打包后的Vue，而不是源码。这样子看起来会很费劲。那么怎么调试Vue源码呢？

上面已经介绍过`sourcemap`的作用了。所以我们想要调试Vue源码，首先得拿到`sourcemap`才行。

npm下载的`Vue`是不携带`sourcemap`的，所以需要自己`clone`源码，修改配置后，再`build`。

```
git clone https://github.com/vuejs/core vue3
```

直接build是没有效果的，需要添加参数`--sourcemap`。

`pnpm run build --sourcemap`后就能看到有`sourcemap`生成了。

![](https://www.clzczh.top/CLZ_img/images/202303070024740.png)

想要调试Vue源码，就将`runtime-core`目录下的`dist`复制到`vue`项目的`node_modules`下的`dist`。

![](https://www.clzczh.top/CLZ_img/images/202303070032504.png)

注意：这时候直接调试可能还是不是源码，需要先删除

vite的缓存`.vite`。

![](https://www.clzczh.top/CLZ_img/images/202303071559148.png)

然后，再去调试，就能调试Vue源码了。

![](https://www.clzczh.top/CLZ_img/images/202303071602884.gif)

但是，这个时候是没有办法编辑源码的。

![](https://www.clzczh.top/CLZ_img/images/202303071604623.png)

根据调用栈里的路径就能发现，这个路径实际是不存在的，所以自然就没有办法编辑。

![](https://www.clzczh.top/CLZ_img/images/202303211818150.png)

![](https://www.clzczh.top/CLZ_img/images/202303211820314.png)

这是因为Vue生成的sourcemap文件中，存转换前的文件的`sources`默认存的是相对路径。

![](https://www.clzczh.top/CLZ_img/images/202303220955454.png)

Sourcemap文件结构可以查看：[Source map 超详细学习攻略_番茄出品_reverse-sourcemap_upward_tomato的博客-CSDN博客](https://blog.csdn.net/wswq2505655377/article/details/127721558)

所以，可以通过VSCode的`ctrl + H`大法，将要用到的sourcemap的相对路径改为绝对路径。

![](https://www.clzczh.top/CLZ_img/images/202303220959476.png)

改成绝对路径之后，理论上就应该成功了。但是，vite貌似搞了一波骚操作，会强行将绝对路径也变成相对路径。（这里判断是vite搞的操作，是因为通过vue-cli搭建的Vue3调试不会出现这个问题）

![](https://www.clzczh.top/CLZ_img/images/202303221003757.png)

解决方案就是添加`optimizeDeps`配置，强制排除依赖项"vue"（解决方案是看的神光大佬的小册评论区知道的）

![](https://www.clzczh.top/CLZ_img/images/202303221007640.png)

![](https://www.clzczh.top/CLZ_img/images/202303221010390.png)

大功告成。。。

### 小优化

我们上面修改成绝对路径是通过手动修改，但是如果需要修改的部分很多就会很麻烦。（全局修改又有可能会影响到其他文件）

这个时候可以修改一下Vue3的build脚本，把相对路径改为绝对路径。

因为Vue3是使用`rollup`打包，而rollup可以通过设置`sourcemapPathTransform`属性来实现设置`sourcemap`的路径为绝对路径。

```js
output.sourcemapPathTransform = (relativeSourcePath, sourcemapPath) => {
  const newSourcePath = path.join(
    path.dirname(sourcemapPath),
    relativeSourcePath
  )
  return newSourcePath
}
```

![](https://www.clzczh.top/CLZ_img/images/202303221045629.png)

> `relativeSourcePath`是源码相对于sourcemap文件的相对路径，如"../src/errorHandling.ts"，而`sourcemapPath`是sourcemap文件的绝对路径。
> 
> 所以可以通过`path.dirname`获得sourcemap的目录路径，然后再通过`path.join`连接找到的目录路径和相对路径来得到源码文件的绝对路径。

这里可以通过上面的调试脚本启动的方法来验证：

VSCode调试配置：

```json
{
  "name": "Launch via NPM",
  "request": "launch",
  "runtimeArgs": [
    "run-script",
    "build",
    "--sourcemap"
  ],
  "runtimeExecutable": "pnpm",
  "skipFiles": [
    "<node_internals>/**"
  ],
  "type": "node"
}
```

![](https://www.clzczh.top/CLZ_img/images/202303221054730.png)
