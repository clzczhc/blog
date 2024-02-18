---
title: 小试Python(二)
date: 2021-08-13 15:12:10
categories: "Python"
tags:
	- Python
---

# 小试 Python(二)

## if x in y

统计两个数组的相同元素个数示例：

```

    num1 = [1, 2, 3, 4, 5, 6]
    num2 = [4, 5, 6, 7, 8, 9]

    count = 0
    for i in num1:
    	if i in num2:
    	count += 1

    print(count)

```

上面的 if i in num2，i 是遍历 num1 数组的每一次值，通过 if i in num2 来判断 i 是否在 num2 数组中。

## 二维数组

不要求每一行的元素个数相同

二维数组定义及遍历：

```

    number_grid = [
    	[1, 2, 3],
    	[4, 5, 6],
    	[1, 2],
    	[1]
    ]
    for row in number_grid:
    	for col in row:
    		print(col)

```

## 运算符\*\*

这个运算符，我目前学过的语言中并没有遇到过，它是指数运算符，它的优先级和数学上一样，是最高的。

```

    print(3 ** 3)

```

结果：

![](https://pic.imgdb.cn/item/610774cf5132923bf8ba1c79.jpg)

## 注释

单行注释：&nbsp;&nbsp;`# 注释内容`

多行注释：三个单引号包住注释部分

```

    '''
    注释内容
    注释内容
    注释内容
    '''

```

## 异常处理

和 Java 的异常处理机制一样

```

	try:
		会出异常代码
	except 监听异常类型:
		提示出异常的代码

```

可以直接在异常类型后加`as 变量名`,之后直接 print(变量名)，打印出提示信息。

代码：

```

    try:
    	value = 10 / 0
    except ZeroDivisionError as err:
    	print("Divided by Zero")
    	print(err)

```

结果：

![](https://pic.imgdb.cn/item/6107e4d85132923bf86dab14.jpg)

## 文件方法

open(参数 a, 参数 b)函数,参数 a 和参数 b 都是字符串形式，参数 a 是要打开的文件的相对路径或绝对路径，参数 b 是文件打开模式。后面需要关闭文件。

参数 b:

1. "r":以只读方式打开文件。文件的指针将会放在文件的开头。这是默认模式。
2. "w":打开一个文件只用于写入。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。
3. "r+":打开一个文件用于读写。文件指针将会放在文件的开头。
4. "w+":打开一个文件用于读写。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。
5. "a":打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。

### 读文件

readable():查看文件是否可读

read():读整个文件

readline():读一行

readlines():返回一个数组，数组的每一个元素分别是
文件的一行

```

    file = open("test.txt", "r")

    print(file.readable())
    print(file.read())

    file.close()

```

read()这里的代码会空两行，一行是 print()的一行，还有一行是 read()每打印出一行的换行。

<p style = "color: red">read()在当前指针处开始读，而执行完一次后，指针在文件尾部，其后为空，所以之后的read()每次运行均为空。readline()、readlines()同理</p>

readlines()

要打开的文件：

```

    123
    456
    789

```

readlines()打印出来：

```

    ['123\n', '456\n', '789\n']

```

### 写文件

1. open()函数的第二个参数为"a",在后面添加新内容，具体如参数 b 中所示。

这里需要注意：没有换行符号的话会出现写的东西非常混乱。

```

    file = open("test.txt", "a")

    file.write("123")
    file.write("123")

    file.close()

```

```

    没执行程序前：
    123
    456
    789
    执行后：
    123
    456
    789123123

```

应该改为`file.write("\n123")`,才可以实现每一次增加的都是单独一行而不会混乱。 2. open()函数的第二个参数为"w"。和上面的相似，不同的是：不是在文件后添加新内容，而是重写文件内容。

### 读写文件

open()函数的第二个参数为"r+"或"w+"。

"r+"和"w+"相同点：

1. 文件权限都是可读可写
2. 文件的指针放在文件的开头

不同点：

"r+"不是重写文件，而是覆盖，即当原来的文件内容比写的文件内容少时，后面的内容还在，而"w+"是重写文件。

例子：

原来的内容：

123456789

写"abc":

"r+":变为"abc456789"

"w+":变为"abc"

自己尝试后出现的问题:

1. 打印不出东西:

```

    file = open("test.txt", "r+")

    file.write("\n123\n456\n789")
    print(file.read())

    file.close()

```

2. 每次打印的都是文件打开前的内容，且从重写文件变化成了在文件后添加内容，即和参数为"a"时一样。

```

    file = open("test.txt", "r+")

    file.write("\n123\n456\n789")
    print(file.readlines())

    file.close()

```

原因：

文件指针在文件的开头，经过 write()方法对文件进行写操作后，这时候的文件指针已经来到了文件的尾部。

read()在当前指针处开始读，而当前指针在文件尾部，其后为空，所以打印文件为空（两行空行）。

readlines()时回到文件开头处开始读。而刚刚写入的还没有保存，所以只能读出写入操作之前的内容。写的时候在文件尾部写。<b style = "color:red">未解决疑问：为什么 write()搭配 readlines()后，写文件时是在文件尾部写，而搭配 read()时是在文件开头写。</b>

上面问题解决方案：使用 seek()函数，让文件指针指向需要的位置。seek(0)即指向文件开头。

```

    file = open("test.txt", "r+")

    file.write("\n123\n456\n789")
    file.seek(0)
    print(file.read())  //print(file.readlines())

    file.close()

```

## 类

### 构造函数

```

    def __init__(self, 参数a): # 注意下划线都是两条,第一个参数不需要传参，相当于其它语言的"this"

```

例子：

其中，类单独放在了另一个 py 文件中。

Student 类

```

    class Student:

    	def __init__(self, name, major, gpa):
    		self.name = name
    		self.major = major
    		self.gpa = gpa

    	def on_honor_roll(self):
    		if self.gpa >= 3.5:
    			return True
    		else:
    			return False

```

main 类:

```

    from Student import Student

    student1 = Student("Jim", "Business", 3.1)
    student2 = Student("Pam", "Business", 3.8)

    print(student1.on_honor_roll())
    print(student2.gpa)

```

### 继承

```

	class 子类(父类):
		子类方法，可以重写覆盖父类的方法，也可以增添方法

```

实例：

Chef 类(父类):

```
    class Chef:

    	def make_chicken(self):
    		print("The chef makes a chicken")

    	def make_salad(self):
    		print("The chef makes salad")

    	def make_special_dish(self):
    		print("The chef makes bbq ribs")

```

ChineseChef（子类）:

```

    from Chef import Chef

    class ChineseChef(Chef):

    	def make_special_dish(self):
    		print("The chef makes orange chicken")

    	def make_fried_rice(self):
    		print("The chef makes fried rice")

```

main 类：

```

    from Chef import Chef
    from ChineseChef import ChineseChef

    chef = Chef()
    chef.make_chicken()
    chef.make_special_dish()

    print()

    chinesechef = ChineseChef()
    chinesechef.make_chicken()
    chinesechef.make_special_dish()
    chinesechef.make_fried_rice()


```

## 解释器

### 环境变量设置

1. 右键点击"计算机"，然后点击"属性"
2. 点击"高级系统设置"

![](https://pic.imgdb.cn/item/610f67525132923bf8ee2b60.jpg) 3. 点击"环境变量"

![](https://pic.imgdb.cn/item/610f67dc5132923bf8ef0326.jpg) 4. 选中"用户变量的 Path",点击"编辑"

![](https://pic.imgdb.cn/item/610f68ca5132923bf8f04bda.jpg) 5. 点击"新建",添加 python 安装路径

![](https://pic.imgdb.cn/item/610f69a15132923bf8f17825.jpg) 6. 之后一直点击"确定"即可

### cmd 写 Python

1. win + r, 输入"cmd"
2. 输入"python", 变成下图所示

![](https://pic.imgdb.cn/item/610f6a4e5132923bf8f270ec.jpg) 3. 写 python 代码，如下图所示

![](https://pic.imgdb.cn/item/610f66855132923bf8ed19c6.jpg)

学习:Youtube：Mike Dane

文件读写部分参考：[Python 文件进行写操作后立即读出的结果、原因分析、解决方法](https://blog.csdn.net/qq_38106472/article/details/86305846)
