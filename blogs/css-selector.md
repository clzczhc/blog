---
title: CSS选择器
date: 2021-10-15 12:26:31
categories: "前端"
tags:
  - CSS
---

# CSS 选择器

## 1. 标签选择器

```html
<div></div>
```

```css
div {
  width: 100px;
  height: 100px;
  background-color: pink;
}
```

## 2. ID 选择器

```html
<div id="box"></div>
```

```css
#box {
  width: 100px;
  height: 100px;
  background-color: pink;
}
```

## 3. 类选择器

```html
<div class="box"></div>
```

```css
.box {
  width: 100px;
  height: 100px;
  background-color: pink;
}
```

## 4. 通配符选择器\*

匹配任何标签。**页面标签越多，效率越低**

```html
<h3>Hello</h3>
<span>World</span>
<div>!</div>
```

```css
* {
  color: red;
}
```

## 5. 后代选择器

```html
<div class="father">
  Father
  <div>
    Son
    <div>孙子</div>
  </div>
</div>
```

```css
.father div {
  color: red;
}
```

## 6. 子元素选择器

```html
<div class="father">
  Father
  <div>
    Son1
    <div>孙子</div>
  </div>
</div>
```

```css
div {
  color: blue;
}

.father > div {
  color: red;
}
```

## 7. 交集选择器

```html
<h3>h3</h3>
<div class="test">.test</div>
<h3 class="test">h3.test</h3>
```

```css
h3.test {
  color: red;
}
```

## 8. 并集选择器

```html
<h3>h3</h3>
<div class="box">box</div>
<span>span</span>
<h2>h2</h2>
```

```css
h3,
.box,
span {
  color: red;
}
```

## 9. 属性选择器

```html
<a href="javascript:;" title="aa hh" class="a b">aa</a>
<a href="javascript:;" class="b a c">bb</a>
<a href="javascript:;" title class="a">cc</a>
<a href="jsvascript:;" title="ac" class="a-test">ac</a>
<a href="jsvascript:;" title="ca">ca</a>
```

- [attr]

  选择存在 attr 属性的元素, attr 属性没有值也会选中

  ```css
  a[title] {
    /** 选择存在title属性的<a>元素, title属性没有值也会选中**/
    color: red;
  }
  ```

- [attr=value]

  选择存在 attr 属性且属性值为 value 的元素

  ```css
  a[title="aa"] {
    /** 选择存在title属性且属性值为aa的<a>元素**/
    color: red;
  }
  ```

- [attr*=value]

  选择存在 attr 属性且属性值中包含 value 值的元素

  ```css
  a[title*="a"] {
    /** 选择存在title属性且属性值中有a的<a>元素**/
    color: red;
  }
  ```

- [attr^=value]

  选择存在 attr 属性且属性值中以 value 值开头的元素

  ```css
  a[title^="a"] {
    /** 选择存在title属性且属性值以a为开头的<a>元素**/
    color: red;
  }
  ```

- [attr$=value]

  选择存在 attr 属性且属性值中以 value 值结尾的元素

  ```css
  a[title$="a"] {
    /** 选择存在title属性且属性值以a为结尾的<a>元素**/
    color: red;
  }
  ```

- [attr~=value]

  选择存在 attr 属性，且该属性是一个以空格作为分隔的值列表，其中至少有一个值为 value 的元素

  ```css
  a[class~="a"] {
    /** 选择存在class属性,且属性值是以空格作为分隔的值一系列值，其中至少有一个属性值是a**/
    color: red;
  }
  ```

- [attr|=value]

  选择存在 attr 属性，且属性值为“value”或是以“value-”为前缀的元素

  ```css
  a[class|="a"] {
    /** 选择存在class属性，且属性值为“a”或是以“a-”为前缀的元素**/
    color: red;
  }
  ```

## 10. 伪类选择器

**伪类**：同一个元素，有**不同的状态，有不同的样式**

可分为两种。

1. **静态伪类**：只能用于**超链接**的样式

   - ` :link`：超链接点击之前
   - ` :visited`：超链接被访问之后

   ```html
   <a href="http://www.xxx.com">xxx</a>
   ```

   ```css
   a:link {
     color: red;
   }

   a:visited {
     color: pink;
   }
   ```

2. **动态伪类**：**所有标签**都使用的样式

   - ` :hover`：鼠标悬停在标签上的时候
   - ` :active`：鼠标点击标签，但是还没松手的时候
   - `:focus`：标签获得焦点时的样式

   ```html
   <input type="text" value="获得焦点变色(:focus)" />
   <p>悬停变色(:hover)</p>
   <div class="box">按住变色(:active)</div>
   ```

   ```css
   input:focus {
     color: red;
   }

   p:hover {
     color: blue;
   }

   .box:active {
     color: purple;
   }
   ```

### 超链接的四个状态

- `:link`
- ` :visited`
- ` :hover`
- ` :active

<b style="color: red">在 css 中，超链接的四个状态必须按固定的顺序写：` :link -> :visited -> :hover -> :active`，否则可能会失效</b>

**按顺序一切正常**：

` <a href="http://www.com">www</a>`

```css
a:link {
  color: red;
}

a:visited {
  color: blue;
}

a:hover {
  color: purple;
}

a:active {
  color: pink;
}
```

**:active 换到最前面去**：按住链接时，不会变粉色了

```css
a:active {
  color: pink;
}

a:link {
  color: red;
}

a:visited {
  color: blue;
}

a:hover {
  color: purple;
}
```

原因：

css 样式是由权重的，上面的权重都相同，所以 a:hover 的样式会覆盖掉前面的 a:active 的样式，因为链接被激活时，鼠标也是悬停在链接上方的，所以效果是什么样，就看谁没有被覆盖了。

知道原理后，就可能会提出，这样的话，就没有必要一定要按照顺序了，只需要提高权重就行了。如下：

```css
a:active {
  color: pink !important;
}

a:link {
  color: red;
}

a:visited {
  color: blue;
}

a:hover {
  color: purple;
}
```

虽然这样子确实可以，但是按照代码规范就可以避免的问题，为什么要绕弯路呢？这样做以后也可能会引发出大问题。

## 11. 相邻兄弟选择器

**相邻兄弟选择器** (`+`) 介于两个选择器之间，当第二个元素*紧跟在*第一个元素之后，并且两个元素都是属于同一个父元素的子元素，则第二个元素将被选中。

```html
<div>
  <p>上</p>
  <h2>相邻兄弟选择器</h2>
  <p>下1</p>
  <p>下2</p>
</div>
```

```css
h2 + p {
  color: red;
}
```

## 12. 通用兄弟选择器

**a~b**：a 和 b 同级，选择 a 元素之后所有同级的 b 元素

```html
<div>
  <p>上</p>
  <h2>通用兄弟选择器</h2>
  <p>下1</p>
  <p>下2</p>
</div>

<p>不同级</p>
```

```css
h2 ~ p {
  color: red;
}
```

## 13. :nth-child() 选择器

**:nth-child()这个伪类选择器会先找到当前元素的兄弟元素，然后按位置先后顺序从 1 开始排序**

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Myself</title>
    <style>
      p:nth-child(2) {
        color: red;
        /* 先把p的兄弟元素找出来，排序，因为h2也是p的兄弟元素，所以h2的序号是1,所以p-1会变红 */
      }

      /* ele:nth-child()选择器是用来选择符合条件的ele元素的，所以p:nth-child(1)不会选择到任何元素，因为没有序号为1的p元素 */
    </style>
  </head>

  <body>
    <h2>h2-1</h2>
    <p>p-1</p>
    <p>p-2</p>
    <p>p-3</p>
    <p>p-4</p>
    <p>p-5</p>
    <p>p-6</p>
    <p>p-7</p>
    <p>p-8</p>
  </body>
</html>
```

其他用法：

<b style="color: red">如果:nth-child()括号中不是数字而是表达式，如 2n + 1，n 是从零开始整数，所以会选择序号为 1, 3, 5, ... , 2n+1 的元素。</b>

```css
p:nth-child(2n + 1)  /* 表示选中序号为奇数的p元素,这里的n是从零开始的 */
p:nth-child(2n)  /* 表示选中序号为偶数数的p元素 */

p:nth-child(odd)	/* 表示选中序号为奇数的p元素 */
p:nth-child(even)	/* 表示选中序号为偶数的p元素 */

p:nth-child(n + 3)	/* 表示选中序号大于等于3的p元素 */
p:nth-child(-n + 3)	/* 表示选中序号小于等于3的p元素 */
```

:nth-last-child()和:nth-child()用法类似，不同的是它从结尾开始排序。

## 14. :nth-of-type()选择器

用法和:nth-child()选择器类似。

不同的是：<b style="color: red">:nth-of-type()选择器是把要选择的元素按先后顺序排序。</b>如下面的例子，p:nth-of-type(2)会先把所有的 p 标签排序，而不是把 p 标签的兄弟元素进行排序，所以变红的就是 p-2

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Myself</title>
    <style>
      p:nth-of-type(2) {
        color: red;
      }
    </style>
  </head>

  <body>
    <h2>h2-1</h2>
    <p>p-1</p>
    <p>p-2</p>
    <p>p-3</p>
    <p>p-4</p>
    <p>p-5</p>
    <p>p-6</p>
    <p>p-7</p>
    <p>p-8</p>
  </body>
</html>
```

## 15. :first-child 选择器

和:nth-child(1)用法一样

## 16. :first-of-type 选择器

和:nth-of-type(1)用法一样

参考链接:

[MDN Web Docs (mozilla.org)](https://developer.mozilla.org/zh-CN/)

[CSS 选择器：伪类（图文详解） - 千古壹号 - 博客园 (cnblogs.com)](https://www.cnblogs.com/qianguyihao/p/8280814.html)

[CSS 的四种基本选择器和四种高级选择器*Jack-CSDN 博客*高级选择器](https://blog.csdn.net/DYD850804/article/details/80997251?utm_medium=distribute.pc_relevant_t0.none-task-blog-2~default~CTRLIST~default-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2~default~CTRLIST~default-1.control)
