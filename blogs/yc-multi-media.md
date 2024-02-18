---
title: Web多媒体笔记
date: 2022-02-09 15:52:43
keywords: Web多媒体
categories: "前端"
tags:
  - 青训营
  - Web多媒体
---

# Web 多媒体笔记

参加字节跳动的青训营时写的笔记。这部分是刘立国老师讲的课。

## 1. 前置知识

### 1.1 图像基本概念

**图像分辨率**：图像的像素数据，指在水平和垂直方向上图像所具有的像素个数

**图像深度**：指存储每个像素所需要的比特数。图像深度决定了图像的每个像素可能的颜色数，或可能的灰度数(单色图像)。例如彩色图像每个像素用 R, G, B 三个分量来表示，每个分量用 8 为所以像素深度是 24 位，可以表示的颜色数目是 2^24。单色图像每个像素需要 8 位，则图像的像素深度是 8 位，灰度数目为 2^8。

**图像的大小不仅要看图像的分辨率，还要看图像深度**

### 1.2 视频基本概念

**分辨率**：每一帧的图像分辨率

**帧率**：视频单位时间内包含的视频帧的数量

**码率**：视频单位时间内传输的数据量，一般用 kbps(千位每秒)表示。

**视频的大小不仅要看图像的分辨率，还要看码率**

## 2. 编码

### 2.1 为什么需要编码

**如果不编码**：

假设一部电影的每一帧的分辨率都是 1920 × 1080。

那么每一帧的大小为 1920 _ 1080 _ 24 / 8 = 6220800 byte = 5.9M (24 就是上面的图像像素深度)

假设帧率为 30FPS，时长为 90 分钟，那么这部电影大小就是 5.9 _ 30 _ 60 \* 90 = 933G(我的电脑竟一部电影都下不了)

### 2.2 DTS 和 PTS

**DTS**：解码时间戳，用来告诉播放器该在什么时候解码这一帧的数据

**PTS**：显示时间戳，用来告诉播放器该在什么时候显示这一帧的数据。

**I 帧**：帧内编码帧、关键帧，不需要参考其他帧。

**P 帧**：前向预测编码帧。P 帧由它的**前一 P 帧或 I 帧**预测而来。

**B 帧**：双向预测内插编码帧。B 帧由它的\*\*前一帧以及后一帧预测而来。

实例：

![image-20220126205313369](https://s2.loli.net/2022/01/26/OyhRCjGIPgB5Xz8.png)

首先，<b style="color: red">PTS 是 1、2、3、4、5、6</b>

因为 P 帧需要由前一 P 帧或 I 帧预测而来，I 帧不需要参考其他帧。所以先解码第 1 帧，解码完第 1 帧后，第 2 帧的 P 帧才可以解码，依次类推。所以，<b style="color: red">DTS 是 1、2、3、4、5、6</b>

![image-20220126205857425](https://s2.loli.net/2022/02/09/2Xchkan9CtLxoAs.png)

首先，<b style="color: red">PTS 是 1、2、3、4、5、6</b>

由上一个例子，依次解码第 1 帧、第 2 帧，想继续解码第 3 帧，发现第 3 帧是 B 帧，所以就去解码第 3 帧的后一帧(第 4 帧)，所以 DTS 的前面部份是 1、2、4、3。同理，解码第 5 帧又会先去解码第 6 帧。所以，<b style="color: red">DTS 是 1、2、4、3、6、5</b>

### 2.3 GOP

GOP(group of picture)：两个 I 帧之间的间隔

![image-20220126210639852](https://s2.loli.net/2022/02/09/cJTMGYsK6gHxQRw.png)

**每个 group 的第一帧是 I 帧(关键帧)，每个 GOP 内的帧的解码不依赖于其他 GOP 的帧。**

GOP 一般 2 到 4s。首先，I 帧是帧内压缩，占用存储空间大。所以 I 帧不适宜过多。如果 I 帧太少，也不行。比如一共有 1000 帧，我需要解码第 500 帧，如果只有一个 I 帧，则要解码第 1 帧、第 2 帧...第 499 帧。(这个场景可能是用户点击进度条时)

### 2.4 各种冗余

#### 2.4.1 空间冗余

![image-20220126212737399](https://s2.loli.net/2022/01/26/yXjMlaoPzBJ4Q95.png)

<b style="color: red">重复的只存储一次</b>

#### 2.4.2 时间冗余

![image-20220126212937820](https://s2.loli.net/2022/02/09/o1bQ2UROdHTaijE.png)

<b style="color: red">只多存储有变化的</b>

#### 2.4.3 编码冗余

![image-20220126213139572](https://s2.loli.net/2022/01/26/VZ4dhGtQ6nrz9FD.png)

<b style="color: red">不同像素值出现的概率不同，概率高的用的字节少，概率低的用的字节多</b>(霍夫曼编码)

上面的 demo：每一个像素值需要 24 位。上面例子，蓝色用 1 表示，白色用 0 表示。则只需要 2 位。

#### 2.4.4 视觉冗余

![image-20220126213341161](https://s2.loli.net/2022/02/09/2ki7t8WZadsrzq6.png)

<b style="color: red">人的视觉系统对某些细节不敏感，所以可以把人眼看不到的部分去除掉。</b>

### 2.5 编码流程

![image-20220126213622028](https://s2.loli.net/2022/01/26/WDFAz7kehPiut34.png)

### 2.6 编码格式

![image-20220126213747091](https://s2.loli.net/2022/01/26/1xE6IotejvUFGrp.png)

## 3. 封装

封装格式的主要作用是把视频码流和音频码流按照一定的格式存储在一个文件中。(可能还有字幕信息)

![image-20220126214123491](https://s2.loli.net/2022/01/26/ZoDWdyXTUgjA4LN.png)

## 4. 多媒体元素和扩展 API

### 4.1 video 和 audio

```html
<body>
  <button onclick="getLength()">获取视频长度</button>
  <p></p>

  <video
    src="http://lf9-cdn-tos.bytecdntp.com/cdn/expire-1-M/byted-player-videos/1.0.0/xgplayer-demo.mp4"
    width="540px"
    controls
    muted
    autoplay
  ></video>

  <script>
    let v = document.getElementsByTagName("video")[0];
    v.playbackRate = "1.5";

    function getLength() {
      let length = Math.floor(v.duration);

      let minute = Math.floor(length / 60);
      let second = length % 60;

      document.getElementsByTagName(
        "p"
      )[0].innerHTML = `视频长度为${minute}分${second}秒`;
    }
  </script>
</body>
```

![video](https://s2.loli.net/2022/01/26/lwsW9KXdOIhrbLD.gif)

除了上面的形式还可以用下面的 source 的形式，这样就可以添加多个不同格式的视频。当浏览器不支持时，可以换一个。

![image-20220126222501539](https://s2.loli.net/2022/01/26/HqOh8E7wL6mzPlB.png)

audio 和 video 类似就不多说了。

属性、事件不多说了，链接奉上。[MDN video](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/video)、[表格版](https://www.runoob.com/tags/ref-av-dom.html)

**缺陷**：

- 不支持直接播放 hls、flv 等视频格式
- 视频资源的请求和加载无法通过代码控制
  - 分段加载(节省流量)
  - 清晰度无缝切换
  - 精确预加载

### 4.2 MSE

媒体源扩展 API(Media Source Extensions)

- 无插件在 web 端播放流媒体
- 支持 hls、flv、mp4 等格式视频
- 可实现视频分段加载、清晰度无缝切换、自适应码率、精确加载

#### 4.2.1 使用

![image-20220126220350669](https://s2.loli.net/2022/01/26/BA2k8ULI6xqh3DF.png)

#### 4.2.2 MSE 播放流程

![image-20220126220416302](https://s2.loli.net/2022/01/26/ayB9zSTvZtGdCm2.png)

### 4.3 播放器播放流程

![image-20220126220523143](https://s2.loli.net/2022/01/26/42flrz8tH3s1oZq.png)

## 5. 流媒体协议

**流媒体**是指将一连串数据压缩后，经过网络分段发送，即时传输以供观看音视频的一种技术。

**流媒体协议**是一种标准化的传递方法，用于将视频分解为多个块，将其发送给视频播放器，播放器重新组合播放。

例子：HLS(HTTP Live Streaming)，是 Apple 公司提出的基于 HTTP 的媒体流传输协议，用于实时音视频流的传输。目前被广泛用于视频点播和直播领域。

![image-20220126221619029](https://s2.loli.net/2022/01/26/q2ZkJcIUAhGHXBY.png)

## 6. 应用场景

![image-20220126221925477](https://s2.loli.net/2022/01/26/pwvlXYhBHb4zQxi.png)
