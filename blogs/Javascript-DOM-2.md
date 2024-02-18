---
title: JavaScript DOM（二）
date: 2021-09-02 14:17:05
categories: 前端
tags:
  - JavaScript
---

# JavaScript DOM（二）

案例只留下案例名称，需复习的话，[下载素材](https://gitee.com/xiaoqiang001/java-script)，按名字搜索后可找到文件

## 节点操作

通过上文可知获取元素可以来利用 DOM 提供的方法来获取元素，如 getElementById、querySelector 等方法，但是也可以利用节点关系来获取元素

### 节点概述

![](https://pic.imgdb.cn/item/611f0cf54907e2d39ca04cce.jpg)

### 节点层级

利用 DOM 树可以把节点划分为不同的层级关系，如父子层级、兄弟层级

#### 父节点

node.parentNode

```

		//node.parentNode
		var son = document.querySelector(".son");
		console.log(son.parentNode);

```

1. parentNode 属性可返回最近的一个父节点
2. 指定的节点没有父节点则返回 null（测试只有 document.parentNode 会返回 null，body 里的节点的父节点可以是 body）

#### 子节点

1. parentNode.childNodes

返回包含指定节点的子节点的集合，包含元素节点、文本节点等

```

	<div class="father">
		<div class="son"></div>
	</div>
	<script>
		var father = document.querySelector(".father");
		console.log(father.childNodes);
	</script>

```

结果：

![](https://pic.imgdb.cn/item/611f120d4907e2d39cabddd5.jpg)

这种方法要获取元素节点，可以遍历所有节点，利用元素节点的 nodeType 为 1 的性质选出来。但是很麻烦。 2. parentNode.children

返回包含指定节点的子元素节点的集合，只返回元素节点

```

	<div class="father">
		<div class="son1"></div>
		<div class="son2"></div>
	</div>
	<script>
		var father = document.querySelector(".father");
		console.log(father.children);
	</script>

```

结果：

![](https://pic.imgdb.cn/item/611f13c84907e2d39cafcdad.jpg) 3. parentNode.firstChild

返回第一个子节点，也是所有的子节点中的第一个节点 4. parentNode.lastChild

返回最后一个子节点，也是所有的子节点中的最后一个节点 5. parentNode.firstElementChild

返回第一个子元素节点 6. parentNode.lastElementChild

返回最后一个子元素节点

也可以：parentNode.children[0]获取第一个子元素节点；parentNode.children[parentNode.children.length -1]获取最后一个子元素节点

#### 案例

新浪下拉菜单

#### 兄弟节点

两种方式，分别是所有的节点和元素节点。和获取子节点相似。

1. node.nextSibling

返回下一个兄弟节点，包含所有的节点。 2. node.previousSibling

返回下一个兄弟节点，包含所有的节点。 3. node.nextElementSibling

返回下一个兄弟元素节点 4. node.previousElementSibling

返回下一个兄弟元素节点

其中，3、4 有兼容性问题，IE9 以上才支持，可以封装兼容性函数

### 创建节点

document.createElement('tagName')
创建的元素原本不存在，是动态生成的，又被称为动态创建元素节点

```

	var div = document.createElement("div");

```

### 添加节点

创建节点后，创建的节点并不会出现，而需要把节点添加上去才可以。添加节点主要是先找到要添加的位置的父节点，然后才添加进去。有两种方法

1. node.appendChild(child)

将节点 child 添加到指定的父节点 node 的子节点末尾。

```

    <body>
    	<div class="box">
    		<div class="one"></div>
    		<div class="two"></div>
    	</div>

    	<script>
    		var div = document.createElement("div");
    		div.className = "three";

    		var box = document.querySelector(".box");
    		box.appendChild(div);

    	</script>
    </body>

```

结果：

![](https://i.loli.net/2021/08/23/wvRsMI43JqlcCzg.png) 2. node.insertBefore(child, 指定元素);

将节点 child 添加到父节点 node 的指定子节点前面

```

    <body>
    	<div class="box">
    		<div class="one"></div>
    		<div class="two"></div>
    	</div>

    	<script>
    		var div = document.createElement("div");
    		div.className = "three";

    		var box = document.querySelector(".box");
    		var one = document.querySelector(".one");
    		box.insertBefore(div, one);

    	</script>
    </body>

```

结果：

![](https://i.loli.net/2021/08/23/KGhHNEVASTIpCue.png)

### 案例

简单版发布留言案例

### 删除节点

node.removeChild(child)

从父节点 node 的子结点中删除指定子节点。，返回删除的节点。

```

	<div class="box">
		<div class="one"></div>
		<div class="two"></div>
	</div>

	<script>
		var box = document.querySelector(".box");
		console.log(box.removeChild(box.children[0]));
	</script>

```

结果:

![](https://i.loli.net/2021/08/23/o3mu1E7GzAtxjSf.png)

### 案例

删除留言案例

### 克隆节点

node.cloneNode()

返回调用该方法的节点的一个副本。

1. 参数为空或者 false，则是浅拷贝，只克隆节点自身，不克隆里面的子节点，包括文本节点
2. 参数为 true，则是深拷贝，克隆节点本身以及里面所有子节点。

```

	<div class="box">
		111
		<div class="one">123</div>
		<div class="two">456</div>
	</div>

	<script>
		var box = document.querySelector(".box");
		console.log(box.cloneNode());
		console.log(box.cloneNode(true));
	</script>

```

结果：

![](https://i.loli.net/2021/08/23/JgSYLnz2ci1kdCm.png)

### 节点操作综合案例

动态生成表格案例

### 三种动态创建元素方法

1. document.write()

会导致页面全部重绘

例子：

```

    <body>
    	<button class="write">write方法</button>
    	<script>
    		let write = document.querySelector(".write");
    		write.onclick = () => {
    			console.log(document.write("<h1>123</h1>"));
    		}
    	</script>
    </body>

```

点击前：

![](https://pic.imgdb.cn/item/6129f92844eaada739a03bef.jpg)

点击后:

![](https://pic.imgdb.cn/item/6129f94d44eaada739a075d2.jpg) 2. element.innerHTML

例子：

```

    <body>
    	<div></div>
    	<script>
    		let div = document.querySelector("div");
    		div.innerHTML = "<h1>123</h1>";
    	</script>
    </body>

```

3. document.createElement()

只能根据参数的标签名创建对应元素节点，无内容，也无类名、id 等。

例子:

```

	<div></div>
    <script>
        let div = document.querySelector("div");
        let h1 = document.createElement("h1");
        h1.innerHTML = "123";
        div.appendChild(h1);
    </script>

```

效率测试：

1. innerHTML（拼接字符串）(1000 个 div)

```

		const t1 = +new Date();
        for (let j = 0; j < 1000; j++) {
            document.body.innerHTML += '<div></div>';
        }
        const t2 = +new Date();
        console.log(t2 - t1);
        //505ms

```

2. innerHTML（数组形式拼接）（10000 个 div）

```
		const t1 = +new Date();
        let arr = [];
        for (let j = 0; j < 10000; j++) {
            arr.push('<div></div>');
        }

        document.body.innerHTML = arr.join('');
        const t2 = +new Date();
        console.log(t2 - t1);
        //15、14、13、11、9(ms)

```

3. createElement(10000 个 div)

```

		const t1 = +new Date();
        for (let j = 0; j < 10000; j++) {
            let div = document.createElement("div");
            document.body.appendChild(div);
        }

        const t2 = +new Date();
        console.log(t2 - t1);
        //14、16、19、13、11

```

innerHTML（数组形式拼接）的效率在测试中比 createElement 的稍微快一点，但只是一点点，一开始用 1000 个测试时，没办法分出区别，加大到 10000 个可以看出前者比后者稍快一点。

innerHTML（数组形式拼接）结构较复杂，需要另外用数组接，后面还得转成字符串，再塞给父节点。

createElement 结构较清晰，创建后直接使用 appendChild 就可以添加到父节点中。

学习链接：[pink 老师前端入门](https://www.bilibili.com/video/BV1Sy4y1C7ha)
