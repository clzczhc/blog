---
title: WebGL基础笔记
date: 2022-01-24 12:49:28
keywords: WebGL
categories: "前端"
tags:
  - 青训营
  - WebGL
---

# WebGL 基础笔记

个人参加字节跳动的青训营时写的笔记。这部分是月影老师讲的 WebGL 基础。

## 1. 简介

WebGL 代码有两部分：

- 运行在 CPU 上的 JavaScript
- 运行在 GPU 上的 GLSL

**CPU 和 GPU**：

- CPU 适合比较复杂的任务，不适合量大但每个单元比较简单的任务
- GPU 有大量的小运算单元构成，每个运算单元只负责处理**简单**的计算，每个运算单元彼此独立。所有计算可以并行处理。适合量大但每个单元比较简单的任务。

图像的处理适合交给 GPU，因为图像会有很多的像素点需要处理。

## 2. 流程

这部分只能说似懂非懂(任重而道远啊)，先贴一下课上的示例代码，方便之后看。

![image-20220121161752150](https://s2.loli.net/2022/01/24/c4mPWogj9vIVfR6.png)

1. 创建 WebGL 上下文

   ```js
   const canvas = document.querySelector("canvas");
   const gl = canvas.getContext("webgl");
   ```

2. 创建 WebGL 程序(GLSL，顶点着色器、片元着色器)

   **顶点着色器**(Vertex Shader)：

   ![image-20220121163200070](https://s2.loli.net/2022/01/24/tapPx5QJwurCEX4.png)

   **片元着色器**(Fragment Shader)：顶点之间的轮廓中的所有像素都会经过片元着色器处理。(并行处理)

   ![image-20220121163223930](https://s2.loli.net/2022/01/24/r19PV7sUClEGcDY.png)

   ![image-20220121163501575](https://s2.loli.net/2022/01/24/jnvh4zaoqIReFJ5.png)

3. 将数据存入缓冲区

   ![image-20220121163750505](https://s2.loli.net/2022/01/24/3lp5OzrWq9ZIJCA.png)

4. 将缓冲区数据读取到 GPU

   ![image-20220121164109751](https://s2.loli.net/2022/01/24/lDbBEtJQh6Paio7.png)

5. GPU 执行 WebGL 程序，输出结果

   ![image-20220121164208556](https://s2.loli.net/2022/01/24/3p1A2IRkV9BZJPX.png)

完整代码：

```js
// 1. 创建WebGL上下文
const canvas = document.querySelector("canvas");
const gl = canvas.getContext("webgl");

// 2. 创建WebGL程序

// 2.1 顶点着色器
const vertex = `
  attribute vec2 position;

  void main() {
    gl_PointSize = 1.0;
    gl_Position = vec4(position, 1.0, 1.0);
  }
`;

// 2.2 片元着色器
const fragment = `
  precision mediump float;

  void main()
  {
    gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0);	// 类似rgba
  }    
`;

// 2.3 加载、编译、使用着色器
const vertexShader = gl.createShader(gl.VERTEX_SHADER);
gl.shaderSource(vertexShader, vertex);
gl.compileShader(vertexShader);

const fragmentShader = gl.createShader(gl.FRAGMENT_SHADER);
gl.shaderSource(fragmentShader, fragment);
gl.compileShader(fragmentShader);

const program = gl.createProgram();
gl.attachShader(program, vertexShader);
gl.attachShader(program, fragmentShader);
gl.linkProgram(program);
gl.useProgram(program);

const points = new Float32Array([-1, -1, 0, 1, 1, -1]);

// 3. 将数据存入缓存区
const bufferId = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, bufferId);
gl.bufferData(gl.ARRAY_BUFFER, points, gl.STATIC_DRAW);

// 4. 将缓冲区数据读取到GPU
const vPosition = gl.getAttribLocation(program, "position");
gl.vertexAttribPointer(vPosition, 2, gl.FLOAT, false, 0, 0);
gl.enableVertexAttribArray(vPosition);

// 5. GPU执行WebGL程序
gl.clear(gl.COLOR_BUFFER_BIT);
gl.drawArrays(gl.TRIANGLES, 0, points.length / 2); // points中有六个点，实际上只有三个点，所以需要除以2
```

效果：

![image-20220121165503382](https://s2.loli.net/2022/01/24/hj6JoKRS4AQau2t.png)

canvas 2D 版本：

```js
const canvas = document.querySelector("canvas");
const ctx = canvas.getContext("2d");

ctx.beginPath();
ctx.moveTo(250, 0);
ctx.lineTo(500, 500);
ctx.lineTo(0, 500);
ctx.fillStyle = "red";
ctx.fill();
```

效果和上图所示一样。(比原生 WebGL 简单好多)

## 3. 多边形

多边形需要进行三角划分

![image-20220121175100166](https://s2.loli.net/2022/01/24/bl275FqhsRkruVN.png)

[Earcut](https://github.com/mapbox/earcut)

```js
// 1. 创建WebGL上下文
const canvas = document.querySelector("canvas");
const gl = canvas.getContext("webgl");

// 2. 创建WebGL程序

// 2.1 顶点着色器
const vertex = `
  attribute vec2 position;

  void main() {
    gl_PointSize = 1.0;
    gl_Position = vec4(position, 1.0, 1.0);
  }
`;

// 2.2 片元着色器
const fragment = `
  precision mediump float;

  void main()
  {
    gl_FragColor = vec4(0.0, 1.0, 1.0, 1.0);
  }    
`;

// 2.3 加载、编译、使用着色器
const vertexShader = gl.createShader(gl.VERTEX_SHADER);
gl.shaderSource(vertexShader, vertex);
gl.compileShader(vertexShader);

const fragmentShader = gl.createShader(gl.FRAGMENT_SHADER);
gl.shaderSource(fragmentShader, fragment);
gl.compileShader(fragmentShader);

const program = gl.createProgram();
gl.attachShader(program, vertexShader);
gl.attachShader(program, fragmentShader);
gl.linkProgram(program);
gl.useProgram(program);

// 顶点集
const vertices = [
  [-1, -1],
  [-1, 0],
  [0, 0],
  [0, -1],
];

const points = vertices.flat(); // 点集
const triangles = earcut(points); // 三角形集(三角划分后的三角性)

const position = new Float32Array(points);
const cells = new Uint16Array(triangles);

// 3. 将数据存入缓存区，这时候需要分点和三角形来分别存
const pointBuffer = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, pointBuffer);
gl.bufferData(gl.ARRAY_BUFFER, position, gl.STATIC_DRAW);

const cellsBuffer = gl.createBuffer();
gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, cellsBuffer);
gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, cells, gl.STATIC_DRAW);

// 4. 将缓冲区数据读取到GPU
const vPosition = gl.getAttribLocation(program, "position");
gl.vertexAttribPointer(vPosition, 2, gl.FLOAT, false, 0, 0);
gl.enableVertexAttribArray(vPosition);

// 5. GPU执行WebGL程序
gl.clear(gl.COLOR_BUFFER_BIT);
gl.drawElements(gl.TRIANGLES, cells.length, gl.UNSIGNED_SHORT, 0); // 这部分还不是很明白
```

效果：

![image-20220121184608412](https://s2.loli.net/2022/01/24/Chzl9qIcwPTYSHU.png)

## 4. 变换

- 平移
- 旋转
- 缩放

![image-20220121203813648](https://s2.loli.net/2022/01/24/dxmy2MaYjuCgec7.png)

## 5. 3D 标准模型的四个齐次矩阵

- 投影矩阵
- 模型矩阵
- 视图矩阵
- 法向量矩阵

**挖了个大坑**

## 6. 相关链接

- [The Book of Shaders](https://thebookofshaders.com/)
- [着色器分享](https://www.shadertoy.com/)
- [Mesh.js](https://github.com/mesh-js/mesh.js)
- [glsl-doodle](https://doodle.webgl.group/)
- [SpriteJS](http://spritejs.com/)
- [three.js](https://threejs.org/)
