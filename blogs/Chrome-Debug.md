---
title: Chrome调试
date: 2021-08-19 14:13:15
categories: "前端"
tags:
	- Chrome Debug
---

# Chrome 调试

## Elements 面板

步骤：

1. 打开 DevTools，有多种方式可以打开，F12 快捷键，右键检查等
2. 查看要检查的元素的样式，点击下图红框框，再点击页面元素，或者鼠标放在要检查的元素上面，右键检查

![](https://img-blog.csdnimg.cn/img_convert/4df1e370963a48cd14e2cf393bcea7ce.png) 3. 在 DevTools 下的 Styles 中增删改查样式
![](https://img-blog.csdnimg.cn/img_convert/777a54a2cedae3fc23d2230d726ee09e.png) 4. 类名操作

1. ，直接双击 Elements 下的类名，就可以进行修改类名
2. 点击"Styles"下的".cls"进行操作

![](https://img-blog.csdnimg.cn/img_convert/ae9a435f7997e9f7fc3d8b0ab3150407.png)

![](https://img-blog.csdnimg.cn/img_convert/a1e2c8ff34b23fe66c0d8aac87db6452.png) 5. 伪类选择器样式修改： 1. 在 Elements 中找到对应元素，右键选择 Force state，再选择伪类，如:hover,即可强制变样式，而伪类样式也可在 Styles 下进行修改

![](https://img-blog.csdnimg.cn/img_convert/b1f6aa872015b3d5cad54bdbe382cd0f.png)

![](https://img-blog.csdnimg.cn/img_convert/bb597773891e7fe233bb48757e93924b.png) 2. 点击"Styles"下的":hover"进行操作

![](https://img-blog.csdnimg.cn/img_convert/2e98d41882aecfe32881f8223e9e95c6.png)

6. 元素样式过多时，点击"Computed"，下面会有该元素的所有样式，点击"Filter"，输入要查看的样式即可

![](https://img-blog.csdnimg.cn/img_convert/8fcc8e36ddac89b59c1be509707375bc.png)

## Console 面板

可以通过程序在控制台中输出东西，来检查程序是否正确运行

例子：

```

	<script>
		var a = 10;
		console.log(a);
		var b = 15;
	</script>

```

![](https://img-blog.csdnimg.cn/img_convert/6717f32476055938b737ffb9eebbedce.png)

左侧可以选择日志等级，可以灵活运用日志等级

![](https://img-blog.csdnimg.cn/img_convert/339bbea3bdbd0d79839a035a935f5fee.png)

console.table()用法：可以用打印对象数组，方便

例子：

```

	<script>
		var student = [
			{
				name: "ttt",
				code: 111,
				price: 11
			},
			{
				name: "xxx",
				code: 333,
				price: 11
			},
			{
				name: "yyy",
				code: 333,
				price: 33
			},
		]
		console.log(student);
		console.table(student);
	</script>

```

结果：

![](https://img-blog.csdnimg.cn/img_convert/a7e687d571e4bbe9f47bae68c9c850ea.png)

红框部分是 console.table()，而红框上面的是 console.log()

占位符：

| 占位符 |      功能      |
| ------ | :------------: |
| %s     |     字符串     |
| %d     |      整数      |
| %f     |     浮点数     |
| %c     | css 格式字符串 |

![](https://img-blog.csdnimg.cn/img_convert/2493831c35c4c8f8ee85fd0c1debe9f7.png)

## Sources 面板

主要用来调试页面中的 JavaScript

步骤：

1. 打开 Sources 面板，找到要调试的 js 代码
2. 点击要调试部分代码左边的数字，添加断点
3. 刷新页面
4. 开始调试

![](https://img-blog.csdnimg.cn/img_convert/dbf743b32021771b7ea0422512e7fdc7.png)

调试常用部分：

![](https://img-blog.csdnimg.cn/img_convert/30fb7d53b50c746b6d634b3e279eaf9a.png)

截图来源：[谷歌浏览器调试--Sources](https://blog.csdn.net/weixin_45621649/article/details/105740165)

有点点特别的：

1. 鼠标悬浮变量可以查看变量值
2. 在程序中添加 debugger;相当于在这里设置断点
3. 特殊断点（事件断点）添加方法和上面的不同，是在 Event Listener Breakpoints 中添加

![](https://pic.imgdb.cn/item/611c6fcc4907e2d39cccfc22.jpg)

![](https://img-blog.csdnimg.cn/img_convert/8bfe87e7909ee0b5643d167a6e51b86e.png)

## Network 面板

可以用来模拟弱网环境

![](https://img-blog.csdnimg.cn/img_convert/2a236c81f21ffbb64292a251b7c96327.png)

## Application 面板

该面板主要是记录网站加载的所有资源信息，包括存储数据（Local Storage、Session Storage、IndexedDB、Web SQL、Cookies）、缓存数据、字体、图片、脚本、样式表等。

## 小技能

用上诉方法选中元素（节点），在 Elements 面板右键，选择下图红框即可截图

![](https://pic.imgdb.cn/item/611ccc5d4907e2d39c645eea.jpg)

截图效果：

![](https://pic.imgdb.cn/item/611ccca74907e2d39c655065.jpg)

拓展：[脱离 996，Chrome DevTools 面板全攻略！！！（收藏）](https://blog.csdn.net/qianyu6200430/article/details/107679089)
