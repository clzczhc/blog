---
title: 小试Python(一)
date: 2021-08-04 19:27:32
categories: "Python"
tags:
	- Python
---

# 小试 Python(一)

初步学习一下 Python。

## 介绍

Python 是一种解释型、面向对象、动态数据类型的高级程序语言。

官方宣布，2020 年 1 月 1 日，停止 Python2 的更新。

Python2.7 是最后一个 Python2.x 版本。

## 第一个 Python 程序

Python 不像 C++、Java 一样，需要有主函数,语句后也不需要分号，函数、条件控制、类等不需要有"{}"包住，但需要有缩进，有缩进相当于加上"{}"，赋值语句缩进会出错。

`print("Hello World!")`
![](https://img-blog.csdnimg.cn/img_convert/667c9c7a38af5c81e762945c00848b4c.png)

## 变量

和其他语言相似，定义时不需要强调类型，直接赋值

```

    name = "John"
    num1 = 10
    num2 = 20
    print(name + ":" + str(num1 + num2))

```

![](https://img-blog.csdnimg.cn/img_convert/e1fbc4554af2acc8b5efddfed08689b0.png)

另外，Python 中没有自增"i++"和自减"i--",但存在"i += 1"

## 输入

输入的默认都是字符串，需要整型、浮点型需转换，
字符串可直接用"+"连接

```
    num1 = input("Enter first number: ")
    num2 = input("Enter second number: ")
    result = int(num1) + int(num2)
    print("Result is: " + str(result))

```

![](https://img-blog.csdnimg.cn/img_convert/d059987cff8a4e5c95d95705bbce9a9c.png)

## 数组(列表)

下标可以是负数，-1 代表最后一个元素，-2 代表倒数第二个...，正数和负数都不可以超出数组范围

取部分：
num[1:]:输出下标 1 及下标比 1 大的所有数组元素
num[1:4]:输出下标范围为[1, 4)的元素，注意：不包括 4

```
    num = [1, 2, 3, 4, 5, 6]
    print(num)
    print(num[1])
    print(num[-2])
    print(num[1:])
    print(num[1:4])
    print(len(num))

```

![](https://img-blog.csdnimg.cn/img_convert/eac7ea7a580a0494b258e92039c7a6e3.png)

## 元组

和数组类似，元组的元素不能修改，使用小括号

```
    tuples = (1, 2, 3, 4, 5, 6)
    print(tuples)
    print(tuples[1])
    print(tuples[1:])
    print(tuples[1:4])

```

![](https://img-blog.csdnimg.cn/img_convert/2b5359646b9644e34ea781f311441d91.png)

## 函数

关键字 def 开头

```

    def add(num1, num2):
    	result = num1 + num2
    	return result
    def display():
    	print("Hello World!")
    print(add(1, 1))
    display()

```

## 条件控制（if）

Python 中没有"&&"和"||"，也没有"if(!0)"这种用法，分别有类似的逻辑运算符"and"，"or"，"not"。

```

	test = True
	if test and False:
    	print(1)
	elif False or 0:
    	print(2)
	elif not(0):
    	print(3)
	else:
    	print(4)

```

结果：细品

## 字典

字典的每个键值对 key=>value 用冒号分割，整个字典包括在花括号中，可存储热恩义类型对象

```

    dictionary = {
    	1: "one",
    	2: "two",
    	"o": 1,
    	"t":"test"
    }
    print(dictionary[1])
    print(dictionary["t"])
    print(dictionary.get("o"))
    print()

    print(dictionary.get(5))  # 用get()找不到不会报错，只有一个参数时，找不到返回"None"
    print(dictionary.get(5, "You can't find it")) # 两个参数时，找不到返回第二个参数

```

![](https://pic.imgdb.cn/item/60fcece85132923bf8ddc485.jpg)

## while 循环

和其他语言基本相同原理

示例如下：

```

    i = 1
    while i <= 10:
    	print(i)
    	i += 1

```

## for 循环

相当于 Java 的增强 for 循环。

### 遍历数组示例：

```

    number = [1, 2, 3, 4, 5, 7]
    for n in number:
    	print(n)

```

结果：

![](https://pic.imgdb.cn/item/610771125132923bf8aebc5b.jpg)

### 搭配 range()函数

1. 只有一个参数 n：遍历[0,n)，不包括 n

```

    for index in range(6):
    	print(index)

```

结果：

![](https://pic.imgdb.cn/item/610772165132923bf8b1b40d.jpg) 2. 有两个参数，第一个参数为 m,第二个参数为 n:遍历[m,n)。

使用 range()函数遍历数组：先用 len()函数得到数组长度，然后用以上方法可以得到数组下标[0,n)。

```

    number = [1, 2, 3, 4, 5, 7]
    for index in range(len(number)):
    	print(number[index])

```

结果和上面遍历数组的方法一样。

学习:Youtube
Mike Dane
