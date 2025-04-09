---
title: 工作中的那点事之 性能优化
date: 2025-04-08 20:48:05
categories: "前端"
tags:
  - 工作中的那点事
  - 性能优化
---

# 工作中的那点事之 性能优化

## 无效渲染

> 一开始想写一下总结的，工作中重构项目用到了一些**无效渲染优化**。但是后面看到下面的文章，已经总结的很好了

[react 无效渲染优化--工具篇](https://www.cnblogs.com/echolun/p/17110031.html)
[请删掉 99%的 useMemo](https://zhuanlan.zhihu.com/p/678698481)
[One simple trick to optimize React re-renders](https://kentcdodds.com/blog/optimize-react-re-renders)

## 组件懒加载

`lazy`和`Suspense`联合使用。

```tsx
import { lazy, Suspense } from "react";

const Home = lazy(() => import("./components/Home"));

export default function App() {
  return (
    <Suspense fallback="loading">
      <Home />
    </Suspense>
  );
}
```

使用`Suspense`可以在等待加载`lazy`组件时，显示加载内容，如上面的`loading`文案。

还可以封装成一个公共方法。

```tsx
import { lazy, ReactElement, Suspense } from "react";

const Home = lazy(() => import("./components/Home"));

const withLoadingComponent = (component: ReactElement) => (
  <Suspense fallback="loading">{component}</Suspense>
);

export default function App() {
  return withLoadingComponent(<Home />);
}
```
