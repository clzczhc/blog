# 工作中的那点事之 React

## PropsWithChildren

> Props 声明带有 children 的 props。

```ts
export type AProps = PropsWithChildren<{
  onPress: () => void;
}>;
```

> PropsWithChildren 有可能会类型报错（直接用 vite、react@18.3.1生成的项目没有报错）

如果报错，则需要用一些类型断言。

```tsx
export const Radio = memo(function Radio(props: RadioProps) {
  // ...
}) as (props: RadioProps) => ReactElement;
```

## ref 传参回调

> react 开发中，经常会遇到使用 ref 操作 dom 的场景，其中也不乏很简单的操作，比如让 input 聚焦。这种简单的操作就可以直接回调的方式来实现。

一般做法：

```tsx
function App() {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    if (inputRef.current) {
      inputRef.current.focus();
    }
  }, []);

  return <input ref={inputRef} />;
}
```

传参回调：

```tsx
function App() {
  const refCallback = useCallback((ref: HTMLInputElement) => {
    ref.focus();
  }, []);

  return <input ref={refCallback} />;
}
```

效果一样。

## 组件间传递数据

> 非常基础的常规的 props、回调函数应该没必要啰嗦了。

### `useImperativeHandle`、`forwardRef`

父组件可以通过回调函数的形式获取子组件返回的数据，不过这种形式没有办法在父组件主动获取子组件的数据。
可以使用`useImperativeHandle`暴露子组件的数据，需要搭配`forwardRef`使用

父组件

```tsx
import { useRef } from "react";
import { Son, SonHandles } from "./Son";

export const Father = () => {
  const ref = useRef<SonHandles>(null);
  const onClick = (): void => {
    console.log(ref.current?.getCount());
  };

  return (
    <>
      <Son ref={ref} count={100} />

      <div>
        <button onClick={onClick}>getSonCount</button>
      </div>
    </>
  );
};
```

子组件

```tsx
import { ForwardedRef, forwardRef, useImperativeHandle, useState } from "react";

interface SonProps {
  count: number;
}

export interface SonHandles {
  getCount: () => number;
}

export const Son = forwardRef(
  (props: SonProps, ref: ForwardedRef<SonHandles>) => {
    const { count: countProp } = props;
    const [count, setCount] = useState(countProp);

    useImperativeHandle(ref, () => ({
      getCount: () => count,
    }));

    return (
      <>
        <div>{count}</div>
        <button onClick={(): void => setCount(count - 1)}>-1</button>
        <button onClick={(): void => setCount(count + 1)}>+1</button>
      </>
    );
  }
);
```

![](https://www.clzczh.top/CLZ_img/images/202504042207371.gif)

### `Context`

> `Context`可以实现跨组件传输数据。添加数据用的`Context`的话，还可以完全不借助第三方的状态库，完全使用`React`实现。

count-context.tsx

```tsx
import { createContext, ReactNode, useContext } from "react";

export interface CountContextModel {
  count: number;
  changeCount: (count: number) => void;
}

const CountContext = createContext<CountContextModel>({
  get count(): never {
    throw new Error("count is not implemented");
  },
  get changeCount(): never {
    throw new Error("changeCount is not implemented");
  },
});

export interface CountProviderProps {
  children: ReactNode;
  value: CountContextModel;
}

export const CountProvider = (props: CountProviderProps) => {
  const { children, value } = props;
  return (
    <CountContext.Provider value={value}>{children}</CountContext.Provider>
  );
};

export const useCount = () => {
  return useContext(CountContext).count;
};

export const useChangeCount = () => {
  const { changeCount } = useContext(CountContext);

  return (count: number) => {
    changeCount(count);
  };
};
```

App.tsx

```tsx
import { useState } from "react";
import { CountProvider } from "./context/count-context";
import { Comp1 } from "./components/Comp1";
import { Comp2 } from "./components/Comp2";

export default function App() {
  const [count, setCount] = useState(100);

  return (
    <CountProvider value={{ count, changeCount: setCount }}>
      <Comp1 />
      <Comp2 />
    </CountProvider>
  );
}
```

Comp1.tsx

```tsx
import { useCount } from "../context/count-context";

export const Comp1 = () => {
  const count = useCount();

  return (
    <div>
      comp1
      <div>{count}</div>
    </div>
  );
};
```

Comp2.tsx

```tsx
import { useChangeCount, useCount } from "../context/count-context";

export const Comp2 = () => {
  const count = useCount();
  const changeCount = useChangeCount();

  return (
    <div>
      comp2
      <div>{count}</div>
      <button onClick={(): void => changeCount(count - 1)}>-1</button>
      <button onClick={(): void => changeCount(count + 1)}>+1</button>
    </div>
  );
};
```

![](https://www.clzczh.top/CLZ_img/images/202504042242773.gif)
