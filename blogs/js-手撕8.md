---
title: js-手撕8
categories: 前端
date: 2022-12-24 12:45:58
tags:
    - JavaScript
    - 手撕
---

# JS手撕(八) Promise

## Promise实现

`Promise`的原理之前有写过两篇博客，就不细讲了。

但还是需要简单复习一下下。

#### Promise构造函数的实现

`promise`的状态一开始是`pending`，只能从`pending`变为`resolved`或从`pending`变为`rejected`。并且是**不可逆**的。

改变`promise`的状态有三种方法，

- 调用 resolve 函数：pending => fulfilled(resolved)
- 调用 reject 函数：pending => rejected
- 抛出异常：pending => rejected

```js
// 1. 初始状态：pending
const p1 = new Promise((resolve, reject) => { });

// 2. 调用resolve函数：pending => fulfilled(resolved)
const p2 = new Promise((resolve, reject) => resolve(123));

// 3. 调用reject函数：pending => rejected
const p3 = new Promise((resolve, reject) => reject('error'));

// 4. 抛出异常：pending => rejected
const p4 = new Promise((resolve, reject) => {
  throw new Error('异常');
});


console.log(p1);
console.log(p2);
console.log(p3);
console.log(p4);
```

从上图可以看出，`promise`对象还会有`PromiseState`和`PromiseResult`属性。

- `PromiseState`：对象的状态

- `PromiseResult`：对象结果值

```js
class MyPromise {
  // Promise对象的状态
  PromiseState = 'pending';

  // Promise对象的结果
  PromiseResult = undefined;

  constructor(executor) { // 执行器函数executor

    // 需要有`resolve`和`reject`函数，因为实例化Promise对象时需要传参`(resolve, reject) => { }`形式的函数
    const resolve = (data) => {
      if (this.PromiseState === 'pending') {
        // 只能从`pending`状态变为`resolved`
        this.PromiseState = 'fulfilled';
        this.PromiseResult = data;
      }
    };

    const reject = (data) => {
      if (this.PromiseState === 'pending') {
        // 只能从`pending`状态变为`rejected`
        this.PromiseState = 'rejected';
        this.PromiseResult = data;
      }
    };

    executor(resolve, reject);
  }
}
```

前3种情况已经能够实现了

```js
// 1. 初始状态：pending
const p1 = new MyPromise((resolve, reject) => { });

// 2. 调用resolve函数：pending => fulfilled(resolved)
const p2 = new MyPromise((resolve, reject) => resolve(123));

// 3. 调用reject函数：pending => rejected
const p3 = new MyPromise((resolve, reject) => reject('error'));

console.log(p1);
console.log(p2);
console.log(p3);
```

但是，抛出异常的情况还不能实现，因为异常没有被捕获，所以会直接报错，后面的代码也不能执行。所以要我们在调用执行器函数`executor`时，应该添加`try catch`，如果被捕获，那就要执行`reject`方法。

```js
try {
  executor(resolve, reject);
} catch (err) {
  reject(err);
}
```

### then方法的实现

首先，简单的实现一下

```js
then(onResolved, onRejected) {
  if (this.PromiseState === "fulfilled") {
    onResolved(this.PromiseResult);
  } else if (this.PromiseState === "rejected") {
    onRejected(this.PromiseResult);
  }
}
```

```js
// 1. 初始状态：pending
const p1 = new MyPromise((resolve, reject) => { });

// 2. 调用resolve函数：pending => fulfilled(resolved)
const p2 = new MyPromise((resolve, reject) => resolve(123));

// 3. 调用reject函数：pending => rejected
const p3 = new MyPromise((resolve, reject) => reject('error'));

// 4. 抛出异常：pending => rejected
const p4 = new MyPromise((resolve, reject) => {
  throw new Error('异常');
});


p1.then((data) => {
  console.log(data);
});

p2.then((data) => {
  console.log(data);
});

p3.then(null, (reason) => {
  console.error(reason);
});

p4.then(null, (reason) => {
  console.error(reason);
});
```

完美(×)，使用`Promise`难免遇到异步任务，所以还需要测试一下。

```js
const p5 = new MyPromise((resolve, reject) => {
  setTimeout(() => {
    resolve('123456');
  }, 0)
})

p5.then((data) => {
  console.log(data);
})
```

没反应。这是因为因为是异步任务，所以执行`then`方法时，状态还是`pending`，所以还需要定义数据结构来存回调函数，如果是异步任务，即状态是`pending`时，存好回调函数。

```js
then(onResolved, onRejected) {
  if (this.PromiseState === "fulfilled") {
    onResolved(this.PromiseResult);
  } else if (this.PromiseState === "rejected") {
    onRejected(this.PromiseResult);
  } else if (this.PromiseState === 'pending') {
    this.resolvedQueue.push(onResolved);
    this.rejectedQueue.push(onRejected);
  }
}
```

存好回调函数，自然还是需要调用的。所以`resolve`函数和`reject`函数最后还需要遍历执行对应回调队列。

```js
const resolve = (data) => {
  if (this.PromiseState === 'pending') {
    // 只能从`pending`状态变为`resolved`
    this.PromiseState = 'fulfilled';
    this.PromiseResult = data;

    this.resolvedQueue.forEach(callback => callback(data));
  }
};
```

`reject`函数同理

这样子就能处理异步任务了。

但是，这个时候`Promise.then()`方法并不是异步任务。

```js
const p5 = new MyPromise((resolve, reject) => {
  resolve('123456');
  console.log(88888);
})

p5.then((res) => {
  console.log(res)
})

console.log(99999);
```

根据JavaScript的执行机制，应该先执行完同步任务，再执行异步任务，并且`Promise.then()`是异步任务(微任务)，所以应该依次输出`88888`、`99999`、`123456`才对。

所以，`then`方法还得改一下，需要添加定时器来让它变成异步任务。

```js
then(onResolved, onRejected) {
  if (this.PromiseState === "fulfilled") {
    setTimeout(() => {
      onResolved(this.PromiseResult);
    })

  } else if (this.PromiseState === "rejected") {
    setTimeout(() => {
      onRejected(this.PromiseResult);
    })

  } else if (this.PromiseState === 'pending') {
    this.resolvedQueue.push(onResolved);
    this.rejectedQueue.push(onRejected);
  }
}
```

小优化：将`then`中的参数变为可选。实际上原生`Promise.then()`方法的参数不传参数或者只传一个参数都不会影响执行。

原理很简单，只需要在`then`方法最前面判断参数是不是函数，不是则把它变成函数即可。

```js
if (typeof onRejected !== "function") {
  onRejected = (reason) => {
    throw reason;
  };
}

if (typeof onResolved !== "function") {
  onResolved = (value) => value;
}
```

`then()`方法返回结果，实现链式调用。原理就是返回一个新的`Promise`对象。当然不能只是返回一个`Promise`对象就行了，还需要判断回调得到的结果是不是`Promise`对象，如果是，还得调用`then()`方法，将状态变为`fulfilled` 或者`rejected`，并变更结果为`then()`方法返回的值。

如：

```js
const p5 = new Promise((resolve, reject) => {
  resolve('123456');
})

const p6 = p5.then(() => {
  return 123567;
});

const p7 = p6.then(() => {
  return new Promise((resolve, reject) => {
    resolve('success');
  })
})

console.log(p6);
console.log(p7);
```

```js
then(onResolved, onRejected) {
  if (typeof onRejected !== "function") {
    onRejected = (reason) => {
      throw reason;
    };
  }

  if (typeof onResolved !== "function") {
    onResolved = (value) => value;
  }

  // 以下部分代码微调
  return new MyPromise((resolve, reject) => {
    if (this.PromiseState === "fulfilled") {
      let result;

      setTimeout(() => {
        result = onResolved(this.PromiseResult);

        // resolvePromise判断回调得到的结果是不是MyPromise，是的话会继续执行then方法
        resolvePromise(result, resolve, reject);
      })

    } else if (this.PromiseState === "rejected") {
      setTimeout(() => {
        result = onRejected(this.PromiseResult);
        resolvePromise(result, resolve, reject);
      })

    } else if (this.PromiseState === 'pending') {
      this.resolvedQueue.push((() => {
        resolvePromise(onResolved(this.PromiseResult), resolve, reject)
      }));
      this.rejectedQueue.push(() => {
        resolvePromise(onResolved(this.PromiseResult), resolve, reject)
      });
    }
  })
}
```

resolvePromise

```js
function resolvePromise(x, resolve, reject) {
  if (x instanceof MyPromise) {

    // 如果是`MyPromise`对象，则需要调用thrn方法，将状态变为 fulfilled 或者 rejecte
    x.then(resolve, reject);
  } else {
    // 普通值直接resolve就行
    resolve(x);
  }
}
```

测试：

```js
const p5 = new MyPromise((resolve, reject) => {
  resolve('123456');
})

const p6 = p5.then(() => {
  return 123567;
});

const p7 = p6.then(() => {
  return new MyPromise((resolve, reject) => {
    resolve('success');
  })
})

console.log(p6);
console.log(p7);
```

### catch方法的实现

在搞完上面的`then`方法后，`catch`方法就迎刃而解了。因为`catch`实际上就是一个语法糖，我们也可以用`then`方法来实现，只需要不传第一个参数，只传第二个参数即可。

```js
catch(onRejected) {
  return this.then(undefined, onRejected);
}
```

测试：

```js
new MyPromise((resolve, reject) => {
  reject('error');
})
  .catch((reason) => {
    console.error(reason);
  })
```

## 完整代码

```js
class MyPromise {
  constructor(executor) { // 执行器函数executor
    // Promise对象的状态
    this.PromiseState = 'pending';

    // Promise对象的结果
    this.PromiseResult = undefined;

    this.resolvedQueue = [];
    this.rejectedQueue = [];

    // 需要有`resolve`和`reject`函数，因为实例化Promise对象时需要传参`(resolve, reject) => { }`形式的函数
    const resolve = (data) => {
      if (this.PromiseState === 'pending') {
        // 只能从`pending`状态变为`resolved`
        this.PromiseState = 'fulfilled';
        this.PromiseResult = data;

        this.resolvedQueue.forEach(callback => callback(data));
      }
    };

    const reject = (data) => {
      if (this.PromiseState === 'pending') {
        // 只能从`pending`状态变为`rejected`
        this.PromiseState = 'rejected';
        this.PromiseResult = data;

        this.rejectedQueue.forEach(callback => callback(data));
      }
    };

    try {
      executor(resolve, reject);
    } catch (err) {
      reject(err);
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

    return new MyPromise((resolve, reject) => {
      if (this.PromiseState === "fulfilled") {
        let result;

        setTimeout(() => {
          result = onResolved(this.PromiseResult);
          resolvePromise(result, resolve, reject);
        })

      } else if (this.PromiseState === "rejected") {
        setTimeout(() => {
          result = onRejected(this.PromiseResult);
          resolvePromise(result, resolve, reject);
        })

      } else if (this.PromiseState === 'pending') {
        this.resolvedQueue.push((() => {
          resolvePromise(onResolved(this.PromiseResult), resolve, reject)
        }));
        this.rejectedQueue.push(() => {
          resolvePromise(onResolved(this.PromiseResult), resolve, reject)
        });
      }
    })
  }

  catch(onRejected) {
    return this.then(undefined, onRejected);
  }
}

function resolvePromise(x, resolve, reject) {
  if (x instanceof MyPromise) {

    // 如果是`MyPromise`对象，则需要调用then方法，将状态变为 fulfilled 或者 rejecte
    x.then(resolve, reject);
  } else {
    // 普通值直接resolve就行
    resolve(x);
  }
}
```

## 参考

- [Promise学习笔记(二) | 赤蓝紫](https://www.clzczh.top/2022/03/05/promise-2)

- [从一道让我失眠的 Promise 面试题开始，深入分析 Promise 实现细节 - 掘金](https://juejin.cn/post/6945319439772434469)
