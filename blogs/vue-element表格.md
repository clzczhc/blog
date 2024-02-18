---
title: Element Plus修改表格行、单元格样式
categories: 前端
date: 2022-04-30 17:24:29
tags:
  - Vue3
  - Element
---

# Element Plus修改表格行、单元格样式

## 前言

实习工作需要根据表格的状态字段来设置行的样式，记录一波。

先来一下基础配置。(Vue3)

```html
<template>
  <el-table :data="tableData" border style="width: 400px">
    <el-table-column prop="name" label="姓名" width="100" />
    <el-table-column prop="age" label="年龄" width="100" />
    <el-table-column prop="job" label="工作" />
  </el-table>
</template>

<script setup>
const tableData = [
  {
    name: "clz",
    age: 21,
    job: "Coder",
  },
  {
    name: "czh",
    age: 21,
    job: "Coder",
  },
  {
    name: "赤蓝紫",
    age: 21,
    job: "Coder",
  },
];
</script>

<style lang="less" scoped>
</style>
```

![image-20220425211149555](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/631eedb6885c44babc3af2a9e48b3642~tplv-k3u1fbpfcp-zoom-1.image)



## 设置某一行的样式

主要是通过` row-style`属性来实现。它是行的` style`的回调方法，可以通过它来实现设置某一行的样式。

先让我们来体验一下它的参数都是些什么。

```html
<el-table 
  style="width: 400px" 
  border 
  :data="tableData" 
  :row-style="rowState"
>
</el-table>
```



```js
const rowState = (arg) => {
  console.log(arg)
}
```

![image-20220425213221090](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/644e485b23d34bd1a2867b2c1a7911e4~tplv-k3u1fbpfcp-zoom-1.image)



可以发现，它是一个对象，一个属性是行的数据，一个是行号(从0开始)，至于不只是打印3次，而是打印9次的原因还没发现，后面单元格的会打印18次，9个单元格打印18次。但是这个并不是本次的研究重点。



那么，我们怎样能设置样式呢？

只需要返回含有属性样式的对象即可。(**驼峰命名法**)

```js
const rowState = (arg) => {
  return {
    backgroundColor: 'pink',
    color: '#fff'
  }
}
```

![image-20220425213934820](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/82c6c7eb457c4a96b53f91f4248ccb0b~tplv-k3u1fbpfcp-zoom-1.image)



然后在搭配参数使用，就能实现根据表格内容设置行的样式。

```js
const rowState = ({ row }) => {
  let style = {}

  switch (row.name) {
    case 'clz':
      style = {
        backgroundColor: 'red'
      }
      break;
    case 'czh':
      style = {
        backgroundColor: 'blue'
      }
      break;
    case '赤蓝紫':
      style = {
        backgroundColor: 'purple'
      }
      break;
  }

  return style;
}
```

![image-20220426211744236](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed99c47db4f9492c97ad837cd819443a~tplv-k3u1fbpfcp-zoom-1.image)



## 设置某一个单元格的样式

通过` cell-style`属性来实现。做法和上面一样，就不多说了，主要的四个参数` row, column, rowIndex, columnIndex`。

* `row`：行的信息
* `column`：列的信息
* `rowIndex`： 行数(0开始算)
* `columnIndex`：列数(0开始算)



```html
<el-table 
  style="width: 400px" 
  border 
  :data="tableData" 
  :cell-style="cellStyle"
>
</el-table>
```



```js
const cellStyle = ({ row, column, rowIndex, columnIndex }) => {
  if (rowIndex === 1 && columnIndex === 1) {
    return {
      backgroundColor: 'pink'
    }
  }
}
```

![image-20220426232712515](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/04e946b91a5f44d2b83b7603b8d40b6e~tplv-k3u1fbpfcp-zoom-1.image)



其实，`cell-state`不只是能设置单元格的样式，因为它的参数中含有` row`和` column`，所以还可以用来设置某一行或某一列的样式。

```js
const cellStyle = ({ row, column, rowIndex, columnIndex }) => {

  if (column.label === '工作') {
    return {
      backgroundColor: 'purple'
    }
  }

  if (row.name === '赤蓝紫') {
    return {
      backgroundColor: 'red'
    }
  }

}
```

![image-20220426233521464](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dfccea39d66f4b4c8045e21c29712fb3~tplv-k3u1fbpfcp-zoom-1.image)



<b style="color: red">注意，这里重叠的地方并不会出现后来的样式覆盖掉前面的样式，而是先到先得</b>

## 表头样式修改(赠品)

特殊的表头，特殊的处理

`header-row-style`：只有一个`rowIndex`属性

```
const headerRowStyle = (args) => {
  console.log(args)

  return {
    height: '100px',
    backgroundColor: 'red'
  }
}
```

![image-20220427220605501](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ec6ae0d71a6490199c26322b3b259d7~tplv-k3u1fbpfcp-zoom-1.image)

发现只有标头的行高有所变化，这是为啥呢？

![image-20220427220742235](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e213d53f38334f71846e8723a640b518~tplv-k3u1fbpfcp-zoom-1.image)

检查样式发现，这是因为单元格本身具有背景颜色，所以并不会生效。

`header-row-style`：和正常的单元格一样，有四个属性

```
const headerCellStyle = ({ row, column, rowIndex, columnIndex }) => {
  if (columnIndex === 1) {
    return {
      backgroundColor: 'pink'
    }
  }
}
```

![image-20220427221737551](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3aba8d031704da3a97767ade4b2c224~tplv-k3u1fbpfcp-zoom-1.image)

也可以通过`column`属性来设置符合条件的表头单元格的样式。

```
const headerCellStyle = ({ row, column, rowIndex, columnIndex }) => {

  if (column.label === '姓名') {
    return {
      backgroundColor: 'red'
    }
  }
}
```

![image-20220427222630382](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b8e2ff1f91304924a85c3c421e8e2f5b~tplv-k3u1fbpfcp-zoom-1.image)

\
