---
title: Node打开选择文件夹弹框
categories: 小技能
date: 2023-03-22 11:01:57
tags:
    - Node
---

# Node打开选择文件夹弹框

## 前言

用脚手架的那套东西写了一个工具，但是想要一个用Node去打开选择文件夹弹框的效果，来设置操作根目录。但是，Node本身没有这个API。

## node执行python脚本

Node本身没有提供打开选择文件夹弹框的API，但是Python的`tkinter`是有这个功能的。所以可以用Python写好脚本来打开选择文件夹，然后通过Node来执行python脚本。

Python脚本也是非常的简单。

```python
import tkinter as tk
import tkinter.filedialog

dirPaths = tkinter.filedialog.askdirectory()

if(len(dirPaths) == 0):
  print('None')
else:
  print(dirPaths)
```

Node执行Python脚本需要通过Node提供的`child_process`来创建子进程（`exec`），它会将紫禁城的输入以回调函数参数的形式一次性返回。

### ESM里使用`__dirname`

因为我用的是`ESM`模式写的，所以是不能直接使用`__dirname`的。这里稍微吹一下下ChatGPT。![](https://www.clzczh.top/CLZ_img/images/202303101405304.png)

> 启用`ESM`模式则是在`package.json`中，添加`type: "module"`

当然，答案有点小瑕疵，实际上得到的是当前文件的绝对地址，并且前面会有文件协议。所以需要进行一些处理。

![](https://www.clzczh.top/CLZ_img/images/202303101449081.png)

![](https://www.clzczh.top/CLZ_img/images/202303101450972.png)

### 代码

```js
import { exec } from 'child_process';
import { EOL } from 'os';  // 回车、换行，通过JSON.stringify()能够观测到
import path from 'path';

const dirname = import.meta.url.slice(8, import.meta.url.lastIndexOf('/'));

const p = new Promise((resolve, reject) => {
  exec(path.join(dirname, 'dialog.py'), (err, stdout, stderr) => {
    if (err) {
      reject(new Error(err));
    } else if (stderr) {
      reject(new Error(stderr));
    } else if (stdout) {
      const result = stdout.trim().split(EOL).toString();  // 将返回的结果去掉前后的空格以及回车换行
      resolve(result);
    }
  })
});

p.then(val => {
    console.log(val);
})
```

![](https://www.clzczh.top/CLZ_img/images/202303101504714.gif)

## 中文路径问题

`python`输出中文是会乱码的，所以当我们选择的路径有中文的话，就会出现问题。

![](https://www.clzczh.top/CLZ_img/images/202303101511099.gif)

在网上找到一些解决方案说是改环境变量。但是，本人想要的效果是只需要下载工具，就能直接使用。而不需要手动修改。所以最好的方案还是在代码上做文章。

最后，功夫不负有心人。找到一个改变标准输入输出的默认编码的方案。

```python
import io
import sys

#改变标准输出的默认编码
sys.stdout= io.TextIOWrapper(sys.stdout.buffer,encoding='utf8')
```

所以，需要小修改Python代码，添加上面的内容。

![](https://www.clzczh.top/CLZ_img/images/202303101515280.png)

![](https://www.clzczh.top/CLZ_img/images/202303101518197.gif)

## 将python程序打包成`exe`文件

上面通过`Node`来执行`python`脚本，实际上是需要电脑有安装`Python`，但是这样子当然是不太好的，有种捆绑的感觉。和Python的耦合度过高，所以最终考虑将`python`程序打包成`exe`文件。

将`py`打包为exe文件需要依赖`pyinstaller`。

更多：[如何将python程序打包成exe文件_py打包成exe_一朝乐的博客-CSDN博客](https://blog.csdn.net/qq_56418482/article/details/127338778)

安装`pyinstaller`可能会遇到的问题以及解决方案：

[如何将python程序打包成exe文件_py打包成exe_一朝乐的博客-CSDN博客](https://blog.csdn.net/qq_56418482/article/details/127338778)

![](https://www.clzczh.top/CLZ_img/images/202303101535010.png)

除了`dist`外，生成的东西都能删掉，因为其他都是都是编译的时候生成的。只有`dist`是我们有我们想要的`exe`文件。

直接双击生成的`exe`文件，也会打开选择文件夹弹框。

![](https://www.clzczh.top/CLZ_img/images/202303101541813.gif)

代码也需要修改成执行`exe`文件，而不再是`python`文件。

```js
import { exec } from 'child_process';
import { EOL } from 'os';  // 回车、换行，通过JSON.stringify()能够观测到
import path from 'path';

const dirname = import.meta.url.slice(8, import.meta.url.lastIndexOf('/'));

const p = new Promise((resolve, reject) => {
  exec(path.join(dirname, 'dist', 'dialog.exe'), (err, stdout, stderr) => {
    if (err) {
      reject(new Error(err));
    } else if (stderr) {
      reject(new Error(stderr));
    } else if (stdout) {
      const result = stdout.trim().split(EOL).toString();  // 将返回的结果去掉前后的空格以及回车换行
      resolve(result);
    }
  })
});

p.then(val => {
    console.log(val);
})
```

效果和前面一样。

还可以编写一个sh文件，帮我们生成`exe`文件，并且删除编译中生成的一些其他文件。

run.sh

```shell
#!/bin/bash
pyinstaller -F dialog.py
rm build/ -rf
rm dialog.specs
```
