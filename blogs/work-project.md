---
title: 工作中的那点事之 项目相关
date: 2025-03-23 13:48:05
categories: "前端"
tags:
  - 工作中的那点事
  - 项目
---

# 工作中的那点事之 项目相关

## 项目依赖冲突

项目同时使用了依赖 a 和 b，而且 a 和 b 都使用了依赖 c，不过版本不一样。

大多数情况下，`npm`可以自动解决依赖项之间的冲突。但是也有解决不了冲突的情况，这种时候可以通过配置`resolutions`来解决。

`resolutions`字段可以指定使用哪个版本。

比如(假设下面的 a 依赖 1.0.0 版本的 c，b 则依赖 2.0.0)

```json
{
  "dependencies": {
    "a": "^1.0.0",
    "b": "^2.0.0"
  },
  "resolutions": {
    "c": "1.0.0"
  }
}
```

## `madge`检查循环依赖

使用起来比较简单，scripts 添加命令`"madge": "madge ./src --circular --extensions ts,tsx"`，然后执行`npm run madge`即可。
会检查`src`目录下`ts`、`tsx`文件有没有存在循环依赖。

### 例子

app.tsx

```tsx
import { UserCard } from "./components/UserCard";

export interface User {
  id: string;
  name: string;
}

export default function App() {
  return (
    <UserCard
      user={{
        id: "1",
        name: "clz",
      }}
      onPress={() => {
        console.log("a");
      }}
    />
  );
}
```

UserCard.tsx

```tsx
import { memo } from "react";
import { User } from "../App";

interface UserProps {
  user: User;
  onPress: () => void;
}

export const UserCard = memo((props: UserProps) => {
  const { user, onPress } = props;
  return <div onClick={onPress}>{user.name}</div>;
});
```

![](https://www.clzczh.top/CLZ_img/images/20250323121221.png)
依赖关系大致如上图。

![](https://www.clzczh.top/CLZ_img/images/20250323121325.png)
执行命令后，会显示哪里出现了循环依赖

把上面的`type User`移动到`UserCard.ts`中后，再执行命令
![](https://www.clzczh.top/CLZ_img/images/20250323121525.png)

### 搭配`husky`、`liny-staged`使用

检查循环依赖只需要在提交代码的时候，对改动文件进行检测即可，所以可以搭配`husky`、`liny-staged`使用。

`husky`、`lint-staged`的使用可以自行学习，也可以参考笔者之前的[React 搭建规范](https://www.clzczh.top/2023/10/03/react-standard)

`package.json`添加`lint-staged`：

```json
"lint-staged": {
  "*.{js,jsx,ts,tsx}": [
    "madge --circular"
  ]
}
```

`lint-staged`已经限制了会被检测的文件，所以`scripts`下的命令也可以简化为`"madge": "madge ./src"`。

效果：
![](https://www.clzczh.top/CLZ_img/images/20250323123600.png)

### `-i madge.png`生成依赖图

> **需要安装`Graphviz`，并配置环境变量**

1. 不能直接在`lint-staged`中配置，不然循环依赖检查就不生效了（生成图片，但是提交并没有被拦截）
2. 配置命令：`"madge": "madge ./src -i madge.png"`。commit 检查的时候不会生成依赖图片。

综上所述，生成依赖图应该单独作为一个命令。`"generate:madge": "madge ./src -i madge.png --extensions ts,tsx"`
![](https://www.clzczh.top/CLZ_img/images/20250323132530.png)

> 改成上面的命令后，就不再是检测循环依赖，而是生成依赖图片。可以从图片中看依赖关系有没有问题。

## 项目分包

[npm、yarn 使用 workspaces](https://www.clzczh.top/2024/04/13/workspaces/)
