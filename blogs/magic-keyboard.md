---
title: 使用物理引擎matterjs实现键盘特效动画
categories: 前端
date: 2023-09-11 13:42:08
tags:
    - 动画
---

# 使用物理引擎matterjs实现键盘特效动画

## 前言

偶然间看到一个网站[magickeyboard](%5Bhttp://magickeyboard.io/%5D(http://magickeyboard.io/))，觉得这个动画很炫酷。就收藏了一下，在稍微学习了一下matterjs后，打算跟着源码学习，弄懂并且自己实现一个。

## 准备

先安装`matter-js`和`webpack`，并且稍微配置一下`webpack`。

```js
const path = require("path");

module.exports = {
  entry: path.join(__dirname, "src", "index.js"),
  mode: "development",
  output: {
    filename: "bundle.js",
    path: path.join(__dirname),
  },
  devServer: {
    hot: true,
    static: {
      directory: path.join(__dirname),
    },
  }
};
```

## Matterjs初体验

根据Matterjs官网的demo，可以知道Matterjs的使用主要分为几个步骤：

> 1. 创建引擎
> 
> 2. 创建渲染器
> 
> 3. 创建物体
> 
> 4. 将物体添加到引擎的`world`中
> 
> 5. 执行渲染器
> 
> 6. 创建执行器runner，并运行引擎（如果没有这一步，则没有办法触发物理动画）

index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <style>
    html, body {
      padding: 0;
      margin: 0;
      overflow: hidden;
    }
  </style>
</head>
<body>
  <div class="content"></div>
  <script src="./bundle.js"></script>
</body>
</html>
```

```js
import Matter from 'matter-js';

// module aliases
const Engine = Matter.Engine;
const Render = Matter.Render;
const Runner = Matter.Runner;
const Bodies = Matter.Bodies;
const Composite = Matter.Composite;

const WIDTH = window.innerWidth;
const HEIGHT = window.innerHeight;

// 1. 创建引擎
const engine = Engine.create();

// 2. 创建渲染器
const render = Render.create({
  element: document.querySelector('.content'),
  engine: engine,
  options: {
    width: WIDTH,
    height: HEIGHT,
    background: '#222'
  }
});

// 3. 创建物体
const boxA = Bodies.rectangle(600, 200, 200, 200);
const boxB = Bodies.rectangle(800, 300, 80, 80);
const ground = Bodies.rectangle(700, 400, 800, 40, { isStatic: true });

// 4. 将物体添加到引擎的`world`中
Composite.add(engine.world, [boxA, boxB, ground]);

// 5. 执行渲染器
Render.run(render);

// 6. 创建执行器runner，并运行引擎
const runner = Runner.create();
Runner.run(runner, engine);
```

## 添加键盘事件，每次点击生成一个物体

```js
document.body.addEventListener('keydown', (e) => {
  const ball = Bodies.circle(400, 200, 50, {
    restitution: 0.9,  // 设置弹力
    friction: 0.001    // 设置摩檫力，默认为0.5
  });

  Composite.add(engine.world, ball);
})
```

![](https://www.clzczh.top/CLZ_img/images/202309082212924.gif)

## 添加力

```js
document.body.addEventListener('keydown', (e) => {
  const ball = Bodies.circle(400, 200, 50, {
    restitution: 0.9,
    friction: 0.001    // 设置摩檫力，默认为0.5
  });

  const vector = {
    x: (Date.now() % 10) * 0.004  - 0.02,
    y: (-1 * HEIGHT / 3200)
  };

  Matter.Body.applyForce(ball, ball.position, vector);

  Composite.add(engine.world, ball);
})
```

> 上面的`vector`就是力。y轴方向上，力会根据窗口高度变化的一个，而x轴方向上，力会随着时间的变化而发生变化，力的区间为[-0.02, 0.02]，将它分成10部分，根据时间戳来获取x轴方向上的力。`(Date.now() % 10) * 0.004 - 0.02`

下面的演示用的里的区间是[-0.2, 0.2]，`(Date.now() % 10) * 0.04  - 0.2`（为了看的效果明显点）

![](https://www.clzczh.top/CLZ_img/images/202309082226517.gif)

## 添加倾斜边界

添加倾斜边界，让弹出来的球能够慢慢消失。

```js
const OFFSET = 1;  // 使用的是Matterjs 0.10.0。如果是最新的，可能要设置成10
const boundaries = generateBoundaries();
Composite.add(engine.world, boundaries);

function generateBoundaries() {
  return [
    Bodies.rectangle(WIDTH / 4, HEIGHT - 400, WIDTH / 2, OFFSET, {
      angle: -0.1,
      isStatic: true,
      friction: 0.001    // 设置摩檫力，默认为0.5
      render: {
        fillStyle: '#fff'
      }
    }),

    Bodies.rectangle((WIDTH / 4) * 3, HEIGHT - 400, WIDTH / 2, OFFSET, {
      angle: 0.1,
      isStatic: true,
      friction: 0.001    // 设置摩檫力，默认为0.5
      render: {
        fillStyle: '#fff'
      }
    }),
  ]
}
```

![](https://www.clzczh.top/CLZ_img/images/202309092315104.gif)

## 添加兜底边界

当我们添加倾斜边界时，球会越变越多，包括消失的球也没有进行处理。所以可以添加一个兜底边界，后面再添加碰撞事件，清除接触到兜底边界的小球。

> 下面的例子会先将倾斜边界变短，查看实际效果

```js
function generateBoundaries() {
  return [
    Bodies.rectangle(WIDTH / 4, HEIGHT - 400, WIDTH / 4, OFFSET, {
      angle: -0.1,
      isStatic: true,
      friction: 0.001    // 设置摩檫力，默认为0.5
      render: {
        fillStyle: '#fff'
      }
    }),

    Bodies.rectangle((WIDTH / 4) * 3, HEIGHT - 400, WIDTH / 4, OFFSET, {
      angle: 0.1,
      isStatic: true,
      friction: 0.001    // 设置摩檫力，默认为0.5
      render: {
        fillStyle: '#fff'
      }
    }),

    Bodies.rectangle(WIDTH / 2, HEIGHT - 200, WIDTH, OFFSET, {
      isStatic: true,
      friction: 0.001    // 设置摩檫力，默认为0.5
      render: {
        fillStyle: '#fff'
      }
    })
  ]
}
```

![](https://www.clzczh.top/CLZ_img/images/202309092321934.png)

添加一个`platform`变量用来存储兜底边界，便于实现碰撞检测。

```js
const OFFSET = 1;  // 使用的是Matterjs 0.10.0。如果是最新的，可能要设置成10
const boundaries = generateBoundaries();
Composite.add(engine.world, boundaries);

let platform = boundaries[2];
Matter.Events.on(engine, 'collisionStart', (e) => {
  e.pairs.forEach(function (pair) {  // e.pairs表示发生碰撞的两个物体
    const bodyA = pair.bodyA;
    const bodyB = pair.bodyB;

    if (bodyA === platform) {   // bodyA是兜底边界，则将bodyB从engine的世界中移除。
      Composite.remove(engine.world, [bodyB]);
    }

    if (bodyB === platform) {
      Composite.remove(engine.world, [bodyA]);
    }
  })
});
```

![](https://www.clzczh.top/CLZ_img/images/202309092333150.gif)

## 位置设定

上面的实现会导致不论点击什么键，小球出现的位置都是固定的，所以进行一个位置设定的操作，来实现点击键盘后，位置和键盘上的位置差不多一样。

1. 首先，需要构建一个二维数组，元素的值则是每一行键盘对应的顺序（功能键为`null`）
   
   ```js
   const KEYS = [
   // 字母键和符号键
   ['`', '1', '2', '3', '4', '5', '6', '7', '8', '9', '0', '-', '=', null],
   [null, 'Q', 'W', 'E', 'R', 'T', 'Y', 'U', 'I', 'O', 'P', '[', ']', '\\'],
   [null, 'A', 'S', 'D', 'F', 'G', 'H', 'J', 'K', 'L', ';', '\'', null],
   [null, null, 'Z', 'X', 'C', 'V', 'B', 'N', 'M', ',', '.', '/', null, null],
   
   // 数字键
   [null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, 'num-/', 'num-*', 'num--'],
   [null, null, null, null, null, null, null, null, null, null, null, null, null, null, 'num-7', 'num-8', 'num-9', 'num-+'],
   [null, null, null, null, null, null, null, null, null, null, null, null, null, null, 'num-4', 'num-5', 'num-6', null],
   [null, null, null, null, null, null, null, null, null, null, null, null, null, null, 'num-1', 'num-2', 'num-3', null],
   [null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, 'num-0', null, 'num-.', null]
   ]; 
   ```

2. 遍历每一个键，设定好每个键的位置。
   
   ```js
   const positions = {};
   
   // 生成键盘字母对应的位置
   generatePositions();
   
   function generatePositions() {
   KEYS.forEach((row) => {
     row.forEach((letter, i) => {
       if (!letter) {
         return; // 功能键，忽略
       }
   
       positions[letter] = ((i / row.length) + (0.5 / row.length)) * WIDTH;
     })
   })
   }
   ```
   
   > `((i / row.length) + (0.5 / row.length)) * WIDTH`：`i`是点在行中的索引，`row.length`是该行中点的总数，`(i / row.length) * WIDTH`的值就是该点应该在的位置，但是这样子得到的将不会是对应单元格的中心位置，而是左侧边缘的位置，所以还应该加半个单元格的宽度，即`(0.5 / row.length) * WIDTH`。
   > 
   > ![](https://www.clzczh.top/CLZ_img/images/202306232015367.png)

3. 使用vkey库，将数字键变为`<num-0>`的形式，用来与一般的区分，并且会将所有的英文变成大写。
   
   ```js
   document.body.addEventListener('keydown', (e) => {
   let key = vkey[e.keyCode];
   
   if (key == null) {
     return;
   }
   
   key = key.replace(/</g, '').replace(/>/g, '');  // 将`<num-0>`形式变为`num-0`
   
   if (key in positions) {
     addLetter(key, positions[key], HEIGHT - 50);
   }
   })
   
   function addLetter(key, x, y) {
   const ball = Bodies.circle(x, y, 30, {
     restitution: 0.9
   });
   
   const vector = {
     x: (Date.now() % 10) * 0.004 - 0.02,
     y: (-1 * HEIGHT / 3600)
   };
   
   Matter.Body.applyForce(ball, ball.position, vector);
   
   Composite.add(engine.world, [ball]);
   }
   ```
   
   ![](https://www.clzczh.top/CLZ_img/images/202309102241072.gif)

## 小球换成图片

通过`render.sprite.texture`来设置图片。

```js
const ball = Bodies.circle(x, y, 30, {
  restitution: 0.9,
  friction: 0.001    // 设置摩檫力，默认为0.5
  render: {
    sprite: {
      texture: getImagePath(key)
    }
  }
}); 

function getImagePath(key) {
  if (key.indexOf('num-') === 0) {
    key = key.substring(4);
  }

  if (key === '*') key = 'star';
  if (key === '+') key = 'plus';
  if (key === '.') key = 'dot';
  if (key === '/') key = 'slash';
  if (key === '\\') key = 'backslash';

  return './img/' + key + '.png';
}
```

\color{red}{还需要修改一下创建渲染器时的设置}

```js
const render = Render.create({
  element: document.querySelector('.content'),
  engine: engine,
  options: {
    width: WIDTH,
    height: HEIGHT,
    background: '#222',
    wireframes: false,  // 关闭线框
  }
});
```

![](https://www.clzczh.top/CLZ_img/images/202309102254432.gif)

## 变化边界长度，并隐藏

最下面的边界长度为WIDTH * 4，这样子为了避免内存泄漏，把一些球给移除。

```js
function generateBoundaries() {
  return [
    Bodies.rectangle(WIDTH / 4, HEIGHT + 50, WIDTH / 2, OFFSET, {
      angle: -0.1,
      isStatic: true,
      friction: 0.001    // 设置摩檫力，默认为0.5
      render: {
        fillStyle: '#fff',
        visible: false
      }
    }),

    Bodies.rectangle((WIDTH / 4) * 3, HEIGHT + 50, WIDTH / 2, OFFSET, {
      angle: 0.1,
      isStatic: true,
      friction: 0.001    // 设置摩檫力，默认为0.5
      render: {
        fillStyle: '#fff',
        visible: false
      }
    }),

    Bodies.rectangle(WIDTH / 2, HEIGHT + 400, WIDTH * 4, OFFSET, {
      isStatic: true,
      friction: 0.001    // 设置摩檫力，默认为0.5
      render: {
        fillStyle: '#fff',
        visible: false
      }
    })
  ]
}
```

## 抽离初始化步骤，并在窗口大小变化时重新初始化

> 如果在串口大小变化时，不重新初始化，就会导致一些字符并不会在窗口中看得见。

```js
let WIDTH;
let HEIGHT;

let engine = null;
let render = null;

let boundaries = null;
let platform = null;

const KEYS = [
  // 字母键、符号键
  ['`', '1', '2', '3', '4', '5', '6', '7', '8', '9', '0', '-', '=', null],
  [null, 'Q', 'W', 'E', 'R', 'T', 'Y', 'U', 'I', 'O', 'P', '[', ']', '\\'],
  [null, 'A', 'S', 'D', 'F', 'G', 'H', 'J', 'K', 'L', ';', '\'', null],
  [null, null, 'Z', 'X', 'C', 'V', 'B', 'N', 'M', ',', '.', '/', null, null],

  // 数字键
  [null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, 'num-/', 'num-*', 'num--'],
  [null, null, null, null, null, null, null, null, null, null, null, null, null, null, 'num-7', 'num-8', 'num-9', 'num-+'],
  [null, null, null, null, null, null, null, null, null, null, null, null, null, null, 'num-4', 'num-5', 'num-6', null],
  [null, null, null, null, null, null, null, null, null, null, null, null, null, null, 'num-1', 'num-2', 'num-3', null],
  [null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, 'num-0', null, 'num-.', null]
];
const positions = {};

function init() {
  WIDTH = window.innerWidth;
  HEIGHT = window.innerHeight;

  if (!engine) {
    engine = Engine.create();
  }

  // 生成键盘字母对应的位置
  generatePositions();

  // 如果渲染器不存在，则创建
  if (!render) {
    render = Render.create({
      element: document.querySelector('.content'),
      engine: engine,
      options: {
        width: WIDTH,
        height: HEIGHT,
        background: '#222',
        wireframes: false
      }
    });
  } else {
    // 如果渲染器已存在，则获取渲染器的画布对象，并重新设置宽度和高度
    const canvas = render.canvas;

    // 设置画布的宽度和高度
    canvas.width = WIDTH;
    canvas.height = HEIGHT;
  }

  render.options.width = WIDTH;
  render.options.height = HEIGHT;

  // 如果有边界，那就移除
  if (boundaries) {
    Composite.remove(engine.world, boundaries)
  }

  boundaries = generateBoundaries();
  Composite.add(engine.world, boundaries);

  platform = boundaries[2];
}

init();
window.addEventListener('resize', init);
```

## 添加音效

```html
<body>
  <div class="content"></div>
  <audio id="type" src="./type.mp3"></audio>
  <script src="./bundle.js"></script>
</body>
```

```js
function playSound(name) {
  const sound = document.getElementById(name);
  sound.play();
}

function addLetter(key, x, y) {
  playSound('type'); 
  ...
}
```

## 预加载图片

> 当实际部署上线后，会发现，一开始点击键盘时，是没有反应的。因为当我们点击时，才去请求对应图片。所以可以做一个预加载的操作，优化一下。

预加载就可以通过新建`Image`，并设置`src`值来实现

```js
function preloadImg (url) {
  const img = new window.Image()
  img.src = url
}
```

当遍历设置字符位置时，就能够进行一个预加载操作。

```js
function generatePositions() {
  KEYS.forEach((row) => {
    row.forEach((letter, i) => {
      if (!letter) {
        return; // 功能键，忽略
      }

      positions[letter] = ((i / row.length) + (0.5 / row.length)) * WIDTH;

       // 预加载图片
       preloadImg(getImagePath(letter));
    })
  })
}
```

![](https://www.clzczh.top/CLZ_img/images/202309110028601.gif)

## 添加下雨模式

通过使用`lastKeys`来记录点击过的字符，当最后点击的字符为`rain`时（不区分大小写），开启一个下雨模式。而下雨模式也很简单，只需要重新设置力`vector`，和球的坐标即可。

```js
function secretWords(key) {
  lastKeys = lastKeys.slice(-3) + key;

  if (lastKeys === 'RAIN') {
    rainMode = !rainMode;

    if (rainMode) {
      playSound('rain');
    }
  }
}
```

```js
function addLetter(key, x, y) {
  playSound('type');

  const ball = Bodies.circle(x, y, 30, {
    restitution: 0.9,
    friction: 0.001,
    render: {
      sprite: {
        texture: getImagePath(key)
      }
    }
  });

  let vector = {
    x: (Date.now() % 10) * 0.004 - 0.02,
    y: (-1 * HEIGHT / 3600)
  }; 

  // 新增。下雨模式重新设置坐标，以及力vector
  if (rainMode) {
    vector = {
      x: 0,
      y: 0
    };

    Matter.Body.setPosition(ball, {
      x: ball.position.x,
      y: -30
    })
  }

  secretWords(key);
  // 新增。

  Matter.Body.applyForce(ball, ball.position, vector);

  Composite.add(engine.world, [ball]);
}
```

![](https://www.clzczh.top/CLZ_img/images/202309110040817.gif)



## 完整代码

```html

```

```js
import Matter from 'matter-js';
import vkey from 'vkey';

// module aliases
const Engine = Matter.Engine;
const Render = Matter.Render;
const Runner = Matter.Runner;
const Bodies = Matter.Bodies;
const Composite = Matter.Composite;

const OFFSET = 1;  // 使用的是Matterjs 0.10.0。如果是最新的，可能要设置成10

let WIDTH;
let HEIGHT;

let engine = null;
let render = null;

let boundaries = null;
let platform = null;

const KEYS = [
  // 字母键、符号键
  ['`', '1', '2', '3', '4', '5', '6', '7', '8', '9', '0', '-', '=', null],
  [null, 'Q', 'W', 'E', 'R', 'T', 'Y', 'U', 'I', 'O', 'P', '[', ']', '\\'],
  [null, 'A', 'S', 'D', 'F', 'G', 'H', 'J', 'K', 'L', ';', '\'', null],
  [null, null, 'Z', 'X', 'C', 'V', 'B', 'N', 'M', ',', '.', '/', null, null],

  // 数字键
  [null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, 'num-/', 'num-*', 'num--'],
  [null, null, null, null, null, null, null, null, null, null, null, null, null, null, 'num-7', 'num-8', 'num-9', 'num-+'],
  [null, null, null, null, null, null, null, null, null, null, null, null, null, null, 'num-4', 'num-5', 'num-6', null],
  [null, null, null, null, null, null, null, null, null, null, null, null, null, null, 'num-1', 'num-2', 'num-3', null],
  [null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, 'num-0', null, 'num-.', null]
];
const positions = {};

function init() {
  WIDTH = window.innerWidth;
  HEIGHT = window.innerHeight;

  if (!engine) {
    engine = Engine.create();
  }

  // 生成键盘字母对应的位置
  generatePositions();

  // 如果渲染器不存在，则创建
  if (!render) {
    render = Render.create({
      element: document.querySelector('.content'),
      engine: engine,
      options: {
        width: WIDTH,
        height: HEIGHT,
        background: '#222',
        wireframes: false
      }
    });
  } else {
    // 如果渲染器已存在，则获取渲染器的画布对象，并重新设置宽度和高度
    const canvas = render.canvas;

    // 设置画布的宽度和高度
    canvas.width = WIDTH;
    canvas.height = HEIGHT;
  }

  render.options.width = WIDTH;
  render.options.height = HEIGHT;

  // 如果有边界，那就移除
  if (boundaries) {
    Composite.remove(engine.world, boundaries)
  }

  boundaries = generateBoundaries();
  Composite.add(engine.world, boundaries);

  platform = boundaries[2];
}

init();
window.addEventListener('resize', init);

Composite.add(engine.world, boundaries);

// 5. 执行渲染器
Render.run(render);

// 6. 创建执行器runner，并运行引擎
const runner = Runner.create();
Runner.run(runner, engine);


Matter.Events.on(engine, 'collisionStart', (e) => {
  e.pairs.forEach(function (pair) {  // e.pairs表示发生碰撞的两个物体
    const bodyA = pair.bodyA;
    const bodyB = pair.bodyB;

    if (bodyA === platform) {   // bodyA是兜底边界，则将bodyB从engine的世界中移除。
      Composite.remove(engine.world, [bodyB]);
    }

    if (bodyB === platform) {
      Composite.remove(engine.world, [bodyA]);
    }
  })
});

document.body.addEventListener('keydown', (e) => {
  let key = vkey[e.keyCode];

  if (key == null) {
    return;
  }

  key = key.replace(/</g, '').replace(/>/g, '');

  if (key in positions) {
    addLetter(key, positions[key], HEIGHT - 50);
  }
})

function generatePositions() {
  KEYS.forEach((row) => {
    row.forEach((letter, i) => {
      if (!letter) {
        return; // 功能键，忽略
      }

      positions[letter] = ((i / row.length) + (0.5 / row.length)) * WIDTH;
    
       // 预加载图片
       preloadImg(getImagePath(letter));
    })
  })
}

let rainMode = false;
function addLetter(key, x, y) {
  playSound('type');

  const ball = Bodies.circle(x, y, 30, {
    restitution: 0.9,
    friction: 0.001,
    render: {
      sprite: {
        texture: getImagePath(key)
      }
    }
  });

  let vector = {
    x: (Date.now() % 10) * 0.004 - 0.02,
    y: (-1 * HEIGHT / 3600)
  }; 

  // 新增。下雨模式重新设置坐标，以及力vector
  if (rainMode) {
    vector = {
      x: 0,
      y: 0
    };

    Matter.Body.setPosition(ball, {
      x: ball.position.x,
      y: -30
    })
  }

  secretWords(key);
  // 新增。

  Matter.Body.applyForce(ball, ball.position, vector);

  Composite.add(engine.world, [ball]);
}

function getImagePath(key) {
  if (key.indexOf('num-') === 0) {
    key = key.substring(4);
  }

  if (key === '*') key = 'star';
  if (key === '+') key = 'plus';
  if (key === '.') key = 'dot';
  if (key === '/') key = 'slash';
  if (key === '\\') key = 'backslash';

  return './img/' + key + '.png';
}

function generateBoundaries() {
  return [
    Bodies.rectangle(WIDTH / 4, HEIGHT + 50, WIDTH / 2, OFFSET, {
      angle: -0.1,
      isStatic: true,
      render: {
        fillStyle: '#fff',
        visible: false
      }
    }),

    Bodies.rectangle((WIDTH / 4) * 3, HEIGHT + 50, WIDTH / 2, OFFSET, {
      angle: 0.1,
      isStatic: true,
      render: {
        fillStyle: '#fff',
        visible: false
      }
    }),

    Bodies.rectangle(WIDTH / 2, HEIGHT + 400, WIDTH * 4, OFFSET, {
      isStatic: true,
      render: {
        fillStyle: '#fff',
        visible: false
      }
    })
  ]
}

function playSound(name) {
  const sound = document.getElementById(name);
  sound.play();
}

function preloadImg (url) {
  const img = new window.Image()
  img.src = url
}

let lastKeys = '';
function secretWords(key) {
  lastKeys = lastKeys.slice(-3) + key;

  if (lastKeys === 'RAIN') {
    rainMode = !rainMode;

    if (rainMode) {
      playSound('rain');
    }
  }
}
```
