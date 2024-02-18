---
title: python(四)
date: 2021-11-27 15:10:40
keywords: python Python
categories: "Python"
tags:
  - Python
---

# python(四)

## 1. 函数

### 1.1 关键参数

```python
def demo(a, b, c=5):
    print(a, b, c)

demo(3, 7)

demo(c=8, a=9, b=1)
```

### 1.2 可变长度参数

**\*p**: 用来接收任意多个实参，并将其放在一个元组中

```python
def demo(*p):
    print(p)


demo(1, 2, 3)
demo(1, 2, 3, 4, 5)
```

<b>\*\*p: </b>接收多个关键参数，并将其放入字典中

```python
def demo(**p):
    print(type(p))
    for item in p.items():
        print(item)


demo(x=1, y=2, z=3)
```

### 1.3 参数传递时的序列解包

```python
def demo(a, b, c):
    print(a + b + c)


seq = [1, 2, 3]
demo(*seq)

tup = (1, 2, 3)
demo(*tup)

dic = {1: 'a', 2: 'b', 3: 'c'}
demo(*dic)
# 使用字典对象作为实参时，默认使用字典的"键",需要"键值对"作为参数，需要使用items()方法，需要"值"作为参数,需要使用values()方法

demo(*dic.values())
```

```python
def demo(a, b, c):
    print(a + b + c)
    print(b)


demo(**{'b': 2, 'a': 3, 'c': 5})
# 使用字典作为函数实参时，并且在前面使用两个星号**进行解包时，会把字典的键作为参数名，值作为参数的值
```

### 1.4 全局变量

```python
def demo():
    global x    # global用来声明全局变量
    x = 3
    y = 4
    print(x, y)


demo()
print(x)
print(y)    # 会报错，因为y是局部变量，所以函数执行完之后，y就不再存在了
```

### 1.5 暂时性死区

```python
x = 3
def f():
    print(x)    # 这里会报错，因为暂时性死区，虽然外面有全局变量x，但是函数内部也有局部变量x，所以函数不会去访问全局变量x
    x = 5
    print(x)

f()
```

### 1.6 lambda 表达式

用于声明匿名函数

语法格式：

```python
lambda 变量: 返回值(操作)
```

```python
f = lambda x, y, z: x + y +z
print(f(1, 2, 3))

g = lambda x, y=2, z=3: x + y + z
print(g(2))
print(g(2, z=3, y=6))
```

### 1.7 map()

```python
# map(方法, 迭代器对象)   把迭代器对象中的元素都用参数一中的方法包装，返回一个map对象，不会改变迭代器对象
print(list(map(str, range(5))))

def add7(v):
    return v + 5
print(list(map(add7, range(7))))

def add(x, y):
    return x + y
print(list(map(add, range(5), range(5, 10))))
```

### 1.8 reduce()

```python
from functools import reduce

def func(factors, x):
    result = reduce(lambda a, b: a * x + b, factors)
    return result

factors = (3, 8, 5, 9, 7, 1)
'''
3 * 1 + 8 = 11
11 * 1 + 5 = 16
...
32 * 1 + 1 = 33
'''
print(func(factors, 1))
```

### 1.9 filter()

```python
seq = ['foo', 'x41', '?!', '***']
def func(x):
    return x.isalnum()

print(list(filter(func, seq)))  # filter(方法， 可迭代对象), 返回可迭代对象中可以使方法的返回值为True的元素组成的filter对象

print(list(x for x in seq if x.isalnum()))
```

### 1.10 生成器函数

```python
def f():
    a, b = 1, 1
    while True:
        yield a
        a, b = b, a + b     # 给出一个值，并暂停代码的执行

a = f()
for i in range(10):
    print(next(a), end=' ')     # next()向生成器索要一个值，恢复代码的运行

```

## 2. 面向对象程序设计

### 2.1 类的定义与使用

```python
class Car:
    price = 100000          # 定义类属性，类属性属于类，可以通过类名或对象名访问
    def __init__(self, c):
        self.color = c      # 定义实例属性，实例属性属于对象，只能通过对象名访问

car1 = Car('Red')
car2 = Car('Blue')
print(car1.color, car2.color)

Car.price = 110000          # 修改类属性
Car.name = 'TT'             # 增加类属性

car1.color = 'Pink'         # 修改实例属性

print(Car.price, Car.name)
print(car1.color, car2.color)
```

### 2.2 私有成员

```python
class A:
    def __init__(self, value1=0, value2=0):
        self.value1 = value1
        self.__value2 = value2      # 私有成员，以两个下划线开头，但不以两个下划线结束

a = A()
print(a.value1)
# print(a.__value2)                  # 私有成员不允许从外部访问
print(a._A__value2)                  # 通过特殊的方式从外部访问私有成员
```

```python
class Test:
    def __init__(self, value):
        self.__value = value        # value是私有成员，所以需要增加访问、修改、删除属性的方法，并通过修饰器定义属性

    def __get(self):
        return self.__value

    def __set(self, v):
        self.__value = v

    def __del(self):
        del self.__value

    value = property(__get, __set, __del)       # 修饰器，定义属性

    def show(self):
        print(self.__value)

t = Test(3)
print(t.value)

t.value = 5
print(t.value)

del t.value
```

## 3. 文件操作

写文件

```python
s = 'test\nhello\nworld'

with open('data.txt', 'a+') as f:
    f.write(s)
```

读文件

```python
with open('data.txt', encoding='utf8') as fp:
    print(fp.read(5))    # 读取并显示文本文件的前5个字符
    print(fp.read(5))    # 读取并显示文本文件的第二轮的5个字符，因为是通过文件指针形式进行读取文件的
    fp.seek(0)           # 移动文件指针回到开头

    for line in fp:      # 读取并显示文本文件的所有行
        print(line)
```

### 3.1 二进制文件操作

#### 3.1.1 使用 pickle 模块

写入二进制文件

```python
import pickle
i = 13000000
a = 99.056
s = '中国人名123abc'
lst = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
tu = (-5, 10, 8)
coll = {4, 5, 6}
dic = {'a': 'apple', 'b': 'banana', 'g': 'grape', 'o': 'orange'}
data = [i, a, s, lst, tu, coll, dic]

with open('sample_pickle.dat', 'wb') as f:
    try:
        pickle.dump(len(data), f)   # 第一次把要写的数据长度写进去
        for item in data:
            pickle.dump(item, f)    # 依次把数据写进去
    except:
        print('写文件异常')
```

读取二进制文件的内容

```python
import pickle

with open('sample_pickle.dat', 'rb') as f:
    n = pickle.load(f)      # 先读长度
    for i in range(n):
        x = pickle.load(f)  # 再读数据
        print(x)
```

### 3.2 文件级操作

```python
# 文件级操作
import os
print([fname for fname in os.listdir(os.getcwd())   # os.getcwd()返回当前工作目录
       if os.path.isfile(fname) and fname.endswith('.py')])     # 返回扩展名为py的文件

file_list  = [filename for filename in os.listdir()
              if filename.endswith('.html')]
for filename in file_list:
    newname = filename[:-4] + 'htm'
    os.rename(filename, newname)                # 把所有扩展名为html的文件都改为htm
    print(filename+"更名为"+newname)

# 使用shutil模块
import shutil
shutil.copyfile('.\data.txt', '.\data2.txt')

shutil.make_archive('.\\a', 'zip', 'C:\\Users\\CZH0318\\Desktop\\wallhaven')    # 压缩
shutil.unpack_archive('.\\a.zip', '.\\a_unpack')                                # 解压缩

shutil.rmtree('.\\a_unpack')        # 删除刚刚解压的文件夹
```

### 3.3 目录操作

```python
# 目录操作
import os
os.mkdir(os.getcwd()+'\\temp')      # 创建目录
print(os.listdir('.'))              # 返回指定目录下的文件和目录信息
os.rmdir('temp')                    # 删除目录
```

## 4. 异常处理

### 4.1 Python 中的异常处理结构

#### 4.1.1 try...except...结构

```python
while True:
    try:
        x = int(input("请输入一个整数: "))
        break
    except ValueError:
        print("输入不符合条件")
```

try 块是被监控的语句，有可能会引发异常

except 块用于处理异常的代码

#### 4.1.2 try...except...else...结构

```python
a_list = ['China', 'America', 'England', 'France']

while True:
    n = input("请输入字符串的序号: ")
    try:
        n = int(n)
        print(a_list[n])
    except IndexError:
        print("列表的下标越界或格式不正确")
    else:
        print("只有try块语句没有抛出异常，才会执行")
        break
```

else 块语句只有当 try 块语句没有抛出任何异常，才会执行

#### 4.1.3 带有多个 except 的 try 结构

```python
try:
    x = input("请输入被除数: ")
    y = input("请输入除数: ")
    z = float(x) / float(y)
except TypeError:
    print("被除数和除数应该是数值类型")
except ZeroDivisionError:
    print("除数不能为0")
except NameError:
    print("变量不存在")
else:
    print(x, " / ", y, " = ", z)
```

一旦某个 except 捕获了异常，则后面的 except 都不会再执行，所以比较精准的异常应该尽量在前面，而 BaseException 应该放在最后一个 except 中。

#### 4.1.4 try...except...finally...结构

```python
a_list = ['China', 'America', 'England', 'France']

while True:
    n = input("请输入字符串的序号: ")
    try:
        n = int(n)
        print(a_list[n])
    except IndexError:
        print("列表的下标越界或格式不正确")
    finally:
        print("一定会执行的语句，无论try块语句有没有抛出异常")
        break
```

```python
def divide(x, y):
    try:
        result = x / y
    except ZeroDivisionError:
        print("division by zero!")
    else:
        print("result is", result)
    finally:
        print("executing finally clause")


divide(2, 1)
divide(2, 0)
divide("2", "1")
```

### 4.2 断言

断言语句语法：

```python
assert expression[, reason]		# 当判断表达式expression为真时，什么都不做；为假时，则会抛出异常
```

```python
try:
    assert 1 == 2, "1 is not equal 2!"
except AssertionError as reason:
    print("%s:%s" %(reason.__class__.__name__, reason))
```

### 4.3 上下文管理

```python
with open('test.txt') as f:   # 读取当前目录下的test.txt文件
    for line in f:
        print(line)         # 使用with语句，文件处理完之后，会自动关闭，不需要手动关闭文件with open('test.txt') as f:   # 读取当前目录下的test.txt文件
    for line in f:
        print(line)         # 使用with语句，文件处理完之后，会自动关闭，不需要手动关闭文件
```

上下文管理语句 with 可以自动管理资源，在代码执行完之后自动还原进入代码块之前的现场
