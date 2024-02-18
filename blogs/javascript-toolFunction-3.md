---
title: 自定义工具函数库(三)
date: 2022-03-22 13:38:47
categories: 前端
tags:
  - JavaScript
---

# 自定义工具函数库(三)

最终仓库：[utils: 自定义工具库](https://github.com/13535944743/utils)

## 1. 自定义 instanceof

> - 语法: myInstanceOf(obj, Type)
> - 功能: 判断 obj 是否是 Type 类型的实例
> - 实现: Type 的原型对象是否是 obj 的原型链上的某个对象, 如果是返回 true, 否则返回 false

之前的笔记：[详解原型链](https://www.clzczh.top/2022/03/16/javascript-prototype/)

```js
// 自定义instanceof

function myInstanceof(obj, fn) {
  let prototype = fn.prototype; // 获取函数的显示原型

  let proto = obj.__proto__; // 获取obj的隐式原型对象

  // 遍历原型链
  while (proto) {
    if (prototype === proto) {
      return true;
    }
    proto = proto.__proto__;
  }
  return false;
}
```

测试

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>对象相关</title>
    <script src="./myInstanceof.js"></script>
  </head>

  <body>
    <script>
      function Person() {}

      let p = new Person();

      console.log(myInstanceof(p, Person));
      console.log(myInstanceof(p, Object));
      console.log(myInstanceof(Person, Object));
      console.log(myInstanceof(Person, Function));
      console.log(myInstanceof(p, Function));
    </script>
  </body>
</html>
```

## 2. 对象/数组拷贝

### 2.1 浅拷贝与深拷贝

**深拷贝和浅拷贝只针对 Object 和 Array 这样的引用数据类型**。

- 浅拷贝：只复制某个对象的引用地址值，而不复制对象本身，新旧对象还是共享同一块内存(<b style="color: red">即修改旧对象引用类型也会修改到新对象</b>)
- 深拷贝：新建一个一摸一样的对象，新对象与旧对象不共享内存，所以修改新对象不会跟着修改原对象。

### 2.2 浅拷贝

#### 2.2.1 利用扩展运算符...实现

```js
// 浅拷贝1：利用扩展运算符...实现
function shallowClone(target) {
  if (typeof target === "object" && target !== null) {
    if (Array.isArray(target)) {
      return [...target];
    } else {
      return { ...target };
    }
  } else {
    return target; // 是null或者不是引用数据类型直接返回
  }
}

// const obj = { x: 'clz', y: { age: 21 } }
// const cloneObj = shallowClone(obj)
// console.log(cloneObj)

// obj.y.age = 111
// console.log(obj, cloneObj)    // 浅拷贝，修改旧对象的引用类型会同步修改新对象

// obj.x = 'xxx'               // 修改的若不是引用数据类型的没有影响
// console.log(obj, cloneObj)
```

#### 2.2.2 遍历实现

```js
// 浅拷贝2：遍历实现
function shallowClone(target) {
  if (typeof target === "object" && target !== null) {
    let ret = Array.isArray(target) ? [] : {};

    for (let key in target) {
      // 遍历拷贝
      if (target.hasOwnProperty(key)) {
        // 不需要考虑原型链上的属性
        ret[key] = target[key];
      }
    }

    return ret;
  } else {
    return target; // 是null或者不是引用数据类型直接返回
  }
}

// const obj = { x: 'clz', y: { age: 21 } }
// const cloneObj = shallowClone(obj)
// console.log(cloneObj)

// obj.y.age = 111
// console.log(obj, cloneObj)    // 浅拷贝，修改旧对象的引用类型会同步修改新对象

// obj.x = 'xxx'               // 修改的若不是引用数据类型的没有影响
// console.log(obj, cloneObj)
```

### 2.3 深拷贝

#### 2.3.1 JSON 转换

<b style="color: red">不能拷贝对象方法</b>

```js
// 深拷贝1: 通过JSON转换
function deepClone(target) {
  let str = JSON.stringify(target); // 转换为字符串

  return JSON.parse(str); // 再把字符串转换为对象
}

// const obj = { x: 'clz', y: { age: 21 }, z: { name: 'clz' }, f: function () { } }
// const cloneObj = deepClone(obj)
// console.log(cloneObj)

// // obj.y.z = obj.z
// // obj.z.y = obj.y  // 循环引用

// obj.y.age = 111
// console.log(obj, cloneObj)
```

#### 2.3.2 递归

```js
// 深拷贝2: 通过递归实现
function deepClone(target) {
  if (typeof target === "object" && target !== null) {
    const ret = Array.isArray(target) ? [] : {};

    for (const key in target) {
      if (target.hasOwnProperty(key)) {
        ret[key] = deepClone(target[key]); // 递归调用函数实现深拷贝，直接赋值的话则只是浅拷贝，因为是引用数据类型时只是复制了引用地址而已
      }
    }

    return ret;
  } else {
    return target;
  }
}
```

测试：

```js
const obj = { x: "clz", y: { age: 21 }, z: { name: "clz" }, f: function () {} };
const cloneObj = deepClone(obj);

obj.y.age = 111;
console.log(obj, cloneObj);
```

开开心心收工？有点问题，如果对象中有循环引用，即"你中有我，我中有你"的话，就会导致形成死循环，会导致无法跑出结果，直到超出最大调用堆栈大小

怎么解决这个 bug 呢？使用 map 来存取拷贝过的数据，每次调用函数时判断有无拷贝过，有的话，直接返回之前拷贝的数据就行了。而且，这里还有个有意思的地方：<b style="color: red">递归调用函数需要共享变量时，可以通过添加一个参数，一直传同一个变量</b>

改进后：

```js
// 深拷贝2: 通过递归实现：使用map来存取拷贝过的数据，每次调用函数时判断有无拷贝过，有的话，直接返回之前拷贝的数据就行了。
function deepClone(target, map = new Map()) {
  if (typeof target === "object" && target !== null) {
    const cache = map.get(target);
    if (cache) {
      return cache; // 有拷贝过的话，直接返回之前拷贝过的数据
    }

    const ret = Array.isArray(target) ? [] : {};

    map.set(target, ret); // 键为拷贝的目标，值为拷贝的结果

    for (const key in target) {
      if (target.hasOwnProperty(key)) {
        ret[key] = deepClone(target[key], map); // 递归调用函数实现深拷贝，直接赋值的话则只是浅拷贝，因为是引用数据类型时只是复制了引用地址而已
      }
    }

    return ret;
  } else {
    return target;
  }
}

// const obj = { x: 'clz', y: { age: 21 }, z: { name: 'clz' }, f: function () { } }

// obj.y.z = obj.z
// obj.z.y = obj.y  // 循环引用可能会造成死循环

// const cloneObj = deepClone(obj)

// obj.y.age = 111
// console.log(obj, cloneObj)
```

**优化遍历性能**：

> - 数组: while | for | forEach() 优于 for-in | keys()&forEach()
> - 对象: for-in 与 keys()&forEach() 差不多

变更部分：分成数组和对象分别处理，使用更优的遍历方式(**个人看不出有什么大的区别**，先记一下)

```js
if (Array.isArray(target)) {
  target.forEach((item, index) => {
    ret[index] = deepClone(item, map);
  });
} else {
  Object.keys(target).forEach((key) => {
    ret[key] = deepClone(target[key], map);
  });
}
```

## 3. 事件

[JavaScript 事件回顾](https://clz.vercel.app/2021/09/09/javascript-Event/)

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      .outter {
        position: relative;
        width: 200px;
        height: 200px;
        background-color: red;
      }

      .inner {
        position: absolute;
        left: 0;
        right: 0;
        top: 0;
        bottom: 0;
        width: 100px;
        height: 100px;
        margin: auto;
        background-color: blue;
      }
    </style>
  </head>

  <body>
    <div class="outter">
      <div class="inner"></div>
    </div>
    <script>
      const outter = document.querySelector(".outter");
      const inner = document.querySelector(".inner");

      outter.addEventListener(
        "click",
        function () {
          console.log("捕获 outter");
        },
        true
      ); // true表示在事件捕获阶段， false或不传参表示在事件冒泡阶段

      inner.addEventListener(
        "click",
        function () {
          console.log("捕获 inner");
        },
        true
      );

      outter.addEventListener(
        "click",
        function () {
          console.log("冒泡 outter");
        },
        false
      ); // true表示在事件捕获阶段， false或不传参表示在事件冒泡阶段

      inner.addEventListener("click", function () {
        console.log("冒泡 inner");
      });
    </script>
  </body>
</html>
```

### 3.1 自定义事件委托函数

自定义事件委托函数关键：获取真正触发事件的目标元素，若和子元素相匹配，则使用 call 调用回调函数(this 指向，变更为 target)

```js
function addEventListener(el, type, fn, selector) {
  // selector是子元素
  el = document.querySelector(el);

  if (!selector) {
    el.addEventListener(type, fn); // 没有传递子元素的选择器，则是普通的给el元素绑定事件
  } else {
    el.addEventListener(type, function (e) {
      const target = e.target; // 获取真正触发事件的目标元素

      if (target.matches(selector)) {
        // 判断选择器与目标元素是否符合
        fn.call(target, e);
      }
    });
  }
}
```

测试

```html
<ul id="items">
  <li>01</li>
  <li>02</li>
  <li>03</li>
  <li>04</li>
  <li>05</li>
  <p>06</p>
</ul>

<script>
  addEventListener(
    "#items",
    "click",
    function () {
      this.style.color = "red";
    },
    "li"
  );
</script>
```

### 3.2 手写事件总线

> on(eventName, listener): 绑定事件监听
> emit(eventName, data): 分发事件
> off(eventName): 解绑指定事件名的事件监听, 如果没有指定解绑所有

```js
// 事件总线
// on(eventName, listener): 绑定事件监听
// emit(eventName, data): 分发事件
// off(eventName): 解绑指定事件名的事件监听, 如果没有指定解绑所有

class EventBus {
  constructor() {
    this.callbacks = {};
  }

  on(eventName, fn) {
    if (this.callbacks[eventName]) {
      this.callbacks[eventName].push(fn);
    } else {
      this.callbacks[eventName] = [fn];
    }
  }

  emit(eventName, data) {
    let callbacks = this.callbacks[eventName];
    if (callbacks && this.callbacks[eventName].length !== 0) {
      callbacks.forEach((callback) => {
        callback(data);
      });
    }
  }

  off(eventName) {
    if (eventName) {
      if (this.callbacks[eventName]) {
        delete this.callbacks[eventName];
      }
    } else {
      this.callbacks = {};
    }
  }
}
```

测试：

```js
const eventBus = new EventBus();

eventBus.on("login", function (name) {
  console.log(`${name}登录了`);
});

eventBus.on("login", function (name) {
  console.log(`${name}又登录了`);
});

eventBus.on("logout", function (name) {
  console.log(`${name}退出登录了`);
});

// eventBus.off('login')
// console.log(eventBus)

eventBus.emit("login", "赤蓝紫");
eventBus.emit("logout", "赤蓝紫");
```

## 4. 自定义发布订阅

```js
// 自定义消息订阅与发布
// PubSub: 包含所有功能的订阅/发布消息的管理者
// PubSub.subscribe(msg, subscriber): 订阅消息: 指定消息名和订阅者回调函数
// PubSub.publish(msg, data): 发布消息: 指定消息名和数据
// PubSub.unsubscribe(flag): 取消订阅: 根据标识取消某个或某些消息的订阅
//  1).没有传值, flag为undefined：清空全部订阅
//  2).传入token字符串: 清除唯一订阅
//  3).msgName字符串: 清除指定消息的全部订阅
class PubSub {
  constructor() {
    this.callbacks = {};
    this.id = 0; // 订阅唯一标识
  }

  subscribe(msg, subscriber) {
    const token = "token_" + ++this.id;

    if (this.callbacks[msg]) {
      // 有过此消息的另一个订阅
      this.callbacks[msg][token] = subscriber;
    } else {
      this.callbacks[msg] = {
        [token]: subscriber,
      };
    }

    return token; // 返回token。用于取消唯一订阅
  }

  publish(msg, data) {
    const callbacksOfmsg = this.callbacks[msg];
    if (callbacksOfmsg) {
      // 有此消息的订阅
      Object.values(callbacksOfmsg).forEach((callback) => {
        callback(data);
      });
    }
  }

  unsubscribe(flag) {
    if (flag === undefined) {
      // 1. 清空全部订阅
      this.callbacks = {};
    } else if (typeof flag === "string") {
      if (flag.indexOf("token") === 0) {
        // 2. 取消唯一订阅
        const callbacks = Object.values(this.callbacks).find((callbacksOfmsg) =>
          callbacksOfmsg.hasOwnProperty(flag)
        ); // 找到flag对应的callbacks：callbacks: {pay: {token_1: f1, token_2: f2}}
        delete callbacks[flag];
      } else {
        // 3. 取消指定消息的全部订阅
        delete this.callbacks[flag];
      }
    } else {
      throw new Error("如果传入参数, 必须是字符串类型");
    }
  }
}
```

测试：

```js
const pubsub = new PubSub();

let pid1 = pubsub.subscribe("pay", (data) => {
  console.log("商家接单: ", data);
});
let pid2 = pubsub.subscribe("pay", () => {
  console.log("骑手接单");
});
let pid3 = pubsub.subscribe("feedback", (data) => {
  console.log(`评价: ${data.title}${data.feedback}`);
});

// console.log(pubsub)

pubsub.publish("pay", {
  title: "炸鸡",
  msg: "预定11:11起送",
});

pubsub.publish("feedback", {
  title: "炸鸡",
  feedback: "还好",
});

console.log("%c%s", "color: blue;font-size: 20px", "取消订阅");

// // 1. 取消全部订阅
// pubsub.unsubscribe()
// console.log(pubsub)

// // 2. 取消唯一订阅
// pubsub.unsubscribe(pid1)
// console.log(pubsub)

// 3. 取消指定消息的订阅
pubsub.unsubscribe("pay");
console.log(pubsub);
```

## 5. 封装 axios

详见：[axios 笔记](https://www.clzczh.top/2022/03/10/axios-1/)

```js
// 封装axios
function axios({ url, method = "GET", params = {}, data = {} }) {
  return new Promise((resolve, reject) => {
    method = method.toUpperCase();

    let queryString = "";
    Object.keys(params).forEach((key) => {
      queryString += `${key}=${params[key]}&`; // 查询参数以key1=value1&key2=value2的形式连接起来
    });

    queryString = queryString.slice(0, -1); // 去掉后面可能出现的&
    url += `?${queryString}`;

    const xhr = new XMLHttpRequest();
    xhr.open(method, url);
    if (method === "GET") {
      xhr.send();
    } else {
      xhr.setRequestHeader("Content-type", "application/json;charset=utf-8");
      xhr.send(JSON.stringify(data)); // 只能发送字符串形式的数据
    }

    xhr.onreadystatechange = function () {
      if (xhr.readyState === 4) {
        const { status } = xhr;
        if (xhr.status >= 200 && xhr.status < 300) {
          const response = {
            status,
            data: JSON.parse(xhr.response),
          };

          resolve(response);
        } else {
          reject(`${status}`);
        }
      }
    };
  });
}

axios.get = (url, options) => {
  return axios(Object.assign(options, { url, method: "GET" })); // 把methods和url合并到options中去
};

axios.post = (url, options) => {
  return axios(Object.assign(options, { url, method: "POST" }));
};

axios.put = (url, options) => {
  return axios(Object.assign(options, { url, method: "PUT" }));
};

axios.delete = (url, options) => {
  return axios(Object.assign(options, { url, method: "DELETE" }));
};
```

测试：

```js
(async function () {
  try {
    const { data: data1 } = await axios({
      url: "https://api.apiopen.top/getJoke",
      method: "get",
      params: {
        a: 10,
        b: 15,
      },
    });
    console.log(data1);

    const { data: data2 } = await axios.post(
      "https://api.apiopen.top/getJoke",
      {
        params: {
          a: 1,
          b: 2,
        },
      }
    );
    console.log(data2);
  } catch (err) {
    console.log(err);
  }
})();
```
