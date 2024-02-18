---
title: Promise学习笔记(二)
date: 2022-03-05 11:29:53
categories: 前端
tags:
  - Promise
  - JavaScript
---

# Promise 学习笔记(二)

## 1. 改变 promise 的状态

promise 的状态一开始是 pending，改变 promise 的状态有三个方法：

- 调用 resolve 函数：pending => fulfilled(resolved)
- 调用 reject 函数：pending => rejected
- 抛出异常：pending => rejected

```js
// 0. 初始状态：pending
const p0 = new Promise((resolve, reject) => {});
console.log(p0);

// 1. 调用resolve函数：pending => fulfilled(resolved)
const p1 = new Promise((resolve, reject) => {
  resolve("ok");
});
console.log(p1);

// 2. 调用reject函数：pending => rejected
const p2 = new Promise((resolve, reject) => {
  reject("error");
});
console.log(p2);
p2.catch((reason) => {
  console.log(reason);
});

// 3. 抛出异常：pending => rejected
const p3 = new Promise((resolve, reject) => {
  throw "抛出异常";
});
console.log(p3);
p3.catch((reason) => {
  console.log(reason);
});
```

![image-20220226173041998](https://s2.loli.net/2022/03/05/4k1fOsAVoqe3Sbi.png)

## 2. 指定多个回调

**当 promise 改变为对应状态时都会调用**

```js
const p = Promise.resolve("ok");
p.then((value) => {
  console.log("指定的第一个回调函数");
});

p.then((value) => {
  console.log("指定的第二个回调函数");
  console.log("%c---------------", "color: red; font-size: 24px");
});

const p1 = Promise.reject("error");
p1.then((value) => {
  console.log("指定的第一个回调函数"); // 失败的结果，不会调用这个成功时才调用的函数
});

p1.catch((reason) => {
  console.log("指定的第二个回调函数");
});
```

![image-20220226174300677](https://s2.loli.net/2022/03/05/nAWPjJdEcqt92si.png)

## 3. then 方法返回结果

then()方法返回新的 Promise 对象

```js
const p1 = Promise.resolve("ok");

// 1. 抛出异常
let result1 = p1.then(
  (value) => {
    throw "抛出异常";
  },
  (reason) => {
    console.warn(reason);
  }
);
console.log(result1); // [[PromiseState]]: "rejected"

// 2. 非Promise类型的对象
const result2 = p1.then((value) => {
  return "Hello, CLZ!"; // [[PromiseState]]: "fulfilled"
});
console.log(result2);

// 3. Promise类型的对象
// 3.1 结果成功
const result3 = p1.then((value) => {
  return new Promise((resolve, reject) => {
    resolve("success");
  });
});
console.log(result3);

// 3.2 结果失败
const result4 = p1.then((value) => {
  return new Promise((resolve, reject) => {
    reject("error");
  });
});
console.log(result4);
```

![image-20220226203535240](https://s2.loli.net/2022/03/05/ax9SIO7D6gKcNvj.png)

## 4. 串联多个任务

**链式调用**原理：then()返回的是 Promise 对象

```js
const p1 = Promise.resolve("ok");

p1.then((value) => {
  return new Promise((resolve, reject) => {
    resolve("success");
  });
})
  .then((value) => {
    console.log(value); // success
    return value;
  })
  .then((value) => {
    console.log(value); // success
    return value;
  });
```

## 5. 异常穿透

- 当使用 Promise 的 then 链式调用时，可以在最后指定失败的回调
- 前面任何操作出异常都会传给最后失败的回调处理

```js
const p1 = Promise.resolve("ok");

p1.then((value) => {
  return new Promise((resolve, reject) => {
    resolve("success");
  });
})
  .then((value) => {
    console.log(value);
    throw "出现异常";
    return value;
  })
  .then((value) => {
    console.log(value);
    return value;
  })
  .catch((reason) => {
    console.warn(reason);
  });
```

## 6. 中断 Promise 链

<b style="color: red">有且只有一个方式：返回一个 pending 状态的 Promise 对象</b>

因为返回一个 pending 状态的对象时，后续的回调就不能执行了，因为后面的回调函数只有在状态发生变化时才能执行。

```js
const p1 = Promise.resolve("ok");

p1.then((value) => {
  return new Promise((resolve, reject) => {
    resolve("success");
  });
})
  .then((value) => {
    console.log(1);
  })
  .then((value) => {
    console.log(2);
    return new Promise(() => {}); // 只输出1, 2
    // return false    // 无效果
  })
  .then((value) => {
    console.log(3);
  })
  .catch((reason) => {
    console.warn(reason);
  });
```

## 7. Promise 自定义封装

### 7.1 基本结构的搭建

promise.js

```js
function Promise(executor) {}

Promise.prototype.then = function (onResolved, onRejected) {};
```

测试用: index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>手写Promise</title>
    <script src="./promise.js"></script>
  </head>

  <body>
    <script>
      let p = new Promise((resolve, reject) => {
        resolve("OK");
      });

      p.then(
        (value) => {
          console.log(value);
        },
        (reason) => {
          console.warn(reason);
        }
      );
    </script>
  </body>
</html>
```

### 7.2 resolve、reject 功能实现

```js
// resolve函数
const resolve = (data) => {
  // 1. 修改对象的状态(promiseState)
  this.PromiseState = "fulfilled";

  // 2. 设置对象结果值(promiseResult)
  this.PromiseResult = data;
};

// reject函数
const reject = (data) => {
  // 1. 修改对象的状态(promiseState)
  this.PromiseState = "rejected";

  // 2. 设置对象结果值(promiseResult)
  this.PromiseResult = data;
};
```

promise.js

```js
function Promise(executor) {
  this.PromiseState = "pending";
  this.PromiseResult = null;

  // resolve函数
  const resolve = (data) => {
    // 1. 修改对象的状态(promiseState)
    this.PromiseState = "fulfilled";

    // 2. 设置对象结果值(promiseResult)
    this.PromiseResult = data;
  };

  // reject函数
  const reject = (data) => {
    // 1. 修改对象的状态(promiseState)
    this.PromiseState = "rejected";

    // 2. 设置对象结果值(promiseResult)
    this.PromiseResult = data;
  };

  // 同步调用执行器函数executor
  executor(resolve, reject);
}

Promise.prototype.then = function (onResolved, onRejected) {};
```

### 7.3 实现抛出异常改变 promise 状态

```js
// 实现抛出异常改变promise状态
try {
  // 同步调用执行器函数executor
  executor(resolve, reject);
} catch (e) {
  reject(e);
}
```

### 7.4 实现 Promise 状态只能修改一次

`pending => fulfilled(resolved)`或`pending => rejected`

只需要在 resolve 函数和 reject 函数最开始加上判断，Promise 的状态是不是 pending 就行，如果不是，直接 return

```js
if (this.PromiseState !== "pending") {
  return;
}
```

### 7.5 实现 then 方法

```js
Promise.prototype.then = function (onResolved, onRejected) {
  if (this.PromiseState === "fulfilled") {
    onResolved(this.PromiseResult);
  } else if (this.PromiseState === "rejected") {
    onRejected(this.PromiseResult);
  }
};
```

### 7.6 实现异步任务的执行

当任务是异步任务时，则无法执行成功或失败的回调函数，因为执行 then 参数时，状态是 pending。

```js
let p = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("OK");
  }, 0);
});
```

实现方法：

- 自定义 Promise 对象增加一个` callback`对象，用于存成功、失败的回调函数

  ```js
  this.callback = {};
  ```

- ` then`函数增加状态为` pending`的判断，这时候只是把回调函数存放起来，等到实际调用` resolve`或` reject`函数时才调用存起来的回调函数

  ```js
  if (this.PromiseState === "fulfilled") {
    onResolved(this.PromiseResult);
  } else if (this.PromiseState === "rejected") {
    onRejected(this.PromiseResult);
  } else {
    // 保存回调函数
    this.callback = {
      onResolved,
      onRejected,
    };
  }
  ```

- 调用` resolve`或` reject`函数时，判断是否需要调用存起来的回调函数

  ```js
  if (this.callback.onResolved) {
    // ` resolve`或` reject`函数最后
    this.callback.onResolved(data); // reject函数对应修改为onRejected
  }
  ```

### 7.7 指定多个回调函数

```js
let p = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("OK");
  }, 1000);
});

p.then(
  (value) => {
    console.log(value);
  },
  (reason) => {
    console.warn(reason);
  }
);

p.then(
  (value) => {
    alert(value);
  },
  (reason) => {
    alert(reason);
  }
);
```

**会发现，最后只会弹出 OK，而不会在控制台输出。**只是因为前面保存的 callback 属性是对象，所以后面的会覆盖掉前面的，改进只需要把 callback 变为数组即可

```js
this.callbacks = [];
```

```js
Promise.prototype.then = function (onResolved, onRejected) {
  if (this.PromiseState === "fulfilled") {
    onResolved(this.PromiseResult);
  } else if (this.PromiseState === "rejected") {
    onRejected(this.PromiseResult);
  } else {
    this.callbacks.push({
      onResolved,
      onRejected,
    });
  }
};
```

```js
this.callbacks.forEach((item) => {
  // ` resolve`或` reject`函数最后
  item.onResolved(data); // reject函数对应修改为onRejected
});
```

### 7.8 then 方法返回结果

成功时的代码修改如下图，失败的类似

**只实现同步**

```js
return new Promise((resolve, reject) => {
  if (this.PromiseState === "fulfilled") {
    try {
      // 获取回调函数的执行结果
      const result = onResolved(this.PromiseResult);

      if (result instanceof Promise) {
        // 结果是Promise类型对象
        result.then(
          (v) => {
            resolve(v);
          },
          (r) => {
            reject(r);
          }
        );
      } else {
        resolve(result);
      }
    } catch (e) {
      reject(e);
    }
  }
  // else if (this.PromiseState === 'rejected') {
  //   onRejected(this.PromiseResult)
  // } else {
  //   this.callbacks.push({
  //     onResolved,
  //     onRejected
  //   })
  // }
});
```

**实现异步**

```js
const self = this;

this.callbacks.push({
  onResolved: function () {
    try {
      let result = onResolved(self.PromiseResult);
      if (result instanceof Promise) {
        result.then(
          (v) => {
            resolve(v);
          },
          (r) => {
            reject(r);
          }
        );
      } else {
        resolve(result);
      }
    } catch (e) {
      reject(e);
    }
  },
  onRejected: function () {
    // 类似上面
  },
});
```

### 7.9 then 方法封装优化

```js
function callback(type) {
  try {
    // 获取回调函数的执行结果
    const result = type(self.PromiseResult);

    if (result instanceof Promise) {
      // 结果是Promise类型对象
      result.then(
        (v) => {
          resolve(v);
        },
        (r) => {
          reject(r);
        }
      );
    } else {
      resolve(result);
    }
  } catch (e) {
    reject(e);
  }
}
```

```js
Promise.prototype.then = function (onResolved, onRejected) {
  return new Promise((resolve, reject) => {
    const self = this;

    function callback(type) {
      try {
        // 获取回调函数的执行结果
        const result = type(self.PromiseResult);

        if (result instanceof Promise) {
          // 结果是Promise类型对象
          result.then(
            (v) => {
              resolve(v);
            },
            (r) => {
              reject(r);
            }
          );
        } else {
          resolve(result);
        }
      } catch (e) {
        reject(e);
      }
    }

    if (this.PromiseState === "fulfilled") {
      callback(onResolved);
    } else if (this.PromiseState === "rejected") {
      callback(onRejected);
    } else {
      this.callbacks.push({
        onResolved: function () {
          callback(onResolved);
        },
        onRejected: function () {
          callback(onRejected);
        },
      });
    }
  });
};
```

### 7.10 catch 方法

**异常穿透**：因为 catch 方法支持只传一个参数` reason => {}`，而 then 方法第二个参数才是` reason => {}`，所以会出问题：最后调用 catch 时没有 onRejected 方法

所以，需要在 then 方法开始的时候添加以下判断

```js
if (typeof onRejected !== "function") {
  onRejected = (reason) => {
    throw reason;
  };
}
```

catch 方法还应实现值传递，即 then 可以不传参数，那么和上面的原理一样，还需要添加当 onResolved 不存在的情况

```js
if (typeof onResolved !== "function") {
  onResolved = (value) => value;
}
```

测试用:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>手写Promise</title>
    <script src="./promise.js"></script>
  </head>

  <body>
    <script>
      let p = new Promise((resolve, reject) => {
        setTimeout(() => {
          resolve("ok");
        }, 1000);
      });

      p.then()
        .then((value) => {
          console.log(222);
        })
        .then((value) => {
          console.log(333);
        })
        .catch((reason) => {
          console.warn(reason);
        });
    </script>
  </body>
</html>
```

### 7.11 API

#### 7.11.1 resolve 方法和 reject 方法

```js
// resolve方法
Promise.resolve = function (value) {
  return new Promise((resolve, reject) => {
    if (value instanceof Promise) {
      value.then(
        (v) => {
          resolve(v);
        },
        (r) => {
          reject(r);
        }
      );
    } else {
      resolve(value);
    }
  });
};

// reject方法
Promise.reject = function (reason) {
  return new Promise((resolve, reject) => {
    reject(reason);
  });
};
```

#### 7.11.2 all 方法

```js
// all方法
Promise.all = function (promises) {
  return new Promise((resolve, reject) => {
    let arr = [];
    for (let i = 0; i < promises.length; i++) {
      promises[i].then(
        (v) => {
          arr[i] = v;
          if (arr.length === promises.length) {
            resolve(arr);
          }
        },
        (r) => {
          reject(r);
        }
      );
    }
  });
};
```

测试：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>手写Promise</title>
    <script src="./promise.js"></script>
  </head>

  <body>
    <script>
      const p1 = new Promise((resolve, reject) => {
        resolve("p1: OK");
      });
      const p2 = Promise.resolve("p2: OK");
      const p3 = Promise.resolve("p3: OK");
      const result1 = Promise.all([p1, p2, p3]); // 所有Promise的结果都成功
      console.log(result1);

      const p4 = Promise.resolve("p4: OK");
      const p5 = Promise.reject("p5: Err");
      const p6 = Promise.reject("p6: Err");
      const result2 = Promise.all([p4, p5, p6]); // 有Promise的结果失败
      console.log(result2);
    </script>
  </body>
</html>
```

![image-20220227171844700](https://s2.loli.net/2022/03/05/n7BaoepVFNtulUS.png)

#### 7.11.3 race 方法

```js
// race方法
Promise.race = function (promises) {
  return new Promise((resolve, reject) => {
    for (let i = 0; i < promises.length; i++) {
      promises[i].then(
        (v) => {
          resolve(v);
        },
        (r) => {
          reject(r);
        }
      );
    }
  });
};
```

### 7.12 then 方法回调是异步执行

```js
// 1 3 2：then方法异步执行，得等同步方法执行完才执行
const p1 = new Promise((resolve, reject) => {
  resolve("ok");
  console.log(1);
});

p1.then((value) => {
  console.log(2);
});

console.log(3);
```

**自定义 Promise 实现 then 方法回调异步执行**：就是把原本调用成功、失败时的回调函数变为异步的就行了

如

```js
if (this.PromiseState === "fulfilled") {
  setTimeout(() => {
    callback(onResolved);
  });
}
```

![image-20220227175150352](https://s2.loli.net/2022/03/05/JWv3BpfiebkuGth.png)

### 7.13. 完整代码

```js
function Promise(executor) {
  this.PromiseState = "pending";
  this.PromiseResult = null;
  this.callbacks = [];

  // resolve函数
  const resolve = (data) => {
    if (this.PromiseState !== "pending") {
      return;
    }

    // 1. 修改对象的状态(promiseState)
    this.PromiseState = "fulfilled";

    // 2. 设置对象结果值(promiseResult)
    this.PromiseResult = data;

    setTimeout(() => {
      this.callbacks.forEach((item) => {
        item.onResolved(data);
      });
    });
  };

  // reject函数
  const reject = (data) => {
    if (this.PromiseState !== "pending") {
      return;
    }

    // 1. 修改对象的状态(promiseState)
    this.PromiseState = "rejected";

    // 2. 设置对象结果值(promiseResult)
    this.PromiseResult = data;

    setTimeout(() => {
      this.callbacks.forEach((item) => {
        item.onRejected(data);
      });
    });
  };

  // 实现抛出异常改变promise状态
  try {
    // 同步调用执行器函数executor
    executor(resolve, reject);
  } catch (e) {
    reject(e);
  }
}

// then方法
Promise.prototype.then = function (onResolved, onRejected) {
  if (typeof onRejected !== "function") {
    onRejected = (reason) => {
      throw reason;
    };
  }

  if (typeof onResolved !== "function") {
    onResolved = (value) => value;
  }

  return new Promise((resolve, reject) => {
    const self = this;

    function callback(type) {
      try {
        // 获取回调函数的执行结果
        const result = type(self.PromiseResult);

        if (result instanceof Promise) {
          // 结果是Promise类型对象
          result.then(
            (v) => {
              resolve(v);
            },
            (r) => {
              reject(r);
            }
          );
        } else {
          resolve(result);
        }
      } catch (e) {
        reject(e);
      }
    }

    if (this.PromiseState === "fulfilled") {
      setTimeout(() => {
        callback(onResolved);
      });
    } else if (this.PromiseState === "rejected") {
      setTimeout(() => {
        callback(onRejected);
      });
    } else {
      this.callbacks.push({
        onResolved: function () {
          callback(onResolved);
        },
        onRejected: function () {
          callback(onRejected);
        },
      });
    }
  });
};

// catch方法
Promise.prototype.catch = function (onRejected) {
  return this.then(undefined, onRejected);
};

// resolve方法
Promise.resolve = function (value) {
  return new Promise((resolve, reject) => {
    if (value instanceof Promise) {
      value.then(
        (v) => {
          resolve(v);
        },
        (r) => {
          reject(r);
        }
      );
    } else {
      resolve(value);
    }
  });
};

// reject方法
Promise.reject = function (reason) {
  return new Promise((resolve, reject) => {
    reject(reason);
  });
};

// all方法
Promise.all = function (promises) {
  return new Promise((resolve, reject) => {
    let arr = [];
    for (let i = 0; i < promises.length; i++) {
      promises[i].then(
        (v) => {
          arr[i] = v;
          if (arr.length === promises.length) {
            resolve(arr);
          }
        },
        (r) => {
          reject(r);
        }
      );
    }
  });
};

// race方法
Promise.race = function (promises) {
  return new Promise((resolve, reject) => {
    for (let i = 0; i < promises.length; i++) {
      promises[i].then(
        (v) => {
          resolve(v);
        },
        (r) => {
          reject(r);
        }
      );
    }
  });
};
```

封装成类版本：

```js
class Promise {
  constructor(executor) {
    this.PromiseState = "pending";
    this.PromiseResult = null;
    this.callbacks = [];

    // resolve函数
    const resolve = (data) => {
      if (this.PromiseState !== "pending") {
        return;
      }

      // 1. 修改对象的状态(promiseState)
      this.PromiseState = "fulfilled";

      // 2. 设置对象结果值(promiseResult)
      this.PromiseResult = data;

      setTimeout(() => {
        this.callbacks.forEach((item) => {
          item.onResolved(data);
        });
      });
    };

    // reject函数
    const reject = (data) => {
      if (this.PromiseState !== "pending") {
        return;
      }

      // 1. 修改对象的状态(promiseState)
      this.PromiseState = "rejected";

      // 2. 设置对象结果值(promiseResult)
      this.PromiseResult = data;

      setTimeout(() => {
        this.callbacks.forEach((item) => {
          item.onRejected(data);
        });
      });
    };

    // 实现抛出异常改变promise状态
    try {
      // 同步调用执行器函数executor
      executor(resolve, reject);
    } catch (e) {
      reject(e);
    }
  }

  then(onResolved, onRejected) {
    if (typeof onRejected !== "function") {
      onRejected = (reason) => {
        throw reason;
      };
    }

    if (typeof onResolved !== "function") {
      onResolved = (value) => value;
    }

    return new Promise((resolve, reject) => {
      const self = this;

      function callback(type) {
        try {
          // 获取回调函数的执行结果
          const result = type(self.PromiseResult);

          if (result instanceof Promise) {
            // 结果是Promise类型对象
            result.then(
              (v) => {
                resolve(v);
              },
              (r) => {
                reject(r);
              }
            );
          } else {
            resolve(result);
          }
        } catch (e) {
          reject(e);
        }
      }

      if (this.PromiseState === "fulfilled") {
        setTimeout(() => {
          callback(onResolved);
        });
      } else if (this.PromiseState === "rejected") {
        setTimeout(() => {
          callback(onRejected);
        });
      } else {
        this.callbacks.push({
          onResolved: function () {
            callback(onResolved);
          },
          onRejected: function () {
            callback(onRejected);
          },
        });
      }
    });
  }

  catch(onRejected) {
    return this.then(undefined, onRejected);
  }

  // resolve方法(静态方法，属于类而不是属于实例对象)
  static resolve(value) {
    return new Promise((resolve, reject) => {
      if (value instanceof Promise) {
        value.then(
          (v) => {
            resolve(v);
          },
          (r) => {
            reject(r);
          }
        );
      } else {
        resolve(value);
      }
    });
  }

  // reject方法
  static reject(reason) {
    return new Promise((resolve, reject) => {
      reject(reason);
    });
  }

  // all方法
  static all(promises) {
    return new Promise((resolve, reject) => {
      let arr = [];
      for (let i = 0; i < promises.length; i++) {
        promises[i].then(
          (v) => {
            arr[i] = v;
            if (arr.length === promises.length) {
              resolve(arr);
            }
          },
          (r) => {
            reject(r);
          }
        );
      }
    });
  }

  // race方法
  static race(promises) {
    return new Promise((resolve, reject) => {
      for (let i = 0; i < promises.length; i++) {
        promises[i].then(
          (v) => {
            resolve(v);
          },
          (r) => {
            reject(r);
          }
        );
      }
    });
  }
}
```

## 8. async 和 await

### 8.1 async 函数

- 函数的返回值是 Promise 对象
- Promise 对象的结果由 async 函数执行的返回值决定

和 then 方法一样

```js
// 1. 返回值为一个非Promise类型的数据
async function test1() {
  return 123;
}
let result1 = test1();
console.log(result1);

// 2. 返回值为一个Promise对象，结果为成功
async function test2() {
  return new Promise((resolve, reject) => {
    resolve("ok");
  });
}
let result2 = test2();
console.log(result2);

// 3. 返回值为一个Promise对象，结果为失败
async function test3() {
  return new Promise((resolve, reject) => {
    reject("error");
  });
}
let result3 = test3();
console.log(result3);

// 4. 抛出异常
async function test4() {
  throw "抛出异常";
}
let result4 = test4();
console.log(result4);
```

![image-20220227192621644](https://s2.loli.net/2022/03/05/TqvdYSm5NwipCaf.png)

### 8.2 await 表达式

- await 右侧的表达式一般是 Promise 对象，但也可以是其他值
- 如果右侧的表达式是 promise 对象，await 返回的是 promise 成功的值
- 如果右侧的表达式是其他值，await 返回的就是该值

注意：

- <b style="color: red">await 必须写在 async 函数中，但 async 函数中不一定要 await</b>
- 如果 await 的 promise 失败，则会抛出异常，所以需要通过` try...catch`捕捉处理

```js
let test = async () => {
  // 1. await右侧为其他类型的情况
  let result = await 123;
  console.log(result); // 123

  // 2. await右侧为Promise的情况
  // 2.1 成功
  result = await new Promise((resolve, reject) => {
    resolve("ok");
  });
  console.log(result); // ok

  // 2.2 失败
  try {
    result = await new Promise((resolve, reject) => {
      reject("error");
    });
    console.log(result); // ok
  } catch (e) {
    console.log(e);
  }
};

test();
```

### 8.3 async 和 await 使用

#### 8.3.1 读取文件

情景：读取 resource 文件夹下的数据(1.txt，2.txt，3.txt)，分别是 111、222、333。并把数据拼接后在控制台打印出来。

**回调函数版本**：

```js
const fs = require("fs");

fs.readFile("./resource/1.txt", (err, data1) => {
  if (err) {
    throw err;
  }
  fs.readFile("./resource/2.txt", (err, data2) => {
    if (err) {
      throw err;
    }
    fs.readFile("./resource/3.txt", (err, data3) => {
      console.log(data1 + data2 + data3);
    });
  });
});
```

![image-20220227194850320](https://s2.loli.net/2022/03/05/2t65LCcdgmM7lwE.png)

**async+await 版本**：

```js
const fs = require("fs");
const { promisify } = require("util");

const readFile = promisify(fs.readFile);

async function myReadFile() {
  try {
    let data1 = await readFile("./resource/1.txt");
    let data2 = await readFile("./resource/2.txt");
    let data3 = await readFile("./resource/3.txt");

    console.log(data1 + data2 + data3);
  } catch (e) {
    console.log(e);
  }
}

myReadFile();
```

#### 8.3.2 axios 使用

安装：` npm install axios`

```js
(async function () {
  const axios = require("axios");

  const { data } = await axios.get("https://qcgx2i.api.cloudendpoint.cn/hello");
  console.log(data);
})();
```

![image-20220227195827006](https://s2.loli.net/2022/03/05/5Q6GFRT9nSvuPoI.png)
