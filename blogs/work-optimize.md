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

### `memo`的第二个参数

> 一般情况下不会用到。使用会有风险，自定义浅比较如果漏掉了相关属性，会导致组件的状态不改变。

`memo`默认情况下，判断需不需要`rerender`通过`shadowEqual`来进行浅比较，不过只会比较第一层。数据结构很深，可以通过第二个参数手动进行浅比较。

如：

```tsx
import { memo, useState } from "react";

interface ComponentProps {
  config: {
    user: {
      name: string;
      age: number;
      other: string;
    };
  };
}

export const Component = memo(
  (props: ComponentProps) => {
    const {
      config: { user },
    } = props;

    const { name, age } = user;

    console.log("rerender");

    return (
      <>
        <div>{name}</div>
        <div>{age}</div>
      </>
    );
  },
  (prevProps, nextProps) => {
    return (
      prevProps.config.user.name === nextProps.config.user.name &&
      prevProps.config.user.age === nextProps.config.user.age
    );
  }
);

export default function Home() {
  const [compProps, setCompProps] = useState<ComponentProps>({
    config: {
      user: {
        name: "clz",
        age: 24,
        other: "hello",
      },
    },
  });

  return (
    <>
      <Component {...compProps} />

      <div>{compProps.config.user.other}</div>

      <div className="btn-container">
        <button
          onClick={(): void => {
            setCompProps({
              config: {
                user: {
                  ...compProps.config.user,
                  name: "ccc",
                },
              },
            });
          }}
        >
          change name
        </button>
        <button
          onClick={(): void => {
            setCompProps({
              config: {
                user: {
                  ...compProps.config.user,
                  other: "hi",
                },
              },
            });
          }}
        >
          change other
        </button>
      </div>
    </>
  );
}
```

上面的例子中，父组件改变`other`的值也不会触发`User`组件的`rerender`。如果把`memo`的第二个参数去掉，则会触发`User`的`rerender`。

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
