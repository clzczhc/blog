# 工作中的那点事之 HTML、SVG

## SVG

### currentColor

```html
<style>
  .svg-container {
    color: red;
  }
</style>
<div class="svg-container">
  <svg
    width="16"
    height="16"
    viewBox="0 0 16 16"
    fill="none"
    xmlns="http://www.w3.org/2000/svg"
  >
    <circle cx="8" cy="8" r="7.25" stroke="#000" stroke-width="1.5" />
    <path
      d="M8 15.25C7.7649 15.25 7.48181 15.1442 7.15894 14.8321C6.83246 14.5165 6.50419 14.0235 6.21224 13.3562C5.62932 12.0239 5.25 10.1307 5.25 8C5.25 5.86928 5.62932 3.97615 6.21224 2.64376C6.50419 1.97645 6.83246 1.48352 7.15894 1.16789C7.48181 0.855751 7.7649 0.75 8 0.75C8.2351 0.75 8.51819 0.855751 8.84106 1.16789C9.16754 1.48352 9.49581 1.97645 9.78776 2.64376C10.3707 3.97615 10.75 5.86928 10.75 8C10.75 10.1307 10.3707 12.0239 9.78776 13.3562C9.49581 14.0235 9.16754 14.5165 8.84106 14.8321C8.51819 15.1442 8.2351 15.25 8 15.25Z"
      stroke="#000"
      stroke-width="1.5"
    />
    <line x1="1" y1="5.75" x2="15" y2="5.75" stroke="#000" stroke-width="1.5" />
    <line
      x1="1"
      y1="10.25"
      x2="15"
      y2="10.25"
      stroke="#000"
      stroke-width="1.5"
    />
  </svg>
</div>
```

上面的代码中，给 svg 的容器以及 svg 添加`color`样式都无法修改 icon 的颜色。
可以通过把由外面容器控制的颜色的值修改为`currentColor`。上面的例子中，则是把上面的`#000`都改成`currentColor`。

如：（上面的代码截取一小节

```svg
<circle cx="8" cy="8" r="7.25" stroke="currentColor" stroke-width="1.5" />
```

### svg use

> `use` 元素用于引用其他 SVG 元素。`xlink:href="#proto-image"`表示引用一个 ID 为 proto-image 的 SVG 图像

### dom 节点转 svg

[dom 节点转 svg](https://www.clzczh.top/2024/04/21/dom-to-svg/)

## `contenteditable`

> 给元素添加上该属性后，元素就可以被用户编辑。市面上的部分富文本编辑器就是基于该属性实现的，如`Quill`。

浅浅尝试：

```html
<div class="box" contenteditable>123</div>
<script>
  const element = document.querySelector("div");
  element.addEventListener("paste", (e) => {
    const file = e.clipboardData.files[0];

    const reader = new FileReader();
    reader.onload = () => {
      if (reader.result) {
        const img = document.createElement("img");
        img.src = reader.result;
        img.width = 200;
        element.appendChild(img);
      }
    };

    reader.readAsDataURL(file);
  });
</script>
```

> 添加`contenteditable`属性后，可以绑定`paste`监听粘贴操作，然后利用`FileReader`可以把文件转成 base64。

效果：
![](https://www.clzczh.top/CLZ_img/images/202504082031257.gif)

更多查看：
[contenteditable](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Global_attributes/contenteditable)
[深入浅出 contenteditable 富文本编辑器](https://zhuanlan.zhihu.com/p/37051858)

## base64 编解码

### `btoa`、`atob`

```js
let encodedData = window.btoa("Hello, world"); // 编码
let decodedData = window.atob(encodedData); // 解码
```

### Buffer

nodejs 环境可以利用`Buffer`实现

```js
const person = {
  name: "赤蓝紫",
};

const personString = JSON.stringify(person);

const buffer = Buffer.from(personString, "utf8");

const base64 = buffer.toString("base64");
console.log(base64);

const originalString = Buffer.from(base64, "base64").toString("utf8");
console.log(originalString);
```

### 可能会用到的其他方案

[How to convert uint8 Array to base64 Encoded String?](https://stackoverflow.com/questions/12710001/how-to-convert-uint8-array-to-base64-encoded-string/12713326#12713326)

## [HTML 标准](https://html.spec.whatwg.org/)

## 黑技术

http://browserhacks.com/
