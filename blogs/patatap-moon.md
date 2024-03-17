---
title: patatap 动效 moon 分析
categories: 前端
date: 2024-03-18 00:17:00
tags:
  - 动画
  - patatap
---

# patatap 动效 moon 分析

## 前言

刷到一个很 🐂 的网站[patatap](https://patatap.com/)，好奇它的实现原理，看了下源码，记录一下分析结果。

## 用到的库

two.js：二维绘图
tween.js：补间引擎

核心就是利用 two.js 绘制图形，然后利用 tween.js 加一些补间动画，过渡。

## 准备工作

使用 rollup 进行代码打包操作
rollup.config.js

```js
import resolve from "@rollup/plugin-node-resolve";
import commonjs from "@rollup/plugin-commonjs";
import typescript from "@rollup/plugin-typescript";

export default {
  input: "src/index.ts",
  output: {
    file: "bundle.js",
  },
  plugins: [
    resolve(), // 处理外部依赖
    commonjs(), // 支持基于commonJS的npm模块
    typescript(),
  ],
};
```

## moon 动效

### 代码

common.js（two.js 实例、container 容器等公用内容）

```js
import Two from "two.js";

export const two = new Two({
  type: Two.Types.canvas, // two.js有三种类型：canvas、svg、webgl
  fullscreen: true,
});

export const container: HTMLElement = document.querySelector("#container");

export const center = new Two.Vector(two.width / 2, two.height / 2);
```

index.js

```js
import Two from "two.js";
import TWEEN, { Tween } from "@tweenjs/tween.js";
import { Vector } from "two.js/src/vector";
import { center, container, two } from "./common.js";

two.appendTo(container);

const duration = 1000;

// 最小的一边
let minDimension = Math.min(two.width, two.height);

// 监听resize事件，resize触发的时候，重新设置中心点
two.bind("resize", () => {
  center.x = two.width / 2;
  center.y = two.height / 2;

  minDimension = Math.min(two.width, two.height);
});

function makeMoon() {
  const amount = 64;
  const half = amount / 2;
  const destinations: Vector[] = [];

  // Two.Anchor对象是Two.Vector对象的扩展。
  // Two.Vector一般用于点的位置坐标，而Two.Anchor一般用于表示路径上的锚点
  const points = [...Array(amount).keys()].map(() => {
    destinations.push(new Two.Vector()); // 与Two.Anchor对象建立关联。
    return new Two.Anchor();
  });

  // 创建路径。（此时的路径对象里面的点是空的）
  const moon = two.makePath(points);
  const options = {
    in: 0,
    out: 0,
  };

  moon.fill = "#e34f0c";
  moon.noStroke();

  function resize() {
    // 让绘制的图形移动到center的位置
    moon.translation.copy(center);
  }
  resize();

  let animate_in: Tween<typeof options>;
  let animate_out: Tween<typeof options>;
  function start() {
    moon.visible = true;
    animate_in.start();
  }

  function onComplete() {
    animate_out.start();
  }

  function reset() {
    // 如果有动画，则先停止动画
    if (animate_in) {
      animate_in.stop();
    }
    if (animate_out) {
      animate_out.stop();
    }

    options.in = 0;
    options.out = 0;

    moon.visible = false;
    moon.rotation = Math.random() * Math.PI * 2; // 随机设置转动，避免千篇一律

    const radius = minDimension * 0.33; // 这里猜测是因为乘以0.33的话，最多就两位小数。但是如果是除以3，可能会是无限小数

    moon.vertices.map((v, i) => {
      const pct = i / (amount - 1);
      const theta = pct * Math.PI * 2;

      const x = radius * Math.cos(theta);
      const y = radius * Math.sin(theta);

      // moon.vertices在y轴上，destinations在y轴下
      destinations[i].set(x, y);
      if (i < half) {
        // i < half的时候，y坐标会是正数，所以需要乘以-1，让它到y轴下
        destinations[i].y *= -1;
      }
      v.set(x, Math.abs(y));
    });

    animate_in = new TWEEN.Tween(options)
      .to({ in: 1 }, duration * 0.5)
      .easing(TWEEN.Easing.Sinusoidal.Out)
      .onUpdate(() => {
        for (let i = half; i < amount; i++) {
          // 将 points[i] 的位置从当前位置平滑地过渡到目标位置 destinations[i]
          // 这里会修改路径的坐标
          // 实际的效果就是：半圆往下翻，形成圆
          points[i].lerp(destinations[i], options.in);
        }
      })
      .onComplete(onComplete); // animate_in完成后，开启补间动效animate_out

    animate_out = new TWEEN.Tween(options)
      .to({ out: 1 }, duration * 0.5)
      .easing(TWEEN.Easing.Sinusoidal.Out)
      .onUpdate(() => {
        for (let i = 0; i < half; i++) {
          points[i].lerp(destinations[i], options.out);
        }
      })
      .onComplete(reset);
  }

  document.addEventListener("click", () => {
    reset();
    start();
  });

  two
    .bind("update", () => {
      TWEEN.update();
    })
    .play();
}

makeMoon();
```

### 分析

1. 实例化 two.js，调用 appendTo 跟 dom 元素关联起来
2. 构建点集 points，并且使用 destinations 跟 points 建立关联（用于后续构建补间动效）
3. `two.makePath(points)`。创建路径，此时是空路径。
4. 设置中心点。利用`moon.translation.copy(center)`让绘制的图形移动到 center 的位置
5. 因为中心点有可能会变，实际上在最开始有个绑定`resize`监听时间的逻辑

```js
// 监听resize事件，resize触发的时候，重新设置中心点
two.bind("resize", () => {
  center.x = two.width / 2;
  center.y = two.height / 2;

  minDimension = Math.min(two.width, two.height);
});
```

6. 核心逻辑：
   ![](https://www.clzczh.top/CLZ_img/images/202403172348645.png)

- 6.1 遍历路径的所有点，根据索引 index 顶点数组的百分比来获取它的角度。并利用上图获取 x，y 坐标。

  ```js
  moon.vertices.map((v, i) => {
    const pct = i / (amount - 1);
    const theta = pct * Math.PI * 2;
    const x = radius * Math.cos(theta);
    const y = radius * Math.sin(theta);
  });
  ```

- 6.2 存储坐标，其中需要做一些处理，`moon.vertices`的点在 y 轴上，`destinations`在 y 轴下。这样子后续就直接把`moon.vertices`的点过渡到`destinations`的点就是动效了。
  ```js
  // moon.vertices在y轴上，destinations在y轴下
  destinations[i].set(x, y);
  if (i < half) {
    // i < half的时候，y坐标会是正数，所以需要乘以-1，让它到y轴下
    destinations[i].y *= -1;
  }
  v.set(x, Math.abs(y));
  ```

* 6.3 利用 tween.js 实现补间动效

  ```js
  const options = {
    in: 0,
    out: 0,
  };
  animate_in = new TWEEN.Tween(options)
    .to({ in: 1 }, duration * 0.5)
    .easing(TWEEN.Easing.Sinusoidal.Out)
    .onUpdate(() => {
      for (let i = half; i < amount; i++) {
        // 将 points[i] 的位置从当前位置平滑地过渡到目标位置 destinations[i]
        // 这里会修改路径的坐标
        // 实际的效果就是：半圆往下翻，形成圆
        points[i].lerp(destinations[i], options.in);
      }
    })
    .onComplete(onComplete); // animate_in完成后，开启补间动效animate_out
  ```

  ```js
  animate_out = new TWEEN.Tween(options)
  .to({ out: 1 }, duration * 0.5)
  .easing(TWEEN.Easing.Sinusoidal.Out)
  .onUpdate(() => {
    for (let i = 0; i < half; i++) {
      points[i].lerp(destinations[i], options.out);
    }
  })
  .onComplete(reset);
  }
  ```

  > animate_in 对后一半的点进行处理，这样子前一半在 y 轴上面，后一半在 y 轴下面，就会形成一个圆。而 animate_out 对前一半的点进行处理，让它们过渡到 y 轴下面，这样子会只有重叠成一个 y 轴下面的半圆。动效效果就是圆折成半圆，完成的时候，隐藏路径。

### 注意点

开启 two.js 的动画时，还需要绑定`update`事件，当`update`事件触发时，调用`TWEEN.update()`更新`TWEEN`。

```js
two
  .bind("update", () => {
    TWEEN.update();
  })
  .play();
```

## 效果

![](https://www.clzczh.top/CLZ_img/images/202403180016222.gif)
