---
title: JavaScript之对象(一)
categories: 前端
date: 2022-04-30 17:21:11
tags:
  - JavaScript
---

# JavaScript之对象(一)
> 看红宝书+查资料，重新梳理JavaScript的知识。

## 对象的属性

### 数据属性

数据属性有四个特性。通过特性，可以设置属性。如通过`[[Enumerable]]`为`false`就能不让该属性被枚举。另外，为了区别是不是特殊的属性，规范会用两个中括号将特性的名称括起来，如`[[Writable]]`。

-   `[[Configurable]]`: 表示属性是否可以被设置。如被`delete`，以及能否修改特性等。
-   `[[Enumerable]]`: 表示属性是不是可被枚举的。
-   `[[Writable]]`: 表示属性的值是否能被修改
-   `[[Value]]`: 表示属性实际值

如果我们直接使用字面量的形式将属性显示添加到对象之后，`[[Configurable]]`、`[[Enumerable]]`、`[[Writable]]`会被设置为`true`，而`[[Value]]`特性则会被设置为指定的值。

```
let person = {
    name: 'clz'
}
```

**设置成不可修改、不可定义的时候，修改、删除属性都不会报错，不过也不会成功，会被忽略，相当于什么都没干**

```
Object.defineProperty(person, "name", {
    writable: false,        // 默认值为false，所以去掉也是一样的结果
    configurable: false,    // 默认值为false，所以去掉也是一样的结果
    value: "clz",
});

console.log(person);

person.name = "czh";
console.log(person);

delete person.name
console.log(person);
```

![image-20220423172621243](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2a58533c21f14c03886389c97c1da605~tplv-k3u1fbpfcp-zoom-1.image)

如果去掉上面注释的两行，也能得到一样的结果，因为默认值就是`false`。

如果设置为`true`，则结果大不相同，因为此时能够修改属性，也能够设置属性了。

```
let person = {};

Object.defineProperty(person, "name", {
    writable: true,
    configurable: true,
    value: "clz",
});

console.log(person);

person.name = "czh";
console.log(person);

delete person.name
console.log(person);
```

![image-20220423172933796](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c555cf3c54314ea58ca486a0657e0575~tplv-k3u1fbpfcp-zoom-1.image)

上面我们可以发现，当我们设置成不可修改时，修改只是不会生效，并不会报错。所以如果我们想要修改设置为不可修改的属性的话，可以使用严格模式，此时还强行修改的话，就会报错。

能反复调用`Object.defineProperty`，但是如果那四个特性有修改就会报错，不知道有啥意义

```
let person = {};

Object.defineProperty(person, "name", {
  value: "clz",
});
console.log(person);

Object.defineProperty(person, "name", {
  value: "czh",
});
console.log(person);
```

![image-20220423170434625](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4da1ab7def374de59817f9d4beecb423~tplv-k3u1fbpfcp-zoom-1.image)

### 访问器属性

-   `[[Configurable]]`
-   `[[Enumerable]]`
-   `[[Get]]`: 获取函数，在读取属性时调用，默认值为 `undefined`
-   `[[Set]]`: 设置函数，在写入函数是调用，默认值是`undefined`

**访问器属性和数据属性中不重合的特性不能同时使用**，比如，如果使用`setter`和`getter`，那再使用`writable`或`value`就会报错。至于为什么会报错，就是因为会有冲突，比如既设置了`value`和`getter`，那么这个时候应该怎么获取数据呢？所以多一事不如少一事，**数据属性和访问器属性不重合的特性不能同时使用**。(后半部分是猜想)

```
let person = {
  name_: "clz",
};

Object.defineProperty(person, "name", {
  // writable: false,        // 互不相容，如果添加这个则会导致报错
  get() {
    console.log("getter");
    return this.name_;
  },
  set(newValue) {
    console.log("setter");
    this.name_ = newValue;
  },
});

console.log(person);
console.log(person.name);

console.log("%c%s", "color: red;font-size:24px;", "===============");

person.name = "czh";
console.log(person);
console.log(person.name);
```

![image-20220423171002134](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/976e36ec3f5d4e37a2b19479a304093e~tplv-k3u1fbpfcp-zoom-1.image)

通过观察上面的例子，不难发现，在我们获取数据或者修改数据时，会进入到`get`和`set`属性中，所以通过这两个属性就能实现数据的劫持。

## 定义多个属性

使用`Object.defineProperty`只能定义一个属性的特殊属性。我们可以通过`Object.defineProperties`定义多个属性。

```
let person = {};

Object.defineProperties(person, {
    name_: {
        value: "clz",
    },
    name: {
        get() {
            console.log("getter");
            return this.name_;
        },
        set(newValue) {
            console.log("setter");
            this.name_ = newValue;
        },
    },
});

console.log(person);
console.log(person.name);

console.log("%c%s", "color: red;font-size:24px;", "===============");

person.name = "czh";
console.log(person);
console.log(person.name);
```

![image-20220423171726675](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/07e71537d116447e9edfc166e5235df3~tplv-k3u1fbpfcp-zoom-1.image)

## 读取属性的特性

在上面我们已经知道怎样定义属性，以设置属性的特性了，有没有什么方法可以读取属性的特性呢？

通过`Object.getOwnPropertyDescriptor`就可以读取属性的特性，其中第一个参数是对象，第二个参数是要读取特性的属性。

```
let person = {};
Object.defineProperties(person, {
    name_: {
        value: "clz",
    },
    name: {
        get: function () {
            return this.name_;
        },
        set: function (newValue) {
            this.name_ = newValue;
        },
    },
});

let descriptor = Object.getOwnPropertyDescriptor(person, "name_");
console.log(descriptor);

descriptor = Object.getOwnPropertyDescriptor(person, "name");
console.log(descriptor);
```

![image-20220423191037056](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d763f617556481aae104e4afa8c2c19~tplv-k3u1fbpfcp-zoom-1.image)

通过`Object.getOwnPropertyDescriptors(person)`可以获取所有属性的特性(对象形式)

```
let person = {};
Object.defineProperties(person, {
    name_: {
        value: "clz",
    },
    name: {
        get: function () {
            return this.name_;
        },
        set: function (newValue) {
            this.name_ = newValue;
        },
    },
});

let descriptors = Object.getOwnPropertyDescriptors(person);
console.log(descriptors);
```

![image-20220423191225329](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/62a6fb85c8f045209b570c19f72355c3~tplv-k3u1fbpfcp-zoom-1.image)
