# 工作中的那点事之 TypeScript

## `as const`处理 ts 类型误差

ts 的联合类型使用就经常容易出现类型误差。

```ts
export type Config = {
  type: "c" | "l" | "z";
  name: string;
}[];

const config: Config = [
  {
    type: "c",
    name: "赤蓝紫",
  },
];
```

上面的代码是没有 ts 类型错误的，但是有时候会需要根据一些参数来处理数组元素的内容，比如：

```ts
export type Config = {
  type: "c" | "l" | "z";
  name: string;
}[];

const flag = true;
const config: Config = [
  ...(flag
    ? [
        {
          type: "c",
          name: "clz",
        },
      ]
    : []),
  {
    type: "c",
    name: "czh",
  },
];
```

![](https://www.clzczh.top/CLZ_img/images/20250312222444.png)

原因则是使用扩展运算符那里，将`{ type: "c", name: "clz"}`识别成`{ type: string, name: string }`了。

这种情况下，可以使用`as const`来处理类似误差。
![](https://www.clzczh.top/CLZ_img/images/20250312223943.png)

## 获取函数返回值类型

```ts
ReturnType<typeof setTimeout>;
```

适用场景：要使用的函数，并没有把返回值类型`export`出来，并且需要保存返回值（通过`useRef`保存的时候，就需要得到返回值的类型）

例子（随便声明一个函数）：

```ts
enum CustomDate {
  Month = "month",
  Week = "week",
}

export function getCustomDate(a: number, b: number): CustomDate {
  if (a + b > 10) {
    return CustomDate.Month;
  }

  return CustomDate.Week;
}
```

![](https://www.clzczh.top/CLZ_img/images/20250312225039.png)
没有办法定义类型的话，就会像上面一样报错。

使用`ReturnType`就可以解决这个问题。

```ts
const customDate = useRef<ReturnType<typeof getCustomDate>>();
customDate.current = getCustomDate(10, 20);
```

![](https://www.clzczh.top/CLZ_img/images/20250312225227.png)
