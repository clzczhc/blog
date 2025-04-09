---
title: 工作中的那点事之 TypeScript
date: 2025-03-12 22:00:05
categories: "前端"
tags:
  - 工作中的那点事
  - TypeScript
---

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

## `ts-node`运行 ts 脚本

脚本需要类型，使用 ts 编写脚本的话，无法直接运行。可以通过安装`ts`来实现。

使用方法也非常简单，`package.json`的`scripts`添加对应的命令，如`execute: ts-node index.ts`即可。也可以直接命令行`npx ts-node index.ts`。

**注意**：`ts-node`只能在`commonJS`中执行。不过可以还是可以直接在`ts`文件中使用`import`、`export`。

这个[issue](https://github.com/TypeStrong/ts-node/issues/2120)中有提到。

> 项目开发中，可以添加`-T`参数：只转译，不做类型检查。类型检查可以通过`vscode`，以及提交前检查。

## `is` 类型保护

联合类型可以通过公共属性进行类型保护。

如：

```ts
interface BaseUser {
  name: string;
}

interface FreeUser extends BaseUser {
  type: "free";
  sayHello: () => void;
}

interface VIPUser extends BaseUser {
  type: "vip";
  sayHi: () => void;
}

export interface Props {
  user: FreeUser | VIPUser;
}

export const handler = (props: Props) => {
  const { user } = props;

  if (user.type === "free") {
    user.sayHello();
  } else {
    user.sayHi();
  }
};
```

没有公共属性可以用于区分的时候，可以使用`is`类型保护。

```ts
const isVIPUser = (user: any): user is VIPUser => {
  return user.sayHi !== undefined;
};
```

> 使用具体类型的属性、方法前，调用一下该方法即可。
> 如：

```ts
interface BaseUser {
  name: string;
}

interface FreeUser extends BaseUser {
  sayHello: () => void;
}

interface VIPUser extends BaseUser {
  sayHi: () => void;
}

export interface Props {
  user: FreeUser | VIPUser;
}

const isVIPUser = (user: any): user is VIPUser => {
  return user.sayHi !== undefined;
};

export const handler = (props: Props) => {
  const { user } = props;

  if (!isVIPUser(user)) {
    user.sayHello();
  } else {
    user.sayHi();
  }
};
```
