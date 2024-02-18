---
title: 简单探析rellax
categories: 前端
date: 2024-02-18 23:08:35
tags:
    - 动画
---

## 简单探析rellax

### 核心

视差公式

```js
(posY - blockTop + screenY) / (blockHeight + screenY)
```

![](https://www.clzczh.top/CLZ_img/images/202401281935673.png)

> 如上图所示，视差公式有两个边界值1和0，当值在0到1之间时，代表元素在视口中（可能只是又一部分）。当元素在视口顶部以上的位置时，值会大于1，在视口底部以下的位置时，值会小于0。而向下滚动时，视差公式的值会从小变大，然后跟根据视差公式的值和速度，给元素添加一个`transform`样式，来实现视差滚动。

## 流程

![](https://www.clzczh.top/CLZ_img/images/202401282229886.png)

## 代码

> 实现一个简单版的Rellax。并且在注释中加入了自己理解看rellax的源码的理解

```js
// SimpleRellax

(function (root, factory) {
  root.SimpleRellax = factory();
})(window, function () {
  const SimpleRellax = function (el, options) {
    // self继承SimpleRellax的属性
    const self = Object.create(SimpleRellax.prototype);
    self.options = {
      speed: -2,
    };

    // 如果有传参数，那么使用传参的选项
    if (options) {
      Object.keys(options).forEach(function (key) {
        self.options[key] = options[key];
      });
    }

    // 简单版。只考虑字符串的情况
    const elements = document.querySelectorAll(el);
    self.elems = elements;

    let screenY;
    let posY = 0;

    /** 设置位置。并且当发生滚动的时候，返回true。否则返回false */
    const setPosition = () => {
      // 旧的位置
      const oldY = posY;

      posY = document.documentElement.scrollTop;

      // 旧的位置不等于新的位置，表示发生了滚动
      if (oldY !== posY) {
        return true;
      }

      return false;
    };

    const updatePosition = (percentageY, speed) => {
      const result = {};

      // 速度乘以视差百分比。乘以100是为了将影响放大，效果更明显
      // 往下滚，percentageY会变大。如果是speed * 100 * percentageY，那valueY的值会变大。
      // 末状态减初状态就会是正数，而速度是正数的情况下，视差元素应该要“走”的比正常元素快。所以需要取一下反。
      // 可以简单的换算一下，验证真假。Y1：初始状态，Y2：末状态。
      // speed * (percentageY1 - percentageY2) = speed * (( 1-percentageY2 ) - (1-percentageY1))
      const valueY = speed * (100 * (1 - percentageY));

      // 取两位小数
      result.y = Math.round(valueY * 100) / 100;

      return result;
    };

    const createBlock = (el) => {
      const dataSpeed = el.getAttribute("data-rellax-speed");

      // 初始状态位置的计算基准为0
      const elPosY = 0;

      const blockTop = elPosY + el.getBoundingClientRect().top;
      const blockHeight = el.clientHeight;

      // 视差公式
      const percentageY =
        (elPosY + screenY - blockTop) / (screenY + blockHeight);

      const speed = dataSpeed ? dataSpeed : self.options.speed;

      const bases = updatePosition(percentageY, speed);

      const style = el.style.cssText;
      let transform = "";

      const searchResult = /transform\s*:/i.exec(style);
      // 如果元素有transform属性的话，保存transform的值
      if (searchResult) {
        const index = searchResult.index;

        const trimmedStyle = style.slice(index);
        const delimiter = trimmedStyle.indexOf(";");

        if (delimiter) {
          // delimiter为true，表示元素的style属性并不只是有transform。这时候只需要截取transform的值
          transform =
            " " + trimmedStyle.slice(11, delimiter).replace(/\s/g, "");
        } else {
          transform = " " + trimmedStyle.slice(11).replace(/\s/g, "");
        }
      }

      return {
        baseY: bases.y,
        top: blockTop,
        height: blockHeight,
        speed: speed,
        style,
        transform,
      };
    };

    let blocks = [];
    const cacheBlocks = () => {
      for (let i = 0; i < self.elems.length; i++) {
        const block = createBlock(self.elems[i]);
        blocks.push(block);
      }
    };

    const animate = () => {
      for (let i = 0; i < self.elems.length; i++) {
        const percentageY =
          (posY + screenY - blocks[i].top) / (screenY + blocks[i].height);

        const position = updatePosition(percentageY, blocks[i].speed);

        const positionY = position.y - blocks[i].baseY;

        const translate =
          `translate3d(0, ${positionY}px, 0)` + blocks[i].transform;
        self.elems[i].style["transform"] = translate;
      }
    };

    let loopId = null;
    const loop =
      window.requestAnimationFrame ||
      window.webkitRequestAnimationFrame ||
      window.mozRequestAnimationFrame ||
      window.msRequestAnimationFrame ||
      window.oRequestAnimationFrame ||
      function (callback) {
        return setTimeout(callback, 1000 / 60);
      };

    const deferredUpdate = () => {
      window.removeEventListener("resize", deferredUpdate);
      window.removeEventListener("orientationchange", deferredUpdate);
      window.removeEventListener("scroll", deferredUpdate);
      window.removeEventListener("touchmove", deferredUpdate);

      loopId = loop(update);
    };

    let pause = true; // 用于添加事件，避免反复添加事件。
    const update = () => {
      if (setPosition() && pause === false) {
        // 在滚动的时候，执行这部分内容
        animate();

        loopId = loop(update);
      } else {
        // 停止滚动的时候，执行这部分内容
        // 通过反复添加、移除事件的方式，可以节省资源。一直绑定一个事件监听器会导致持续持续消耗资源，即使没发生滚动
        window.addEventListener("resize", deferredUpdate);
        window.addEventListener("orientationchange", deferredUpdate);
        window.addEventListener("scroll", deferredUpdate);
        window.addEventListener("touchmove", deferredUpdate);
      }
    };

    const init = () => {
      screenY = window.innerHeight;

      setPosition();

      blocks = [];
      cacheBlocks();

      animate();

      if (pause) {
        pause = false;
        window.addEventListener("resize", init);
        update();
      }
    };

    init();
  };

  return SimpleRellax;
});
```

## 使用

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title></title>
    <style>
        html,
        body {
            color: #223;
            font-size: 21pt;
            margin: 0;
            padding: 0;
        }

        nav {
            font-size: 21pt;
            line-height: 1.5em;
            position: fixed;
            bottom: 0;
            left: 0;
        }

        nav a {
            color: inherit;
        }

        .col {
            text-align: center;
            width: 50%;
            float: left;
            box-sizing: border-box;
        }

        h4 {
            text-align: center;
        }

        .container {
            outline: 1px solid #eed;
            font-size: 42pt;
            padding: 37.5vh 12.5vw;
        }

        .block {
            background: #223;
            color: #eed;
            line-height: 25vh;
            text-align: right;
            padding: 0 42pt;
            position: relative;
        }

        span {
            background: #eed;
            color: #223;
            line-height: 25vh;
            text-align: left;
            padding: 0 42pt;
            display: block;
            width: 50%;
            position: absolute;
            top: 0;
            left: 0;
            box-sizing: border-box;
        }
    </style>
</head>

<body>
    <h4>verticalScroll X / verticalScroll XY</h4>
    <section>
        <div class="col">
            <div class="container">
                <div class="block">#1<span class="rellax" data-rellax-speed="2"
                        style="transform: translate3d(0, 0, 0);">#1</span></div>
            </div>
        </div>
    </section>


    <!-- Scripts -->
    <script src="../simpleRellax.js"></script>
    <script>
        var rellax = new SimpleRellax('.rellax', {
            horizontal: true,
        });

    </script>
</body>

</html>
```
