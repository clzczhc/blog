---
title: 搭建一个简易的Playground
date: 2024-08-18 22:22:43
categories: 前端
tags:
  - Playground
---

# 搭建一个简易的Playground

## 前言
看一个动效的效果时，看到源码用到了`ace`实现了代码实时效果。（案例是直接使用`eval`执行代码，但是看了一下`codepen`的html结构，所以自己利用`ace`+`iframe`来搞一个简单的Playground玩玩。

> “玩后感”：感觉`ace`这个库使用起来还挺不方便的（比如设置主题什么的，参数是`ace/theme/twilight`这种形式，还得另外`import`），实际使用可能还应该调研一下其他的`editor`工具。

```js
import 'ace-builds/src-noconflict/theme-twilight';

//...
editor.setTheme('ace/theme/twilight');
```

## 核心
准备一个iframe用的`html`，然后对这个`html`的内容进行处理，比如`<script></script>`替换成`<script>${content}</script>`。处理结果会是字符串，所以还需要使用`URL.createObjectURL`转换成URL。
```js
URL.createObjectURL(
  new Blob([result], { type: "text/html" })
)
```

## 实现
用到`ace`的部分比较简单：
1. `ace.edit()`：参数为`dom`元素，把dom元素变成一个编辑器，返回editor对象
2. `editor.setTheme()`：设置主题
3. `editor.session.setValue()`: 设置编辑器初始值
4. `editor.getValue()`：获取编辑器当前值

代码：
App.tsx
```tsx
import { useCallback, useEffect, useRef, useState } from "react";
import ace, { Ace } from "ace-builds";
import confettiRaw from "./assets/confetti.browser.js?raw";   // 直接import文件会报错无法找到模块。加上?raw表示不当成模块处理
import iframeRaw from "./assets/iframe.html?raw";
import 'ace-builds/src-noconflict/theme-twilight';

import "./App.css";

function App() {
  const editorRef = useRef<HTMLDivElement>(null);
  const [editor, setEditor] = useState<Ace.Editor>();

  const handleInit = useCallback(() => {
    const defaultCode = `
      confetti({
        particleCount: 100,
          spread: 70,
          origin: { y: 1 },
      });
    `;

    if (editorRef.current) {
      const newEditor = ace.edit(editorRef.current);
      newEditor.setTheme('ace/theme/twilight');
      newEditor.session.setValue(defaultCode);

      setEditor(newEditor);
    }
  }, []);

  useEffect(() => {
    handleInit();
  }, [handleInit]);

  const importScriptSrc = URL.createObjectURL(
    new Blob([confettiRaw], { type: "text/javascript" })
  );
  const getIframeUrl = () => {
    const res = iframeRaw
    .replace(
      '<script id="import-script"></script>',
      `<script id="import-script" src="${importScriptSrc}"></script>`
    )
    .replace("<script></script>", `<script>${editor?.getValue()}</script>`);

  return URL.createObjectURL(
    new Blob([res], { type: "text/html" })
  );
}

  const [iframeUrl, setIframeUrl] = useState<string>(getIframeUrl());
  const handleClick = () => {
    setIframeUrl(getIframeUrl());
  };

  return (
    <>
      <button onClick={handleClick}>fire</button>
      <div className="box">
        <div ref={editorRef} id="editor"></div>
        <iframe src={iframeUrl}></iframe>
      </div>
    </>
  );
}

export default App;
```

iframe.html
```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Iframe</title>
  </head>
  <body>
    <script id="import-script"></script>
    <script></script>
  </body>
</html>
```

第三方javascript: `confetti.browser.js`可以替换成自己需要的第三方资源

## 效果
![](https://www.clzczh.top/CLZ_img/images/202408182126654.gif)

## 添加`js-beautify`美化代码
初始化编辑器的代码时，传入的字符串格式可能会有问题。
![](https://www.clzczh.top/CLZ_img/images/202408182148543.png)

比如上面的例子，初始化的效果就是这样。所以可以`setValue`之前用`js-beautify`做一下美化。
```js
import beautify from 'js-beautify';

const pretty = (code: string) => {
  return beautify.js(code);
}

// ...
newEditor.session.setValue(pretty(defaultCode));
```
![](https://www.clzczh.top/CLZ_img/images/202408182150919.png)

## 使用`monaco-editor`
`monaco-editor`是VSCode的代码编辑器。`monaco-editor`也是功能很强大，还有代码提示功能，写法也是组件写法。

```tsx
import Editor from "@monaco-editor/react";
import "./App.css";

function App() {

  return (
    <Editor height="60vh" defaultLanguage="javascript" defaultValue="console.log('Hello clz!!!')" theme="vs-dark" />
  );
}

export default App;
```

![](https://www.clzczh.top/CLZ_img/images/202408182220908.png)
![](https://www.clzczh.top/CLZ_img/images/202408182221724.png)