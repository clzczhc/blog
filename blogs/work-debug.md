# 工作中的那点事之调试

## 调试`onMouseEnter`后出现的元素

众所周知，调试 DOM 元素时，可以使用 Elements 面板，hover 等状态则是可以通过 Styles 小面板，来修改。

但是，鼠标悬浮（`onMouseEnter`）后出现，鼠标移开就直接从 dom 树中移除的元素则无法直接通过 Element 面板调试，鼠标移到 Elements 面板时，元素已经消失了。

代码示例：

```tsx
function App() {
  const [textVisible, setTextVisible] = useState(false);

  return (
    <>
      <button
        onMouseEnter={() => {
          setTextVisible(true);
        }}
        onMouseLeave={() => {
          setTextVisible(false);
        }}
      >
        click
      </button>

      {textVisible ? <div style={{ color: "red" }}>clz</div> : null}
    </>
  );
}
```

### 步骤

1. 先到 Sources 面板，勾选上停用断点
   ![](https://www.clzczh.top/CLZ_img/images/20250310220623.png)
2. 鼠标悬浮到目标元素，然后点击`F8`键。

按上面的操作就可以让页面停留到点击`F8`键时的状态。
![](https://www.clzczh.top/CLZ_img/images/20250310221252.png)

## Chrome 小技巧

### Console 面板打印

`$0`: 打印当前选中的元素

`$_`：打印上一个打印的值。

这两个值笔者经常使用，可以直接在 Console 面板中操作 dom，查看信息。以及一些正则测试等操作。

### 为弹出式窗口自动打开 Devtools

调试代码时，有一些场景是点击后新窗口打开，此时想要看请求是不可行的，到新窗口后，打开 Devtools，空空如也。

可以通过偏好设置，设置一下**为弹出式窗口自动打开 Devtools**。这样即使新窗口打开也能看到所有的请求。

> 这个小技巧是公司的后端教笔者的，公司的老前端（8 年）甚至都不知道。

![](https://www.clzczh.top/CLZ_img/images/20250310222237.png)

![](https://www.clzczh.top/CLZ_img/images/20250310222317.png)

### 显示用户代理 Shadow DOM

![](https://www.clzczh.top/CLZ_img/images/20250310222810.png)

`input:range`元素：默认情况下，在 Elements 面板中只能看到`input:range`元素，但是实际上它是由容器、滚动块组合而成的，只不过是内置的。

跟上面的类似，可以设置一下偏好设置的**显示用户代理 Shadow DOM**。这样就可以在 Elements 面板中，选中更加具体的元素。

![](https://www.clzczh.top/CLZ_img/images/20250310223525.png)

![](https://www.clzczh.top/CLZ_img/images/20250310223634.png)

并且可以通过一些伪元素来修改默认样式：[::-webkit-slider-runnable-track](https://developer.mozilla.org/zh-CN/docs/Web/CSS/::-webkit-slider-runnable-track)

![](https://www.clzczh.top/CLZ_img/images/20250310223911.png)

### click-to-react-component

当项目代码非常多的时候，找对应的组件都有可能很困难。即使通过关键字查询，也可能会出现一堆匹配的代码。

这个时候可以使用`click-to-react-component`来辅助调试。

使用方式也非常简单：

1. 安装：`npm install click-to-react-component`
2. 入口文件引入

```tsx
import { createRoot } from "react-dom/client";
import { ClickToComponent } from "click-to-react-component";

import App from "./App";

const rootEl = document.querySelector("#root");
createRoot(rootEl!).render(
  <>
    <App />
    <ClickToComponent />
  </>
);
```

3. `alt + 左键`点击页面元素，即可打开 vscode，并定位到对应的代码。

![](https://www.clzczh.top/CLZ_img/images/20250312212247.png)
![](https://www.clzczh.top/CLZ_img/images/20250312212423.png)

> 从上面的链接，可以定位到具体文件的行、列。

![](https://www.clzczh.top/CLZ_img/images/20250312212731.png)
`alt + 右键`可以看到所有的父组件，同样可以选中具体的父组件，使用 vscode 打开。

可以看[React 项目里，如何快速定位你的组件源码？](https://juejin.cn/post/7374631918111178790)，会提到实现原理。
