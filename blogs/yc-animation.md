---
title: 前端动画实现笔记
date: 2022-01-25 16:13:36
keywords: WebGL
categories: "前端"
tags:
  - 青训营
  - 动画
---

# 前端动画实现笔记

参加字节跳动的青训营时个人写的笔记。这部分是蒋翔老师讲的课。

动画：动画是通过快速连续排列彼此差异极小的连续图像来制造运动错觉和变化错觉的过程。

- 快速
- 连续排列
- 彼此差异极小
- 制造错觉

![duck](https://clz.vercel.app/medias/my_comment_bg.jpg)

**动画都需要定义两个基本状态，即起始状态和结束状态，然后填补两者之间的空白，让动画连贯。**

空白的补全方法有两种：

- **补间动画**：传统动画。主画师绘制关键帧，补间动画师补充关键帧。(而在前端中，补间动画师就由浏览器来当，如 keyframe 和 transition)
- **逐帧动画**：每一帧都由主画师绘制。(如由 steps 实现的精灵动画)

常见的前端动画技术：Sprite 动画、CSS 动画、JS 动画、SVG 动画、WebGL 动画

## 1. CSS 动画

CSS animation 是常见的 CSS 动画实现方式

- animation-name：应用的一系列动画。每个名称代表一个由` @keyframes`定义的动画序列
- animation-duration：一个动画周期的时长(默认是 0s，表示无动画)
- animation-timing-function：CSS 动画在每一动画周期中执行的节奏
- animation-delay：动画延时播放
- animation-iteration-count：动画在结束前运行的次数，可以是 1 次，也可以是无限循环
- animation-direction：动画是否反向播放
- animation-play-state：定义一个动画是否运行或暂停

### 1.1 translate(移动)

定义元素的平移变换。

```css
div {
  width: 200px;
  height: 200px;
  background-color: red;
  animation: Translate infinite 4s linear;
}

@keyframes Translate {
  0% {
    transform: translate(0, 0);
  }
  25% {
    transform: translate(0, 200px);
  }
  50% {
    transform: translate(200px, 200px);
  }
  75% {
    transform: translate(200px, 0);
  }
  100% {
    transform: translate(0, 0);
  }
}
```

![动画](https://s2.loli.net/2022/01/25/uHF3YfOLzotbG1l.gif)

### 1.2 scale(缩放)

定义元素的缩小放大

```css
div {
  width: 200px;
  height: 200px;
  background-color: red;
  animation: Scale infinite 2s linear;
  transform-origin: 0 0; /*缩放基准点*/
}

@keyframes Scale {
  from {
    transform: scale(1, 1);
  }
  to {
    transform: scale(0.5, 0.5); /*第一个、第二个参数分别是x轴、y轴缩放的倍数*/
  }
}
```

![动画1](https://s2.loli.net/2022/01/25/NmUHI8zh4fOw1xF.gif)

### 1.3 rotate(旋转)

定义元素的旋转

```css
div {
  width: 200px;
  height: 200px;
  margin: 200px;
  background-color: red;
  animation: Scale infinite 4s linear;
}

@keyframes Scale {
  from {
    transform: rotate(0);
  }
  100% {
    transform: rotate(360deg);
  }
}
```

![动画](https://s2.loli.net/2022/01/25/mHpSAQilKRXtwuc.gif)

### 1.4 skew(倾斜)

定义元素的倾斜

```css
div {
  width: 200px;
  height: 200px;
  margin: 200px;
  background-color: red;
  animation: Translate 1s linear forwards;
}

@keyframes Translate {
  from {
    transform: skew(0, 0);
  }
  to {
    transform: skew(
      0,
      45deg
    ); /* 第一个参数是水平方向的倾斜角度，第二个参数是垂直方向的倾斜角度 */
    /* transform: skew(45deg, 0); */
  }
}
```

![动画](https://s2.loli.net/2022/01/25/8OZCty5rSQKXsNx.gif)

### 1.5 CSS 精灵动画

[CSS steps 实现逐帧动画](https://codepen.io/jiangxiang/pen/GRmarxE)

效果(直接打开可能会看不了，可能要科学上网，蒋翔老师的这张图片好像是放到 github 上的)

![动画](https://s2.loli.net/2022/01/25/9gUBVTw8yD5Outs.gif)

### 1.6 CSS 动画优缺点

优点：简单、高效。不依赖于主线程，采用硬件加速(GPU)。

缺点：不能动态修改或定义动画的内容，不同的动画无法实现同步，多个动画无法堆叠

使用场景：简单的 H5 活动/宣传页

相关库：animation.css、shake.css

### 1.7 CSS 属性

[filter CSS 属性](https://developer.mozilla.org/zh-CN/docs/Web/CSS/filter)

这部分不知道为什么放在了 SVG 部分，不过个人又放到了 CSS 这边。(感觉以后可能能用上)

## 2. SVG 动画

SVG 是基于 XML 的矢量图形描述语言，可以和 CSS、JS 很好的配合。

实现 SVG 动画有三种方式：

- SMIL
- JS
- CSS

### 2.1 line

JS 笔画的原理：stroke-dashoffset、stroke-dasharray 配合使用实现笔画效果。

- 属性 stroke-dasharray 可控制用来描边的点画线的图案。它是一个数列，指定短划线和空白的长度。如果提供奇数个值，则这个值的数列重复一次。如 1,2,3 等同于 1,2,3,1,2,3
- 属性 stroke-dashoffset 属性指定 dash 模式到路径开始的距离

```svg
<line stroke-dasharray="10, 5"
      x1="10" y1="10" x2="100" y2="10" />
<!--  10像素短划线，5像素空白。起点是(10, 10), 终点是(100, 10)  -->
```

![image-20220122184357682](https://s2.loli.net/2022/01/25/MmXsOeSiEj1u9C6.png)

[更多例子](https://codepen.io/jiangxiang/pen/LYzvvxd)

### 2.2 path

**这部分待后续填坑**

![image-20220122184551802](https://s2.loli.net/2022/01/25/ceImrnOgFi9S6XH.png)

[例子](https://codepen.io/jiangxiang/pen/eYWagxq)

### 2.3 演示

<b style="color: red">不是本人写的。属于是分享链接</b>

[文字变形](https://codepen.io/jiangxiang/pen/MWmdjeY)

[写字效果](https://codepen.io/jiangxiang/pen/rNmgjqX)

### 2.4 SVG 优点与缺点

- **优点**：通过矢量元素实现动画，不同的屏幕下都有较好的清晰度。可以实现描字、形变等特殊效果
- **缺点**：使用复杂(个人现阶段属于是一头雾水)

## 3. JS 动画

JS 可以实现很多复杂的动画，还可以操作 canvas 进行绘制。

**JS 动画函数封装**(蒋翔老师讲课用上的)：

```js
function animate({ easing, draw, duration }) {
  let start = performance.now(); // 不使用Date.now()的原因是performance.now()以恒定速度自增，精确到微秒级别，不易被篡改

  return new Promise((resolve) => {
    requestAnimationFrame(function animate(time) {
      // timeFraction goes from 0 to 1
      let timeFraction = (time - start) / duration;
      if (timeFraction > 1) timeFraction = 1;

      // calculate the current animation state
      let progress = easing(timeFraction);

      draw(progress); // draw it

      if (timeFraction < 1) {
        requestAnimationFrame(animate);
        /*
         * 使用RequestAnimationFrame而不是使用setTimeout或setInterval的原因：
         * 该方法允许回调函数在浏览器准备重绘时运行，而且很快
         * 当页面在后台时，也就不会有重绘，所以回调函数也不会运行，所以动画会暂停，不会消耗资源
         */
      } else {
        resolve();
      }
    });
  });
}
```

参数：

- **easing**：缓动函数。决定执行进度在时间增加的过程中的变化，可以是线性的，也可以是非线性的

  ```js
  easing(timeFraction) {
    return timeFraction * 100;
  },
  ```

- **draw**：绘制函数。相当于画笔，会一直反复被调用。入参是当前执行的进度 progress，是一个介于 0 到 1 之间的数字

  ```js
  const draw = (progress) => {
    ball.style.transform = `translate(${progress}px, 0)`;
  };
  ```

- **duration**：持续时间

### 3.1 匀速运动

```js
const draw = (progress) => {
  ball.style.transform = `translate(${progress * 100}px, 0)`; // 一秒走100像素
};

animate({
  easing(timeFraction) {
    return timeFraction;
  },
  draw,
  duration: 1000,
});
```

[更多](https://codepen.io/jiangxiang/pen/rNmgVKK)

这部分还不是很明白，和数学挂上钩了(累)。

## 4. 优化

性能角度：页面渲染的一般过程：JS -> CSS -> 计算样式 -> 布局 -> 绘制->渲染层合并。其中，布局(重排)和绘制(重绘)是最耗时的两部分，所以应尽可能减少这两部分。

![image-20220122215918667](https://s2.loli.net/2022/01/25/TZVLP8nfoyhHmb2.png)
