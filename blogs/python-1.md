---
title: Python(一)
date: 2021-09-25 17:05:25
categories: "Python"
tags:
  - Python
---

# Python(一)

本系列是个人学习 Python 的学习笔记。董付国编著的 Python 程序设计（第三版）

## 1. 介绍

- Python 是一门<b style="color: red">跨平台、开源、免费的解释型高级动态编程语言</b>b>，支持伪编译将 Python 源程序转换为字节码来优化程序和提高运行速度，支持使用 py2exe、**pyinstaller**或 cx_Freeze 工具将 Python 程序转换为二进制可执行程序。
- Python 支持<b style="color: red">命令式编程、函数式编程</b>，完全支持面向对象程序设计，拥有大量成熟<b style="color: red">扩展库</b>。

## 2. 对象模型

对象是 python 中最基本的概念，在 python 中处理的一切都是对象。

![](https://pic.imgdb.cn/item/61401ef344eaada739a0a693.jpg)

![](https://pic.imgdb.cn/item/61401f0c44eaada739a0cf04.jpg)

## 3. 变量

- 不需要事先声明变量名及其类型，直接赋值即可创建各种类型的对象变量。

- Python 属于<b style="color: red">强类型编程语言</b>，Python 解释器会根据赋值或运算来自动推断变量类型。

- Python 还是一种<b style="color: red">动态类型语言</b>，变量的类型可以随时变化。

- **字符串和元组属于不可变序列**，不能通过下标的方式来修改其中的元素值。

- Python 允许多个变量指向同一个值

  ```python
  x = 3
  print(id(x))

  y = x
  print(id(y))
  ```

![image-20210914130135133](https://s2.loli.net/2022/01/11/ysDYe9Cfp5VBgZU.png)

​ 其中的一个变量修改值之后，对应的内存地址会发生变化，但是不会影响另一个变量

```python
x = 3
print(id(x))

y = x
print(id(y))

x += 6
print(x)
print(id(x))

print(y)
print(id(y))

```

![image-20210914130401482](https://s2.loli.net/2022/01/11/CNzZvMPSsVLEIGd.png)

上面两个例子原因：Python 采用<b style="color: red">基于值的内存管理方式</b>,如果为不同变量复制为相同值，这个值在内存中只保存一份，多个变量指向同一个值的内存空间首地址，这样可以减少内存空间的占用，提高内存利用率。

Python 启动时，会对[-5, 256]区间的整数进行缓存，即多个变量的值相等，为整数，且介于[-5, 256]区间内，这些变量公用同一个值的内存空间。

在区间[-5, 256]区间之外的整数以及剩下的实数，会分为同一个程序和交互模式的区分。交互模式不常用，暂不考虑。同一个程序的同值不同变量会共用同一个内存空间

- 赋值语句的执行过程：首先把等号右侧表达式的值计算出来，然后在内存中寻找一个位置把值放进去，最后创建变量并指向这个内存地址。<b style="color: red">Python 中的变量并不是直接存储值，而是存储值的内存地址或者引用</b>，这也是变量类型可以随时改变的原因。
- Python 具有**自动管理内存**的功能，会跟踪所有的值，并自动删除不再使用或引用次数为 0 的值。

## 4. 数字

- 可以表示任意大小的值

- 整数类型可以分为：

      1. 十进制整数： 如0， -123
      2. 十六进制整数：必须用0x开头，如0xabcdf9
      3. 八进制整数：必须以0o开头，如0o35
      4. 二进制整数：必须以0b开头，如0b101

- 浮点数，如 15.0、1.2e2、1.2e-2

- Python 支持复数类型

  ```python
  a = 3 + 4j
  b = 5 + 6j
  c = a + b

  print(c)

  print(c.real)      #查看复数实部
  print(c.imag)      #查看复数虚部

  print(c.conjugate())      #返回共轭复数

  print(a * b)    # 复数乘法
  ```

- Python 3.6.x 开始支持在数字中间位置使用单个下划线作为分隔来提高数字的可读性

  ```python
  print(1_000_000)
  print(1_2_3_3)
  print(1_2 + 3_4j)
  print(1_2.3_45_678)
  ```

## 5. 字符串

字符串前面加字母 r 或 R 表示原始字符串，其中的特殊字符不进行转义，但字符串的最后一个字符不能是\。

![](https://pic.imgdb.cn/item/6141966a2ab3f51d91c404f8.jpg)

## 6. 运算符和表达式

- +运算符除了用于算数加法外，还可以用于列表、元组、字符串的连接，但<b style="color: red">不支持不同类型的对象之间相加或连接</b>，部分语言字符串和数字相加时，会把数字转成字符串后连接。

```python
a = [1, 2, 3]
b = [4, 5, 6]
print(a + b)

a = (1, 2, 3)
b = (4,)  # 元组只有一个元素时，后面应该带一个","
print(a + b)

a = "Hello"
b = " World!"
print(a + b)

print(True + 2)
print(False + 4)   # python会把True当成1，把False当成0处理
```

- \*运算符除了用于算数乘法外，还可以用于列表、字符串、元组等类型，对内容进行重复，并返回重复后的新对象

```python
print("a" * 4)
print([1, 2, 3] * 3)
print((4, 5, 6) * 2)
```

- python 中的除法有两种，"/"表示除法运算(真正数学意义上的除法)，"//"表示整除运算

  ```python
  print(3 / 5)         # 0.6
  print(3 // 5)        # 0

  print(3.0 / 5)       # 0.6
  print(3.0 // 5)      # 0.0

  print(13 // 10)      # 1
  print(-13 // 10)     # -2, -13 = -2 * 10 + 7
  ```

- 关系运算符可以进行连用，一般用于<b style="color: red">同类型对象之间</b>的大小比较，或者测试集合之间的包含关系。

  ```python
  print(1 < 3 < 5)    # 等价 1 < 3 and 3 < 5

  print("Hello" > "world")    # 比较字符串大小

  print([1, 2, 3] < [3, 1, 1])   # 比较列表大小

  print({1, 2} < {1, 2, 4})      # 测试是否是子集
  ```

- 成员测试运算符 in 用于成员测试，即测试一个对象是否是另一个对象的元素。

  ```python
  print(4 in [1, 2, 3])  # 测试4是否存在于列表[1, 2, 3]中

  print(5 in range(1, 10, 1))   # 测试5是否在[1, 10)中

  print("abc" in "abcdef")     # 子字符串测试

  for i in (3, 5, 7):
      print(i, end = ' ')  # 循环，成员遍历， end = ' ',可以更换print()的默认模式，print()默认会换行，这里改成空格
  ```

- 同一性测试运算符 is 用来测试两个对象是否是同一个，如果是，返回 True，否则返回 False。<b style="color: red">如果两个对象是同一个，二者具有相同的内存地址</b>。

  ```python
  a = 300.0
  b = 300.0
  print(a is b)

  x = [300, 300, 300]
  print(x[0] is x[1])   # 基于值的内存管理，同一个值在内存中只有一份

  a = [1, 2, 3]
  b = [1, 2, 3]
  print(a is b)
  ```

- **位运算符只能用于整数**，内部执行过程：先将整数转换为二进制数，然后右对齐，必要的时候左侧补 0，按位进行运算。

  ```python
  print(3 << 2)
  '''
  左移一位相当于乘以2，移两位相当于乘以4
  0011
  1100
  '''

  print(3 & 7)  # 位与运算
  '''
  0011 & 0111 = 0011
  '''

  print(3 | 8)  # 位或运算

  print(3 ^ 5)  # 位异或运算
  ```

- 集合的交集、并集、对称差集等运算借助位运算符来实现，而差集利用减号运算符实现（<b style="color: red">并集运算符不是用加号</b>）

  ```python
  print({1, 2, 3} | {2, 3, 4, 5})       # 并集，自动去除重复元素

  print({1, 2, 3} & {3, 2, 5})         # 交集

  print({1, 2, 3} ^ {3, 4, 5, 6})      # 对称差集

  print({1, 2, 3} - {3, 4, 5})        # 差集
  ```

- and 和 or 对应于 c 语言的&&和||，具有惰性求值的特点，只计算必须计算的表达式

  ```python
  print(3 > 5 and a > 3)
  '''
  PyCharm会显示错误，不过运行不会出现错误
  因为 3 > 5 的值为False,由于and运算符的惰性求值，不会计算之后的表达式
  '''

  print(3 < 5 or a > 3)   # 同理

  '''
  print(3 > 5 or a > 3)
  这个输出会报错，因为3>5的值为False，所以or被迫要去计算之后的表达式
  '''

  print(5 and 3)  # 最后一个计算表达式的值作为整个表达式的值
  print(3 and 5 < 3)

  print(3 not in [1, 2, 3]) # 逻辑非运算符not，对应于c语言的!
  ```

- 逗号不是运算符，只是普通分隔符

  ```python
  print('a' in 'b', 'a')
  print('a' in ('b', 'a'))

  x = 3,     # 赋值时使用逗号，变量会变为元组
  print(x)
  print(type(x))

  x = 3 + 5, 7
  print(x)
  ```

- <b style="color: red">Python 不支持++和--运算符</b>

## 7. 常用内置函数

int()：把实数转换为整数，或者把数字字符串按指定进制转换为十进制数

```python
print(int(3.5))
print(int(-3.5))

print(int(" \t  8 \n "))  # 自动忽略数字两侧的空格

print(int('101', 2))    # 转换为二进制
print(int('101', 16))   # 转换为十六进制
print(int('x2', 36))    # 转换为三十六进制
```

- ord()和 chr()是一对功能相反的函数。ord()返回单个字符对应的 ASCII 数值，或者 Unicode 数值。chr()返回传入的整数对应的字符。

  ```python
  print(ord('a'))
  print(chr(65))
  print(chr(ord('A') + 1))
  ```

- str()：将任意类型参数转换成字符串

  ```python
  print(str(7777))
  print([1, 2, 5])
  print(str(True))
  ```

- max()和 min()的 key 参数可以指定比较规则

  ```python
  arr = [12, 2, 4, 67, 99]
  print(max(arr))

  arr = ["0123", "456", "88"]
  print(max(arr))   # 正常比，比较每个字符的ASCII值，直到找到最大值
  print(max(arr, key=len))    # 比较字符串长度最大值
  print(max(arr, key=int))    # 字符串转为整型后比较最大值
  ```

- sum()可以实现**非数值型列表元素**的求和

  ```python
  print( sum([1, 2, 3, 4]) )
  ```

- type()和 isinstance()可以判断数据类型

  ```python
  print(type([3]))
  print(type((3, )))
  print(type({3}) in (list, tuple, dict))     # 判断{3}是否是list、tuple或dict的类型

  print(isinstance(2, int))       # 判断3是否是int类型的实例
  print(isinstance(3j, (int, float, complex)))    # 判断3j是否为int,float或complex的类型
  ```

- sorted()对列表、元组、字典、集合或其他可迭代对象进行排序并返回新列表。

  ```python
  a = (2, 77, 33, 33, 0, 12, 99)
  print(sorted(a))
  ```

- reversed()对可迭代对象（生成器对象和具有惰性求值特性的 zip、map、filter、enumeate 等类似对象除外）进行翻转，并返回可迭代的 reversed 对象。

  ```python
  a = (2, 77, 33, 33, 0, 12, 99)
  print(reversed(a))
  print(type(reversed(a)))
  print(list(reversed(a)))   # 列表 list() 方法用于将可迭代对象（字符串、列表、元祖、字典）转换为列表。
  ```

- range([start, ] end [,step]), 返回具有懒惰求值特点的 range 对象，其中范围是[start, end)内以 step 为步长的整数。start 默认为 0，step 默认为 1.

  ```python
  print(range(5))         # start默认为0，step默认为1
  print(type(range(5)))
  print(list(range(5)))

  print(list(range(1, 10, 3)))

  print(list(range(1, 10, -3)))  # 步长为负数时，如果start比end小，得到的range对象不含有任何整数
  print(list(range(10, 1, -3)))
  ```

- enumerate()：用来枚举可迭代对象中的元素，返回可迭代的 enumerate 对象，其中每个元素都是包含索引和值和元组。

  ```python
  print(list(enumerate("abcd")))          # 枚举字符串的元素

  print(list(enumerate(['Hello', "World"])))   # 枚举列表的元素

  print(list(enumerate({      # 枚举字典的元素
      'a': 97,
      'b': 98,
      'c': 99
  }.items())))    # items() 函数以列表返回可遍历的(键, 值) 元组数组。
  ```

- map()：把一个函数 func 依次映射到序列或迭代器对象的每个元素上，并返回一个可迭代 map 对象作为结果，map 对象中的每个元素是原序列中元素经过函数 func 处理后的结果。

  ```python
  print(map(str, range(5)))   # 把列表中的元素转换为字符串
  print(list(map(str, range(5))))

  print(list(range(5)))   # 没有用map方法的


  def add5(a):
      return a + 5


  print(list(map(add5, range(10))))   # 相当于把range()中的所有整数都会去调用add5


  def add(x, y):
      return x + y


  print(list(map(add, range(5), range(6, 12))))	# 两个参数版本的
  ```

- filter()：通过参数中的函数来筛选参数中的序列中符合条件的元素组成的 filter 对象，如果指定函数为 None，则返回序列中等价于 True 的元素。

  ```python
  arr = ['123', 'abc', '45?', '***']


  def func(x):
      return x.isalnum()  # 测试是否是字母或数字


  print(filter(func, arr))
  print(list(filter(func, arr)))

  print(list(filter(str.isalnum, arr)))    # 等价用法
  ```

  ```python
  data = list(range(20))

  filterObject = filter(lambda x: x % 2 == 1, data)   # 过滤，只留下所有奇数

  print(3 in filterObject)

  print(list(filterObject))       # filter只会访问一次元素，所以，3和3之前的元素不会输出

  print(list(filterObject))       # 上面的语句访问了所有的元素
  ```

- zip()：把多个可迭代对象中的元素压缩在一起，返回一个可迭代的 zip 对象，其中每个元素都是包含原来的多个可迭代对象对应位置上元素的元组。

  ```python
  print(list(zip('abcd', [1, 2, 3])))     # 压缩字符串和列表

  print(list(zip('123', 'abc', ',?!')))       # 压缩3个序列
  ```

- map、filter、enumeraye、zip 等对象具有惰性求值的特点，<b style="color: red">访问过的元素不可以再次访问</b>。

  例子见 filter()的第二个例子

## 8. 基本输入输出

### 8.1 input()：返回输入的对象。

<b style="color: red">input()函数的返回结果都是字符串</b>

```python
a = input("请输入第一个数:")
b = input("请输入第二个数:")
print("加法运算结果是:" + str(int(a) + int(b)))	# a, b都是字符串，转换为int类型后进行加法运算，之后再转换成字符串和前面的字符串进行拼接操作
```

### 8.2 print()：进行输出

```python
print(3, 5, 7)
print(1, 2, 3, sep=':: ')     # 指定分隔符

for i in (3, 5, 7):
    print(i, end='!! ')

'''
循环，成员遍历， end = ' ',可以更换print()的默认模式
print()默认会换行，这里改成空格
'''
```

## 9. 模块的导入

```python
import random
print(random.randint(1, 100))    # 获得[1, 100]上的整数
```

- ` from 模块名 import 对象名[ as 别名]` 可以减少查询次数，提高执行速度

  ```python
  from math import sin as f
  print(f(1.57))
  ```

- 如果需要导入多个模块，一般建议以下顺序进行导入

  1. 标准库
  2. 成熟的第三方扩展库
  3. 自己开发的库
