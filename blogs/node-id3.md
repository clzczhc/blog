---
title: node-id3进行音乐tag编辑
date: 2021-11-18 23:43:39
keywords: nodejs
categories: 小技能
tags:
  - Node.js
---

# 使用 node-id3 对 mp3 进行标签写入

## 前言

> 使用起来很简单，具体可以查看官方的[README](https://github.com/Zazama/node-id3)。这篇文章主要记录一下使用`node-id3`对 mp3 进行标签写入，包括图片插入。

> $\color{red}{在网上看到，对 mp3 进行标签操作好像有可能会引起版权问题，所以仅供玩耍，不要玩火}$。

## 对 mp3 进行标签写入

```js
import path from "path";
import NodeID3 from "node-id3";
import fetch from "node-fetch";

// esm里面没有__dirname
const __dirname = import.meta.url.slice(8, import.meta.url.lastIndexOf("/"));

/**
 * 根据图片url得到图片buffer，并返回node-id3需要的image对象
 * @param {*} imgUrl
 * @returns
 */
const getAlbumImage = async (imgUrl) => {
  const res = await fetch(imgUrl);
  const arrayBuffer = await res.arrayBuffer();
  const imageBuffer = Buffer.from(arrayBuffer);

  const image = {
    mime: "image/jpeg",
    type: {
      id: 3,
      name: "front cover",
    },
    imageBuffer,
  };

  return image;
};

const writeTagsToMp3 = async () => {
  const tags = {
    title: "clz's toy",
    artist: "clz",
    album: "haha",
    image: await getAlbumImage(
      "https://profile-avatar.csdnimg.cn/8540ebf7598f4057967727a8cb84474e_chilanzi.jpg"
    ),
  };

  NodeID3.write(tags, path.join(__dirname, "music", "b.mp3"));
};

writeTagsToMp3();
```

![](https://www.clzczh.top/CLZ_img/images/202402231420662.png)

![](https://www.clzczh.top/CLZ_img/images/202402231433032.png)

## 非 mp3 文件无法添加 tag

比如 m4a 文件，即使重命名为 mp3，并且也能使用媒体播放器进行播放，但是并不是 mp3，所以对它进行标签写入并没有效果。可以通过 ffmpeg 把它转成 mp3 格式。

```js
import ffmpegInstaller from "@ffmpeg-installer/ffmpeg";
import ffprobeInstaller from "@ffprobe-installer/ffprobe";

import ffmpeg from "fluent-ffmpeg";
// 设置后就不需要专门安装ffmpeg了
ffmpeg.setFfmpegPath(ffmpegInstaller.path);
ffmpeg.setFfprobePath(ffprobeInstaller.path);

/**
 * 转换为mp3格式
 * @param {*} sourcePath 要转换的文件路径
 * @param {*} targetPath 转换好的mp3文件路径
 */
const toMp3 = (sourcePath, targetPath) => {
  const command = ffmpeg();
  command.input(sourcePath).save(targetPath);
};
```
