---
title: fluent-ffmpeg + worker实现视频切片合成视频
categories: 小技能
date: 2022-11-02 11:27:46
tags:
    - ffmpeg
---

# fluent-ffmpeg + worker实现切片合成视频

## 前因

最近发现之前在B站下载的视频，有一些突然变成大会员才能看了。(我下载的时候，还是都能看的。把我下载的文件给加密了，想逼我充大会员，这谁忍得了)。于是，决定把之前下载的文件都给保存到自己的硬盘中。但是量有点小大，20G。所以就排除了用网上的下载B站视频的方法。于是上网搜索了一下，然后发现了音视频开发库中的王者**ffmpeg**。

## ffmpeg极简使用

B站下载视频的地址：Android\data\tv.danmaku.bili\download

```shell
ffmpeg -i 0.blv -c copy o1.mp4
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202211011053607.png)

```shell
ffmpeg -i video.m4s -i audio.m4s -c copy o2.mp4
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202211011056743.png)

上面两种都只是只有一个视频文件，其中第二种是视频和音频分开的。

所以接下来就来试一下多个文件的。

```shell
ffmpeg -f concat -i file.txt o.mp4
```

file.txt是一个文件列表，依次存储输入文件：

```
file 0.blv
file 1.blv
file 2.blv
file 3.blv
file 4.blv
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202211011235025.png)

## fluent-ffmpeg

`fluent-ffmpeg`对FFmpeg的命令行进行了封装，抽象为我们常用的方法和API。

下面就稍微解释一下本次懒人工具用到的几个方法。

`input()`：指定输入（不仅可以指定视频，也可以指定音频）

```js
ffmpeg()
  .input('input1.avi')
  .input('input2.avi');
```

`save()`：将输出保存到文件

```js
ffmpeg()
  .input('input.avi')
  .save('output.mp4');
```

`mergeToFile()`：连接多个输入

```js
ffmpeg()
  .input('input1.avi')
  .input('input2.avi')
  .mergeToFile('output.mp4');
```

`save()`和`mergeToFile()`的区别就是`mergeToFile()`是当有多个视频文件需要合成时，连接，而`save()`则是一个视频（也可以包括音频文件）。

## 实现代码

代码可能写的有点拉。勿喷。

入口文件`index.js`

这里引入了一个`findDeepest`方法。就是用来递归出当前目录以及子目录下的所有文件。

```js
const path = require('path');
const { findDeepest, createVideoDir } = require('./utils.js')

// 创建Video文件夹
createVideoDir();
findDeepest(path.join(__dirname));
```

工具文件`utils.js`

```js
const fs = require('fs');
const path = require('path');
const {
  Worker
} = require('worker_threads');

let dirName = '';
let fileName = '';

// 合成视频的数组：用于Worker
const generateData = [];

const detailsJson = 'entry.json';

function findDeepest(way) {
  /*
   * 1. 首先获取递归下的文件列表（包括文件和文件夹）。
   * 如果没有文件夹，就调用`generateMp4`方法生成mp4。
  */
  const files = fs.readdirSync(way);

  if (files.every(file => !fs.statSync(path.join(way, file)).isDirectory())) {
    generateMp4(way, files);

    return;
  }

  /*
   * 2. 如果文件列表里包含`entry.json`文件，
   * 那么就调用`getDirAndFileName`方法获取文件夹名、文件名（用全局变量来存储）
  */
  if (files.includes(detailsJson)) {
    getDirAndFileName(path.join(way, files[files.indexOf(detailsJson)]));
  }

  /*
   * 3. 遍历文件列表，如果是文件夹，并且不是`node_modules`，则递归调用`findDeepest`方法
  */
  files.forEach(file => {
    if (fs.statSync(path.join(way, file)).isDirectory() && file != "node_modules") {
      findDeepest(path.join(way, file));
    }
  })

  /*
   * 4. 如果files包含package.json，证明递归完了，并且回退到最初的起点了。
   *   这个时候开启Worker来真正合成视频
  */ 
  if (files.includes('package.json')) {
    let count = Math.ceil(generateData.length / 10);

    while (generateData.length > 0) {
      const workerData = generateData.splice(0, count);

      const worker = new Worker('./worker.js');
      worker.postMessage(workerData);

      worker.on('message', (msg) => {
        if (msg === 'done') {
          worker.terminate();
        }
      })
    }
  }
}

// 获取视频文件夹名称、文件名称
function getDirAndFileName(filePath) {
  let fileData = fs.readFileSync(filePath, 'utf8');

  dirName = JSON.parse(fileData).title;
  fileName = JSON.parse(fileData).page_data.part;
}

// 生成Mp4
function generateMp4(way, files) {

  let types = ['blv', 'm4s'];

  files = files.filter(file => {
    return types.some(type => file.endsWith(type));
  })


  if (files.length === 0) {
    return;
  }

  files.sort((a, b) => {
    return (+a.split('.')[0]) - (+b.split('.')[0]);
  })

  generateData.push({
    way,
    files,
    dirName,
    fileName
  });
}

// 创建Video文件夹
function createVideoDir() {
  if (!fs.readdirSync(path.join(__dirname)).includes('Video')) {
    fs.mkdirSync(path.join(__dirname, 'Video'));    
  }
}

module.exports = {
  findDeepest,
  createVideoDir
}
```

解析：

1. 首先获取递归下的文件列表（包括文件和文件夹）。如果没有文件夹，就调用`generateMp4`方法生成mp4

2. 如果文件列表里包含`entry.json`文件，那么就调用`getDirAndFileName`方法获取文件夹名、文件名（用全局变量来存储）

3. 遍历文件列表，如果是文件夹，并且不是`node_modules`，则递归调用`findDeepest`方法

4. 如果files包含package.json，证明递归完了，并且回退到最初的起点了。这个时候开启Worker来真正合成视频。这里的做法是：将前面调用`generateMp4`方法时，存起来的生成视频的数据数组分成10组。分别开启10个Worker，每个Worker负责生成1组的视频。如果收到`done`的消息，则调用`worker.terminate()`终止worker。

### utils.js下的方法简单介绍

`getDirAndFileName`：因为视频的信息时存在`entry.json`文件中的，所以需要获取该文件下的信息。这里本人用的是`title`属性作为文件夹名称，`page_data.part`作为文件名称。

```js
// 获取视频文件夹名称、文件名称
function getDirAndFileName(filePath) {
  let fileData = fs.readFileSync(filePath, 'utf8');

  dirName = JSON.parse(fileData).title;
  fileName = JSON.parse(fileData).page_data.part;
}
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202211012257324.png)

`generateMp4`：首先过滤，获取视频格式、音频格式的文件。之后调用`sort`方法排序。最后给`合成视频的数组generateData`添加数据。

```js
// 生成Mp4
function generateMp4(way, files) {

  let types = ['blv', 'm4s'];

  files = files.filter(file => {
    return types.some(type => file.endsWith(type));
  })


  if (files.length === 0) {
    return;
  }

  files.sort((a, b) => {
    return (+a.split('.')[0]) - (+b.split('.')[0]);
  })

  generateData.push({
    way,
    files,
    dirName,
    fileName
  });
}
```

`createVideoDir`：创建Video文件夹。如果不存在Video文件夹，那就创建。

```js
// 创建Video文件夹
function createVideoDir() {
  if (!fs.readdirSync(path.join(__dirname)).includes('Video')) {
    fs.mkdirSync(path.join(__dirname, 'Video'));    
  }
}
```

### worker.js

```js
const fs = require("fs");
const path = require("path");

const { parentPort } = require("worker_threads");

const ffmpegInstaller = require("@ffmpeg-installer/ffmpeg");
const ffprobeInstaller = require("@ffprobe-installer/ffprobe");

const ffmpeg = require("fluent-ffmpeg");
ffmpeg.setFfmpegPath(ffmpegInstaller.path);
ffmpeg.setFfprobePath(ffprobeInstaller.path);

const disbleReg = /\/|:|\*|\?|"|<|>|\|/g;

let dirName = "";
let fileName = "";

parentPort.on("message", async (workerData) => {

  for (let i = 0, len = workerData.length; i < len; i++) {

    let data = workerData[i];

    dirName = data.dirName.replace(disbleReg, "-");
    fileName = data.fileName.replace(disbleReg, "-");

    createDir();

    await createFile(data.way, data.files);

    if (i === len - 1) {
      // worker执行结束通知主进程
      parentPort.postMessage("done");
    }
  }
});

// 创建次级文件夹
function createDir() {
  const files = fs.readdirSync(path.join(__dirname, "Video"));

  if (!files.includes(dirName)) {
    fs.mkdirSync(path.join(__dirname, "Video", dirName));
  }
}

// ffmpeg合成视频
function createFile(way, files) {
  return new Promise((resolve, reject) => {
    // 如果原本就有这个文件了
    if (fs.readdirSync(path.join(__dirname, "Video", dirName)).includes(`${fileName}.mp4`)) {
      resolve('done');
      return;
    }

    let ff = ffmpeg();

    ff.on("end", () => {
      console.log('end');
      resolve('done');
    });

    for (const file of files) {
      ff.input(path.join(way, file));
    }

    console.log(path.join(__dirname, "Video", dirName, `${fileName}.mp4`));

    if (files[0].endsWith("blv") && files.length > 1) {
      ff.mergeToFile(path.join(__dirname, "Video", dirName, `${fileName}.mp4`));
    } else {
      ff.save(path.join(__dirname, "Video", dirName, `${fileName}.mp4`));
    }
  });
}
```

> 本项目添加了`@ffmpeg-installer/ffmpeg`、`@ffprobe-installer/ffprobe`库。它们能为当前平台安装ffmpeg二进制文件，这样子的话，还能够在多个环境中使用（包括远程环境）。所以不需要在电脑安装ffmpeg，并且设置环境变量，但是需要先调用`ffmpeg.setFfmpegPath`、`ffmpeg.setFfprobePath`设置路径。

解析：

1. worker引入`parentPort`，监听主线程的信息。遍历数据，将不合法符号修改为'-'。

2. 创建文件夹、创建文件。如果数据遍历完了，通知主线程停止该Worker线程。

这里再讲一下实际合成视频的部分。

原理很简单，就是遍历文件，调用`input()`方法来添加输入。然后判断是不是有多个视频文件，如果是，则调用`mergeToFile()`。否则，调用`save()`。并且通过添加`end`事件的回调函数和`Promise`来实现，如果合成完毕，才进行下一个视频的合成。这样子就能做到，只有10个线程在合成视频。

实现效果：

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202211012328942.png)

## 总结

递归获取所有视频信息，平均分给10个Worker，开启Worker实际执行合成视频操作。

## 仓库地址

有需要可以查看整个部分的代码：运行只需要使用`node index.js`命令即可。需要合成的视频文件夹就放在项目根目录中，运行完成后，会在根路径生成一个Video文件夹。

[GitHub - 13535944743/bilibili_ffmpeg](https://github.com/13535944743/bilibili_ffmpeg)

我的博客即将同步至腾讯云开发者社区，邀请大家一同入驻：https://cloud.tencent.com/developer/support-plan?invite_code=2q6bhd3w8qm8o
