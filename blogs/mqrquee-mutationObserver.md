---
title: MutationObserver 实现 react-fast-marquee 溢出滚动效果
date: 2024-03-01 23:57:00
keywords: MutationObserver
categories: 前端
tags:
  - JavaScript
---

# 利用 MutationObserver 实现 react-fast-marquee 溢出滚动效果

## 前言

> 在公司的项目需求中，需要公告栏内容小于指定宽度时，不能滚动，溢出时才显示滚动效果。然后使用`react-fast-marquee`时，发现通过 ref 没法获取到内容。

## 问题复现

```js
import { useEffect, useRef } from "react";
import Marquee from "react-fast-marquee";

import "./App.css";

function App() {
  const containerRef = useRef < HTMLDivElement > null;
  const marqueeRef = useRef(null);

  useEffect(() => {
    console.log(marqueeRef.current); // 值是null，并且marqueeRef更新的时候并不会触发useEffect。此时Marquee还没添加到DOM树上
    debugger;
  }, [marqueeRef]);

  return (
    <div className="marquee-container" ref={containerRef}>
      <Marquee ref={marqueeRef}>
        <span className="content">12</span>
      </Marquee>
    </div>
  );
}

export default App;
```

![](https://www.clzczh.top/CLZ_img/images/202403012343873.png)

> 没办法获取`marqueeRef`，那就没法操作 dom，得到元素的宽度进行判断。所以不可行。虽然可以通过 setTimeout 滞后获取，但是请教同事后，得到一个使用 MutationObserver 来实现的方案，而且 MutationObserver 之前刚好学过还写了篇文章，能用上项目里就很有 b 格。

## 实现

```js
import React, { useEffect, useRef, useState } from "react";
import Marquee from "react-fast-marquee";

import "./App.css";
import classNames from "classnames";

function App() {
  const containerRef = useRef < HTMLDivElement > null;
  const contentRef = useRef < HTMLSpanElement > null;

  const [scrollable, setScrollable] = useState(false);

  // 利用MutationObserver监听外层containerRef.current。观察子节点。
  useEffect(() => {
    if (!containerRef.current) {
      return;
    }

    const observer = new MutationObserver(() => {
      if (containerRef.current && contentRef.current) {
        if (contentRef.current.offsetWidth > containerRef.current.offsetWidth) {
          // 当内容的宽度大于容器的宽度时，设置为可以滚动
          setScrollable(true);
        }
      }
    });

    observer.observe(containerRef.current, {
      childList: true, // 观察子节点的新增与删除
    });

    return () => observer.disconnect(); // 结束观察
  }, [containerRef]);

  return (
    <div className="marquee-container" ref={containerRef}>
      <Marquee play={scrollable}>
        <span className="content" ref={contentRef}>
          12
        </span>
      </Marquee>
    </div>
  );
}

export default App;
```

样式

```css
.marquee-container {
  max-width: 200px;
}
```

![](https://www.clzczh.top/CLZ_img/images/202403012353413.png)

![](https://www.clzczh.top/CLZ_img/images/202403012355621.gif)

> 内容溢出时滚动 s
