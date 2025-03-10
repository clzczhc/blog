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
