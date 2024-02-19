---
title: 简单探析canvas-nest
categories: 前端
date: 2024-02-19 23:08:35
tags:
  - 动画
---

# 简单探析 canvas-nest

## 流程

![](https://www.clzczh.top/CLZ_img/images/202402061529746.png)

绘制流程：

> 遍历每一个非鼠标点，绘制点。
>
> 在遍历到每个点的时候，并且从下一个点（包括鼠标点）开始遍历，判断其他点有没有在当前点的吸附区域内。
>
> 如果在，需要画线。当前点之前的不需要遍历是因为，当前点是否在前面的点的吸附区域，在之前已经判断过了。

##

## 代码

```js
(() => {
  const createCanvas = (el) => {
    const canvas = document.createElement("canvas");
    canvas.width = el.clientWidth;
    canvas.height = el.clientHeight;

    el.appendChild(canvas);

    return canvas;
  };

  class CanvasNest {
    constructor(el, config) {
      this.el = el;

      this.config = {
        color: "0,0,0",
        pointColor: "0,0,0",
        count: 100,
        opacity: 0.2,
        ...config,
      };

      this.canvas = createCanvas(el);
      this.context = this.canvas.getContext("2d");

      this.points = this.randomPoints();
      this.current = {
        x: null, // 初始鼠标坐标设置为null，避免影响到(0, 0)位置的点
        y: null,
        zoom: 150, // 鼠标的吸附距离比普通点的大。
      };
      this.pointsContainMouse = this.points.concat([this.current]);

      this.bindEvent();

      window.requestAnimationFrame(() => this.paint());
    }

    randomPoints() {
      // 直接new Array(n)生成的empty数组不会执行map里面的内容
      return new Array(this.config.count).fill(0).map(() => ({
        x: Math.random() * this.canvas.width,
        y: Math.random() * this.canvas.width,
        vx: 2 * Math.random() - 1, // 速度范围为-1~1，随机
        vy: 2 * Math.random() - 1,
        zoom: 100, // 吸附距离
      }));
    }

    bindEvent() {
      window.addEventListener("resize", () => {
        this.canvas.width = this.el.clientWidth;
        this.canvas.height = this.el.clientHeight;
      });

      document.addEventListener("mousemove", (e) => {
        this.current.x = e.clientX;
        this.current.y = e.clientY;
      });

      document.addEventListener("mouseout", (e) => {
        this.current.x = null;
        this.current.y = null;
      });
    }

    paint() {
      const width = this.canvas.width;
      const height = this.canvas.height;
      this.context.clearRect(0, 0, width, height);

      this.points.forEach((point, index) => {
        point.x += point.vx;
        point.y += point.vy;

        // 如果碰到边界就反弹
        point.vx *= point.x >= width || point.x < 0 ? -1 : 1;
        point.vy *= point.y >= height || point.y < 0 ? -1 : 1;

        this.context.fillStyle = `rgba(${this.config.pointColor})`;
        this.context.fillRect(point.x - 0.5, point.y - 0.5, 1, 1); // 绘制宽高为1的矩形，坐标为矩形中心

        for (let i = index + 1; i < this.pointsContainMouse.length; i++) {
          const p = this.pointsContainMouse[i];

          const dx = point.x - p.x;
          const dy = point.y - p.y;

          const distance = Math.sqrt(dx * dx + dy * dy);

          if (distance < point.zoom) {
            if (p === this.current && distance >= 0.8 * point.zoom) {
              // 如果当前点是鼠标，并且距离大于等于吸附距离的80%时，
              // 鼠标靠近的时候加速。这样子会形成一个线圆。因为如果点是向着圈外的方向的话，最终会和这个加速抵消；
              // 而如果是向着原因的方向的话，最终会变成向着另一边的圈外，也会和这个靠近加速抵消。所以会形成一个线圆

              // 抵消具体效果：当距离变小时，靠近会越变越慢，当小于临界值时，又会被点原本的速度带着像圈外走，就像拔河一样一直此起彼伏。最后会稳定在一个圈中。

              point.x -= 0.03 * dx;
              point.y -= 0.03 * dy;
            }

            // 获取距离占吸附距离的比例。根据该比例来设置线的粗细和透明度
            const percentage = (point.zoom - distance) / point.zoom;

            this.context.lineWidth = percentage;
            this.context.fillStyle = `rgba(${this.config.color}, ${percentage})`;

            this.context.beginPath();
            this.context.strokeStyle = `rgba(${this.config.color})`;
            this.context.moveTo(point.x, point.y);
            this.context.lineTo(p.x, p.y);
            this.context.stroke();
          }
        }
      });

      window.requestAnimationFrame(() => this.paint());
    }
  }

  new CanvasNest(document.body);
})();
```

## 使用

```html

```
