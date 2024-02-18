---
title: MutationObserver接口-2-观察范围
categories: 前端
date: 2022-07-05 20:38:16
tags:
    - JavaScript
---



# MutationObserver接口(二)    观察范围

## 观察范围

上一节，我们使用`MutationObserver`时，都只是观察节点的属性。但是实际上并不仅仅是只能观察节点的属性，还可以观察子节点、子树等。只需要调用`observe()`方法时，第二个参数添加对应配置即可。

| 属性                      | 说明                                                                            |
| ----------------------- | ----------------------------------------------------------------------------- |
| `attributes`            | 布尔值，表示观察目标节点的属性变化                                                             |
| `attributeFilter`       | 字符串数组，表示要观察哪些属性的变化。(类似白名单，只有白名单的才会被观察)                                        |
| `attributeOldValue`     | 布尔值，表示`MutationRecord`是否记录变化之间的数据。**设置该属性为`true`，会将`attributes`的值转换为`true`**。 |
| `characterData`         | 布尔值，表示观察文本节点。                                                                 |
| `characterDataOldValue` | 布尔值，表示`MutationRecord`是否记录变化之间的数据。和`attributeOldValue`一样，对应`characterData`    |
| `childList`             | 布尔值，表示观察子节点                                                                   |
| `subtree`               | 布尔值。表示观察目标节点及其子树。如果为`false`，则之观察目标节点的变化，为`true`                               |

### 观察属性

观察属性就是上一节一直在用的。

```js
const observer = new MutationObserver((mutationsRecords) => {
  console.log(mutationsRecords)
})

observer.observe(document.body, {
  attributes: true
})

document.body.setAttribute('name', 'clz')
```

如果我们不需要观察所有属性，而只是观察某个或某几个属性，可以使用` attributeFilter`属性来设置白名单，值是一个属性名数组。

```js
let observer = new MutationObserver((mutationRecords) => {
  console.log(mutationRecords)
})


observer.observe(document.body, {
  attributeFilter: ['name', 'age']
})


document.body.setAttribute('name', 'clz')
document.body.setAttribute('age', 21)
document.body.setAttribute('job', 'FontEnd-Coder')
```

![image-20220619130352108](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8450e0031dd64dbe948e0b5ad5adb186~tplv-k3u1fbpfcp-zoom-1.image)

上面设置了`name`和`age`为白名单，即只观察`name`和`age`属性，所以后面设置`job`属性不会触发回调。

从上图，我们可以看到一个oldValue属性，它就是用来保存属性原来的值的。而默认是不会保存属性原来的值的，如果想要记录原来的值，可以将` attributeOldValue`属性设置为` true`。**设置该属性为`true`，会将`attributes`的值转换为`true`**。

```js
const observer = new MutationObserver((mutationRecords) => {
  mutationRecords.map(mutationRecord => console.log(mutationRecord.oldValue))
})


observer.observe(document.body, {
  attributeOldValue: true
})

document.body.setAttribute('name', 'clz')
document.body.setAttribute('name', 'czh')
```

![OrufED.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d840d01903b5422d846243726bf78b31~tplv-k3u1fbpfcp-zoom-1.image)

设置`name`属性为`clz`的时候打印原来的值，原来没有值，所以打印`null`，设置为`czh`的时候打印原来的值`czh`。

### 观察文本节点

`MutationObserver`可以观察文本节点。

```js
const observer = new MutationObserver((mutationRecords) => {
  console.log(mutationRecords)
})

document.body.firstChild.textContent = 'hello'

observer.observe(document.body.firstChild, {
  characterData: true
})

document.body.firstChild.textContent = '123'
document.body.firstChild.textContent = '456'
document.body.firstChild.textContent = '789'
```

如果想要记录原来的值，可以将` characterDataOldValue`属性设置为` true`。**设置该属性为`true`，会将`characterData`的值转换为`true`**。

```js
const observer = new MutationObserver((mutationRecords) => {
  mutationRecords.map(mutationRecord => console.log(mutationRecord.oldValue))
})


document.body.firstChild.textContent = 'clz'

observer.observe(document.body.firstChild, {
  characterDataOldValue: true
})

document.body.firstChild.textContent = '123'
document.body.firstChild.textContent = '456'
document.body.firstChild.textContent = '789'
```

![image-20220619131126781](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39363b6e05f24eec8441139b9f2840f6~tplv-k3u1fbpfcp-zoom-1.image)

注意：`innerText`和`textContent`有点点相似，但是`innerText`可能会引发一些问题。

首先，`innerText`是**元素节点**的属性，表示一个节点及其后代的“渲染”文本内容。而`textContent`是**节点**的属性，表示节点的一个节点及其后代的文本内容。

举个小例子，说明他们两的区别。

```html
<body>
    <div>
        <span>
            123
        </span>
        <span style="display:none">
            456
        </span>
    </div>
    <script>
        const div = document.querySelector('div')

        console.log(div.innerText)
        console.log(div.textContent)

        console.log('%c%s', 'color:red;font-size:24px', '============')

        const divChild = div.firstChild

        console.log(divChild.textContent)
        console.log(divChild.innerText)

        divChild.textContent = '456'  // 会在span节点前添加上456
        // divChild.innerText = '456'   // 没有效果,因为文本节点没有innerText属性
    </script>
</body>
```

差异：

1. `innerText`属性不会获取`display`为`none`的隐藏元素，而`textContent`会获取。
2. `innerText`没有格式，而`textContent`有格式
3. 文本节点没有`innerText`属性

从上面可以看到，`innerText`属性不会获取`display`为`none`的隐藏元素，而`textContent`会获取。也就是说，`innetText`属性值的获取会触发回流，因为它需要考虑到CSS样式(如`display`)，而`textContent`只是单纯读取文本内容，所以不会发生回流。

当我们观察节点时修改的是`innerText`，而不是`textContent`的话，会引发不一样的情况(个人认为算bug了，如果有了解原因的小伙伴，可以评论交流)

另外红宝书不建议使用`innerText`，但是，**明知山有虎，偏向虎山行**。(了解使用后会有什么隐患)

```js
const observer = new MutationObserver(
  (mutationRecords) => mutationRecords.map((x) => console.log(x.oldValue))
);

document.body.innerText = 'clz'

observer.observe(document.body.firstChild, { characterDataOldValue: true });

document.body.firstChild.textContent = '123'
document.body.firstChild.textContent = '789'
```

1. 观察前设置的`innerText`值也能被观察到

2. `oldValue`不再是旧值，而是设置的新值

上面开始观察后，使用的是`textContent`，因为使用`innerText`又会导致另一个bug发生。

```js
const observer = new MutationObserver(
  (mutationRecords) => mutationRecords.map((x) => console.log(x.oldValue))
);

document.body.innerText = 'clz'

observer.observe(document.body.firstChild, { characterDataOldValue: true });

document.body.firstChild.textContent = '123'
document.body.innerText = '456'
document.body.firstChild.textContent = '789'
```

3. 开始观察后，修改`innerText`属性会导致观察失效。包括开始观察后`innerText`之前和之后的。

即使不混用，也还是有问题。

```js
const observer = new MutationObserver(
  (mutationRecords) => mutationRecords.map((x) => console.log(x.oldValue))
);

document.body.innerText = 'clz'

observer.observe(document.body.firstChild, { characterDataOldValue: true });

document.body.innerText = '123'
document.body.innerText = '456'
document.body.innerText = '789'
```

上面的代码不会打印任何东西。所以**尽可能不要使用`innerText`，而是使用`textContent`**。

### 观察子节点

` MutationObserver`还可以观察目标节点子节点的添加和移除，只需要将`childList`属性设置为`true`即可。

```html
<body>
    <div id="box"></div>
    <script>
        const box = document.getElementById('box')

        const observer = new MutationObserver(
            (mutationRecords) => console.log(mutationRecords)
        );

        observer.observe(box, { childList: true })

        box.appendChild(document.createElement('span'))    // 在MutationRecord的addedNodes属性中可以查看到添加的节点

        box.innerHTML = '<div></div>'     // 使用innetHTML还会移除节点，表现为removedNodes中有被移除的节点
    </script>
</body>
```

![image-20220619134510262](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2c90758a66db41e6867992731307687b~tplv-k3u1fbpfcp-zoom-1.image)

交换子节点顺序会导致发生两次变化，因为交换子节点顺序实际上有两个步骤，第一次是节点被移除，第二次是节点被添加。

```html
<body>
    <div id="box">
        <b>1</b>
        <span>2</span>
    </div>
    <script>
        const box = document.getElementById('box')

        const observer = new MutationObserver(
            (mutationRecords) => console.log(mutationRecords)
        );

        observer.observe(box, { childList: true })

        // box.insertBefore(box.firstElementChild, box.lastElementChild)  // 即使最后顺序并没有发生改变，实际也是被移除后，再次插入原来的位置
        box.insertBefore(box.lastElementChild, box.firstElementChild)       

    </script>
</body>
```

![image-20220619135018950](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/02bbcff516174657976a400f6c13d2a4~tplv-k3u1fbpfcp-zoom-1.image)

## 观察子树

` MutationObserver`可以观察子树，只需要将`subtree`属性设置为`true`即可。

```html
<body>
  <div id="box">
    <div>1</div>
    <span>2</span>
  </div>
  <script>
    const box = document.getElementById('box')

    const observer = new MutationObserver(
      (mutationRecords) => console.log(mutationRecords)
    );

    observer.observe(box, {
      attributes: true,
      subtree: true
    });

    box.firstElementChild.setAttribute('haha', 'haha')
    box.firstElementChild.appendChild(document.createElement('b'))
  </script>
</body>
```

![image-20220619135321868](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/215b96a16488408498e9ef696b1064db~tplv-k3u1fbpfcp-zoom-1.image)

但是，从上面，我们可以发现，只有修改属性才会被观察到，添加节点时并没有被观察到，那是不是观察子树不能观察节点的添加和移除呢？
并不是，这里只是因为分工明确，`subtree`观察子树(不包括节点的添加和删除)，`childList`观察子节点，所以需要同时实现的话，那就需要两个属性都有。

```js
const box = document.getElementById('box')

const observer = new MutationObserver(
  (mutationRecords) => console.log(mutationRecords)
);

observer.observe(box, {
  attributes: true,
  subtree: true,
  childList: true
});

box.firstElementChild.setAttribute('haha', 'haha')
box.firstElementChild.appendChild(document.createElement('b'))
```

![image-20220619135459550](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42ac5ae90a9541eaab3cf9df2d1d996e~tplv-k3u1fbpfcp-zoom-1.image)
