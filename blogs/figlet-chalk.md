---
title: figlet和chalk实现多彩文字
categories: 前端
date: 2023-03-03 16:15:30
tags:
    - JavaScript
---

# figlet和chalk实现多彩文字

## 前言

最近，在搞一个小工具。用脚本架那套工具来实现批量修改文件(夹)、删除文件(夹)的操作。大体功能实现后，决定再加一下`figlet`来炫一下，但是后面发现不太支持配合`chalk`来实线多彩文字，百度大法也没起作用，所以就小试了一下源码大法。（暂时只看懂个大概，但是已经够我的需求了）

figlet用法：[figlet](https://www.npmjs.com/package/figlet)

chalk用法：[chalk](https://www.npmjs.com/package/chalk)

## 实现单色文字

```js
import figlet from 'figlet';
import chalk from 'chalk';

figlet.text('CLZ', {
  font: 'Isometric2'
}, function(err, data) {
  if (err) {
      console.log('Something went wrong...');
      console.dir(err);
      return;
  }

  console.log(chalk.red(data));
});
```

![](https://www.clzczh.top/CLZ_img/images/202303031037954.png)

从上面的例子中，就能发现，输出的文字是由很多小字符组成的。而如果想要实现多彩文字，应该是类似红`C`+蓝`L`+紫`Z`的形式才对。

## 小调试源码

既然，不能直接实现，那就先调试一下源码，看下能不能偷学点东西。

先打一个断点，准备进去看下源码：

![](https://www.clzczh.top/CLZ_img/images/202303031043201.png)

然后，就能发现，调用了一个`loadFont`方法，还贴心的注释了“加载字体，如果加载了，它的数据将被保存再figFonts对象中”。

![](https://www.clzczh.top/CLZ_img/images/202303031055349.png)

进到`loadFont`方法中，就能够知道是怎么加载字体的了。

![](https://www.clzczh.top/CLZ_img/images/202303031058879.png)

![](https://www.clzczh.top/CLZ_img/images/202303031059357.png)

好家伙，难怪不支持中文，原来是提前写好的。

然后，看一下，加载完之后，拿加载的东西干嘛用

![](https://www.clzczh.top/CLZ_img/images/202303031107167.png)

\color{red}{注意：这里需要打个断点，直接单步调试会直接结束，因为`readFile`是异步的}

进去，`parseFont`方法后，可以看到很多很多操作，但是可以先别管，直接跳到函数执行完那里，在那时再看看上面的一些内容变成什么样了，来猜猜干了啥。

![](https://www.clzczh.top/CLZ_img/images/202303031112354.png)

就是把加载的字符串变成数组形式，并且字母还是按照`ASCII`码作为索引来存储的。

并且，`figFont`和`figFonts[fontName]`还是指向同一块堆内存，所以在其他地方访问就能直接通过`figFonts[fontName]`来访问了。

![](https://www.clzczh.top/CLZ_img/images/202303031117801.png)

既然如此，那么我们直接通过`figFonts[fontName][ASCII码]`来拿到我们想要的文字，然后一行一行遍历。以行为单位进行字符串拼接不就行了。事实上，`figlet`的`horizontalSmush`方法就是这样干的。（详情可以自己调试着“玩”）

## 实现多彩文字

既然`figlet`都加载好了字体了，并且还存好在数组里了，那么我们直接在回调里使用就好了。

```js
import figlet from 'figlet';
import chalk from 'chalk';

const fontName = 'Isometric2';
const str = 'CLZ';

figlet.text('', {
  font: fontName
}, function(err, data) {
  if (err) {
      console.log('Something went wrong...');
      console.dir(err);
      return;
  }

  console.log(figlet.figFonts[fontName][str.charCodeAt(0)]);
});
```

接下来就来拼接大法：

```js
import figlet from 'figlet';
import chalk from 'chalk';

const fontName = 'Isometric2';
const str = 'CLZ';

figlet.text('', {
  font: fontName
}, function (err, data) {
  if (err) {
    console.log('Something went wrong...');
    console.dir(err);
    return;
  }


  const figFont = figlet.figFonts[fontName];

  const fig1 = figFont[str.charCodeAt(0)];
  const fig2 = figFont[str.charCodeAt(1)];
  const fig3 = figFont[str.charCodeAt(2)];

  let result = [];

  for (let i = 0; i < fig1.length; i++) {
    result[i] = (chalk.red(fig1[i]) + chalk.blue(fig2[i]) + chalk.magenta(fig3[i]));
  }

  console.log(result.join('\n'));   // 拼接结束后，将元素以换行分隔开
});
```

![](https://www.clzczh.top/CLZ_img/images/202303031146607.png)

> 因为本人并不需要很强的自定义功能，就实现到这了。包括字符串都不是用的循环遍历得到，而是直接枚举法。

顺带分享一下：自己写的小工具（还在优化中，不喜勿喷）：https://github.com/13535944743/tool
