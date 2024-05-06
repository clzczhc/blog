---
title: 使用 tiptap 实现简易富文本编辑器
categories: 前端
date: 2024-05-06 23:38:00
tags:
  - tiptap
  - 富文本
---

# 使用 tiptap 实现简易富文本编辑器

## 前言

> 在工作中遇到实现富文本编辑器的功能，最后是使用了 tiptap 来实现，记录一下。

## 简单使用

直接看官网的`快速开始`教程即可，安装`@tiptap/react`、`@tiptap/pm`、`@tiptap/starter-kit`这三剑客。

使用方法也很简单，使用`useEditor` Hook 得到 Editor 对象，然后赋值给`EditorContent`的 editor 属性即可

EditorX.tsx

```tsx
import { EditorContent, useEditor } from "@tiptap/react";
import StarterKit from "@tiptap/starter-kit";

import "./index.css";

const extensions = [StarterKit];

type EditorXProps = {
  defaultValue?: string;
  onChange?: (val?: string) => void;
};

const EditorX = (props: EditorXProps) => {
  const { defaultValue: content, onChange } = props;

  const editor = useEditor({
    extensions: [...extensions],
    content,
    onUpdate({ editor }) {
      if (!editor.getText()) {
        onChange?.(undefined);
      } else {
        onChange?.(editor.getText());
      }
    },
  });

  return (
    <>
      <EditorContent editor={editor} />
    </>
  );
};

export default EditorX;
```

index.css

```css
.tiptap.ProseMirror {
  min-height: 62px;
  outline: none;
}

.tiptap p.is-editor-empty:first-child::before {
  color: #adb5bd;
  content: attr(data-placeholder);
  float: left;
  height: 0;
  pointer-events: none;
}
```

app.tsx

```tsx
import { useState } from "react";
import EditorX from "./components/EditorX";

function App() {
  const [outerValue, setOuterValue] = useState("clz");

  const handleChange = (value?: string) => {
    if (value) {
      setOuterValue(value);
    }
  };

  return (
    <>
      {outerValue}
      <EditorX defaultValue="clz" onChange={handleChange} />
    </>
  );
}

export default App;
```

![](https://www.clzczh.top/CLZ_img/images/202405032257204.gif)

## 使用插件`@tiptap/extension-placeholder`

tiptap 编辑器并不是 input 这种输入框，而是通过普通的 div 元素，只是通过`contenteditable`这个属性来让 dom 元素可以编辑来实现的。所以自然不能直接使用 `placeholder`。tiptap 使用`placeholder`需要安装插件。

插件使用：

1. 引入

   > ```tsx
   > import Placeholder from "@tiptap/extension-placeholder";
   > ```

2. extensions 添加插件

   ```tsx
   const editor = useEditor({
     extensions: [
       ...extensions,
       Placeholder.configure({
         placeholder: "赤蓝紫到此一游",
       }),
     ],
     // ...
   });
   ```

![](https://www.clzczh.top/CLZ_img/images/202405032308654.gif)

## 添加操作菜单

```tsx
const EditorX = () => {
  // ...
  const handleBold = () => {
    editor?.commands.toggleBold();
  };

  return (
    <>
      <EditorContent editor={editor} />

      {editor && (
        <BubbleMenu editor={editor}>
          <span className="menu-bold" onClick={handleBold}>
            B
          </span>
        </BubbleMenu>
      )}
    </>
  );

  // ...
};
```

样式

```css
.menu-bold {
  display: block;
  padding: 0 12px;
  font-weight: bold;
  cursor: pointer;
}

.menu-bold:hover {
  background-color: #eee;
}
```

> 通过`BubbleMenu`展示操作菜单，并利用`editor?.commands`提供的 api 进行 tiptap 内置的一些操作，如上面的加粗。
> ![](https://www.clzczh.top/CLZ_img/images/202405032325196.gif)

## 粘贴图片

`useEditor`参数对象可以接收`editorProps`属性，而`editorProps`属性里面有一个`handlePaste`方法，该方法的第二个方法就可以获取粘贴的文件信息，然后再利用`FileReader`就可以把文件以`base64`码的形式读取。

```tsx
const handlePaste = async (view: EditorView, event: ClipboardEvent) => {
  const files = event.clipboardData?.files;
  const file = files?.item(0);

  if (!file || !file.type.startsWith("image/")) {
    return;
  }

  const reader = new FileReader();
  reader.readAsDataURL(file);

  reader.onload = () => {
    if (reader.result) {
      console.log(reader.result);
    }
  };
};

const editor = useEditor({
  extensions: [
    ...extensions,
    Placeholder.configure({
      placeholder: "赤蓝紫到此一游",
    }),
    Image.configure({
      inline: true,
    }),
  ],
  content,
  onUpdate({ editor }) {
    console.log(editor);

    if (!editor.getText()) {
      onChange?.(undefined);
    } else {
      onChange?.(editor.getText());
    }
  },
  editorProps: {
    handlePaste(view, event) {
      handlePaste(view, event);
    },
  },
});
```

![](https://www.clzczh.top/CLZ_img/images/202405062330746.gif)

然后通过添加`@tiptap/extension-image`插件把图片添加到编辑器上即可。

> $\color{red}{需要注意的是，需要配置`inline`为`true`，否则会出现图片粘贴上去，立马就被移出DOM树}$

```tsx
reader.onload = () => {
  if (reader.result) {
    editorRef.current
      ?.chain()
      .focus()
      .setImage({
        src: reader.result as string,
      })
      .run();
  }
};
```

![](https://www.clzczh.top/CLZ_img/images/202405062336265.gif)

### 完整代码

EditorX.tsx

```tsx
import { useRef } from "react";
import { BubbleMenu, Editor, EditorContent, useEditor } from "@tiptap/react";
import StarterKit from "@tiptap/starter-kit";
import Placeholder from "@tiptap/extension-placeholder";
import Image from "@tiptap/extension-image";

import { EditorView } from "@tiptap/pm/view";
import "./index.css";

const extensions = [
  StarterKit,
  Placeholder.configure({
    placeholder: "赤蓝紫到此一游",
  }),
  Image.configure({
    inline: true,
  }),
];

type EditorXProps = {
  defaultValue?: string;
  onChange?: (val?: string) => void;
};

const EditorX = (props: EditorXProps) => {
  const { defaultValue: content, onChange } = props;

  const editorRef = useRef<Editor | null>(null);
  const handlePaste = async (view: EditorView, event: ClipboardEvent) => {
    const files = event.clipboardData?.files;
    const file = files?.item(0);

    if (!file || !file.type.startsWith("image/")) {
      return;
    }

    const reader = new FileReader();
    reader.readAsDataURL(file);

    reader.onload = () => {
      if (reader.result) {
        editorRef.current
          ?.chain()
          .focus()
          .setImage({
            src: reader.result as string,
          })
          .run();
      }
    };
  };

  const editor = useEditor({
    extensions: extensions,
    content,
    onUpdate({ editor }) {
      if (!editor.getText()) {
        onChange?.(undefined);
      } else {
        onChange?.(editor.getText());
      }
    },
    editorProps: {
      handlePaste(view, event) {
        handlePaste(view, event);
      },
    },
  });
  editorRef.current = editor;

  const handleBold = () => {
    editor?.commands.toggleBold();
  };

  return (
    <>
      <EditorContent editor={editor} />

      {editor && (
        <BubbleMenu editor={editor}>
          <span className="menu-bold" onClick={handleBold}>
            B
          </span>
        </BubbleMenu>
      )}
    </>
  );
};

export default EditorX;
```

index.css

```css
.tiptap.ProseMirror {
  min-height: 62px;
  outline: none;
}

.tiptap p.is-editor-empty:first-child::before {
  color: #adb5bd;
  content: attr(data-placeholder);
  float: left;
  height: 0;
  pointer-events: none;
}

.menu-bold {
  display: block;
  padding: 0 12px;
  font-weight: bold;
  cursor: pointer;
}

.menu-bold:hover {
  background-color: #eee;
}
```
