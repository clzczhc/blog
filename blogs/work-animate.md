---
title: 工作中的那点事之 动画
date: 2025-03-24 20:08:05
categories: "前端"
tags:
  - 工作中的那点事
  - 动画
---

# 工作中的那点事之 动画

## AutoAnimate

[AutoAnimate](https://auto-animate.formkit.com/)

可以零配置实现添加动画。

使用方法则是使用`useAutoAnimate`hook，会返回第一个元素为`ref`的数组。把该`ref`放到想要添加动画的元素上即可。

```tsx
import { useAutoAnimate } from "@formkit/auto-animate/react";

function MyList() {
  const [animationParent] = useAutoAnimate();
  return (
    <ul ref={animationParent}>{/* 🪄 Magic animations for your list */}</ul>
  );
}
```

自动动画会应用于父元素（设置 auto animate 的 ref）的**直属子元素**。

> 官方的文档说是会应用于父元素及其直属父元素。但是直接移除父元素并不会触发动画。

1. DOM 中添加了子元素。
2. DOM 中的子元素被移除。
3. 子元素在 DOM 中被移动

> 当发生上述三种情况的时候，会触发动画

### 例子

在官网的 demo 中加了一个直接移除父元素的情况。代码也比较简单易懂。

```tsx
import { useState } from "react";
import { useAutoAnimate } from "@formkit/auto-animate/react";

export default function App() {
  const [items, setItems] = useState([1, 2, 3]);
  const [parent, enableAnimations] = useAutoAnimate();

  const add = () => setItems([...items, items.length + 1]);
  const remove = () =>
    setItems((prev) => {
      prev.splice(1, 1);
      return [...prev];
    });

  const [visible, setVisible] = useState(true);
  const removeParent = () => {
    setVisible(false);
  };

  return (
    <>
      {visible ? (
        <ul ref={parent}>
          {items.map((item) => (
            <li key={item}>{item}</li>
          ))}
        </ul>
      ) : null}

      <button onClick={add}>Add number</button>
      <button onClick={remove}>Rmove number</button>
      <button onClick={removeParent}>Rmove Parent</button>
      <button onClick={() => enableAnimations(false)}>Disable</button>
    </>
  );
}
```

效果：
![](https://www.clzczh.top/CLZ_img/images/auto-animate.gif)
![](https://www.clzczh.top/CLZ_img/images/202503241104819.gif)

图 1 中是启用 AutoAnimate 的效果，图 2 则是关闭后的效果。

## React Transition Group

> Transition 组件使用会把样式跟组件耦合在一起，使用也不是很方便。所以主要介绍一下`CSSTransition`

### CSSTransition

使用方法跟 Vue 的`transition`组件类似。

```tsx
import { useRef, useState } from "react";
import { CSSTransition } from "react-transition-group";
import "./App.css";

export default function App() {
  const [visible, setVisible] = useState(false);
  const nodeRef = useRef(null);

  return (
    <div>
      <CSSTransition
        nodeRef={nodeRef}
        in={visible}
        timeout={200}
        unmountOnExit
        classNames="container"
      >
        <>
          <div ref={nodeRef}>I am CLZ!!!</div>
          <div>no transition</div>
        </>
      </CSSTransition>

      <button type="button" onClick={() => setVisible(true)}>
        Click to Enter
      </button>

      <button type="button" onClick={() => setVisible(false)}>
        Click to Leave
      </button>
    </div>
  );
}
```

> `nodeRef`: 需要转换的 don 元素的 React 引用
> `in`: `CSSTransition`的`children`是否显示
> `unmountOnExit`：触发`exit`后，是否从 dom 树中移除。触发`exit`的时间一般是`in`对应的值修改为`false`。

**`CSSTransition`还需要添加样式才能成功添加动画**。类名的规则如下：

> 其中`$$`为`CSSTransition`的`classNames`（这里的`s`是因为还可以给每个阶段单独指定`className`，具体查看官网。

1. `$$-enter`: 触发`enter`前的样式
2. `$$-enter-active`：触发`enter`后的样式。效果就是从`$$-enter`过渡到`$$-enter-active`的样式
3. `$$-exit`: 触发`exit`前的样式
4. `$$-exit-active`：触发`exit`后的样式。同`$$-enter-active`

```css
.container-enter {
  transform: translateX(-100px);
  opacity: 0;
}
.container-enter-active {
  transform: translateX(0);
  opacity: 1;
  transition: all 200ms;
}
.container-exit {
  transform: translateX(0);
  opacity: 1;
}
.container-exit-active {
  transform: translateX(-100px);
  opacity: 0;
  transition: all 200ms;
}
```

效果：
![](https://www.clzczh.top/CLZ_img/images/202503242004727.gif)
