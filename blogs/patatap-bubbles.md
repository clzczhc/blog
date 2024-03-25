---
title: patatap 动效 bubbles 分析
categories: 前端
date: 2024-03-25 15:30:00
tags:
  - 动画
  - patatap
---

# patatap 动效 bubbles 分析

> 跟之前分析的 moon 类似，看着其实更像是为了实现 two.js 而写的 demo，但是看源代码。发现这种动效更多的是要有好思路，有好思路就能化繁为简。

## 前言

简单封装了一下，这样后续再添加动画，就不需要改一些公共部分之类。方式采用 patatap 的，相当于简陋版，函数 register 注册动画，存储在一个 map 对象中。

## 动效代码

代码

```ts
import * as TWEEN from "@tweenjs/tween.js";
import Two from "two.js";
import animations, { register } from "./index";
import { two, center } from "../common";
import { Circle } from "two.js/src/shapes/circle";

let direction = true;
let animate_ins = [],
  animate_outs = [];

const TWO_PI = Math.PI * 2;
const duration = 1000;
const amount = 24;
const last = amount - 1;
const circles = [...Array(amount).keys()].map((i) => {
  const circle: Circle & {
    theta?: number;
    destination?: number;
  } = new Two.Circle();
  circle.theta = 0;
  circle.destination = 0;
  return circle;
});

const group = two.makeGroup(circles);
group.noStroke().fill = "#e34f0c";

function start() {
  animate_ins[0].start();
}

function resize() {
  group.translation.copy(center);
}

function reset() {
  if (animate_ins.length > 0) {
    animate_ins.forEach(stop);
    animate_ins.length = 0;
  }
  if (animate_outs.length > 0) {
    animate_outs.forEach(stop);
    animate_outs.length = 0;
  }

  direction = Math.random() > 0.5;
  group.rotation = Math.random() * TWO_PI;

  const radius = animations.minDimension / 3;
  const bubbleRadius = animations.minDimension / 90;

  for (let i = 0; i < circles.length; i++) {
    let pct = i / amount;
    let npt = (i + 1) / amount;

    const circle = circles[i];
    circle.visible = false;
    circle.radius = bubbleRadius;

    circle.destination = TWO_PI * pct;
    circle.theta = 0;
    circle.translation.set(radius, 0);

    const ain = new TWEEN.Tween(circle)
      .to({ theta: circle.destination }, (0.2 * duration) / (i + 1))
      .onStart(() => (circle.visible = true))
      .onUpdate(() => {
        const theta = circle.theta * (direction ? 1 : -1);
        const x = radius * Math.cos(theta);
        const y = radius * Math.sin(theta);

        circle.translation.set(x, y);
      })
      .onComplete(() => {
        if (i >= last) {
          animate_outs[0].start();
          return;
        }

        const next = circles[i + 1];
        const tween = animate_ins[i + 1];
        next.theta = circle.theta;
        next.translation.copy(circle.translation);
        tween.start();
      });

    animate_ins.push(ain);

    const destination = Math.min(npt * TWO_PI, TWO_PI);

    const aout = new TWEEN.Tween(circle)
      .to(
        {
          theta: destination,
        },
        (0.2 * duration) / (amount - (i + 1))
      )
      .onUpdate(() => {
        const theta = circle.theta * (direction ? 1 : -1);
        const x = radius * Math.cos(theta);
        const y = radius * Math.sin(theta);

        circle.translation.set(x, y);
      })
      .onComplete(() => {
        circle.visible = false;

        if (i >= last - 1) {
          reset();
        } else {
          animate_outs[i + 1].start();
        }
      });

    animate_outs.push(aout);
  }
}

function stop(tween) {
  tween.stop();
}

resize();
reset();

const animation = {
  start: start,
  clear: reset,
  resize: resize,
  name: "bubbles",
};

register(animation.name, animation);
```

因为该动效与 moon 动效原理类似，就不过多赘述了。简单讲一下不同点。

1. 利用 Two.Circle 构建指定数量的圆，然后通过`two.makeGroup(circles)`让它们成为一个整体，并给 circle 添加自定义属性`theta`、`destination`用于后续动画
2. 动画为两个数组，animate_ins、animate_outs，因为每个点都会有动画
3. 根据索引 index 顶点数组的百分比来获取角度，然后设置成 tween.js 过渡的终点状态，并利用(0.2 \* duration) / (i + 1)的方式，让动效的速度越来越快
4. onUpdate 事件中，根据实时的角度，来计算并设置圆的 x、y 坐标
5. 当动画执行完的时候，如果不是最后一个点，那么还需要利用`next.translation.copy(circle.translation)`来把动画起始点移动到当前点，不然每次动效都是从第一个点开始。然后开始下一个点的动效。结束动效类似

## 结果

![](https://www.clzczh.top/CLZ_img/images/202403251550533.gif)
