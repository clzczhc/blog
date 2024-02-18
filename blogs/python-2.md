---
title: Python(二) 序列
date: 2021-10-20 14:26:05
categories: "Python"
tags:
  - Python
---

# Python(二) 序列

常用的序列结构：列表、元组、字符串、字典、range、zip、enumerate 等

## 1. 列表

**列表对象常用方法**

​ 1. append(x)：将元素 x 添加到列表尾部

​ 2. extend(可迭代对象)：将可迭代对象中所有的元素添加到列表尾部

​ 3. insert(index, x)：在列表指定位置 index 处添加元素 x

​ 4. remove(x)：在列表中删除首次出现的指定元素

​ 5. pop([index])：删除并返回列表中指定位置的元素，默认为最后一个元素

​ 6. clear()：删除列表中所有元素，会保留列表对象

​ 7. index(x)：返回第一个值为 x 的元素的下标，不存在则抛出异常

​ 8. count(x)：返回指定元素 x 在列表中的出现次数

​ 9. reverse()：对列表元素进行原地翻转

​ 10. sort()：对列表元素进行排序

​ 11. copy()：返回列表对象的浅复制

### 1.1 列表创建、元素的增加、元素的删除

```python
a_list = [1, "Hello", "World"]
print(a_list)
print(type(a_list))  # 列表的创建
print(list("Hello World!"))  # 通过list()方法将可迭代对象类型转换为列表
print()

a_list += [7]   # 元素增加方法1
print("元素增加方法1：")
print(a_list)
print()

a_list.append("two")    # 元素增加方法2(append())
print("元素增加方法2：")
print(a_list)
print()

a_list.extend(["three1", "three2"])     # 元素增加方法3(extend())
print("元素增加方法3：")
print(a_list)
print()

a_list.insert(0, 99)    # 元素增加方法4(insert),第一个参数是要新增元素插入的位置,第二个参数是要增加的元素
print("元素增加方法4：")
print(a_list)
print()

a_list = [1, 2, 3, 4, 5]
del a_list[1]   # 使用del命令删除列表指定位置上的元素
print(a_list)
print()

print(a_list.pop())    # 使用列表的pop()删除并返回指定位置上的元素(默认是最后一个)
print(a_list)
print()

a_list = [1, 2, 3, 2, 4]
a_list.remove(2)    # 使用列表的remove()删除第一次出现的指定元素
print(a_list)
```

### 1.2 列表元素的访问、计数、成员资格判断

```python
aList = [1, 2, 3, 4, 5, 4, 9]
print(aList[2])     # 列表的访问
aList[2] = "Hello"  # 修改元素
print(aList)
print()

print("计数：")
aList = [1, 2, 3, 4, 7, 7, 8]
print(aList.count(7))   # 指定元素在列表中出现的次数
print(aList.count(0))
print()

aList = [1, 2, 3]
print("成员判断：")
print(3 in aList)
print(5 in aList)
print(5 not in aList)

```

### 1.3 切片

```python
aList = [1, 2, 3, 4, 5, 4, 9]

print("切片:")
print(aList[::])
print(aList[::-1])  # 步长为负数时，切片从后往前切
print(aList)
print(aList[2:6:2])     # 下标范围在[2, 6)且间隔为2
print(aList[3:])    # 得到下标>=3的元素

aList[len(aList):] = [89]     # 在尾部追加元素
print(aList)

aList[:3] = [77, 88, 99]    # 替换前3个元素
print(aList)

aList[:3] = []      # 删除前三个元素
print(aList)

aList = [1, 2, 3, 4, 5, 6]
del aList[:3]   # 结合del删除前3个元素
print(aList)
```

```python
aList = [3, 5, 7]
bList = aList[:]    # 切片，浅复制

print(aList)
print(bList)
print(aList == bList)   # bList和aList包含同样的元素引用
print(aList is bList)   # 不是同一个对象

bList[0] = 99   # 这个时候，bList里面的都是整型， 所以修改bList不影响aList
print(aList)
print(bList)

aList = [1, [88], 2]
bList = aList[:]

bList[1].append(99)     # 这里使用append方法改变bList会同时影响到aList，如果是直接通过复制，则结果会和上面的相同
print(aList)
print(bList)

# 下面的例子是深复制的例子
print("下面的例子是深复制的例子")
import copy
aList = [1, [88], 2]
bList = copy.deepcopy(aList)    # aList和bList完全独立，互相不影响

print(aList == bList)   # bList和aList包含同样的元素引用
print(aList is bList)   # 不是同一个对象
print(aList)
print(bList)

bList[1].append(99)     # 深复制，修改bList不会影响到aList
print(aList)
print(bList)
```

### 1.4 列表排序和逆序

```python
print("排序：")
aList = [1, 2, 6, 15, 256, 51, 8]

aList.sort()    # 默认升序排列
print("sort()升序")
print(aList)
print()

aList.sort(reverse=True)    # 降序排列
print("sort()降序")
print(aList)
print()

aList.sort(key=lambda x: len(str(x)))       # 按转换为字符串后的长度排序
print("sort()其他方式排序：转换为字符串后的长度排序")
print(aList)
print()

print("sorted()升序")
print(sorted(aList))    # sorted()默认升序，返回新列表,不会改变原列表

print("sorted()降序")
print(sorted(aList, reverse=True))  # 降序
print()

aList = [1, 2, 6, 5, 2, 5, 8]
aList.reverse()  # 原地逆序
print(aList)
print()
```

```python
# reversed方法返回逆序排列后的迭代对象，不会影响到原列表
aList = [3, 4, 5, 6, 7, 7, 8, 4, 6]
newList = reversed(aList)
print(newList)

print(aList)
print(list(newList))

print(list(newList))    # 访问过的元素不可以再次访问
```

### 1. 5 用于序列操作的常用内置函数

any()用来测试序列或可迭代对象中是否存在等价于 True 的元素

all()用来测试序列或可迭代对象中是否所有元素都等价于 True

```python
print("any():")
print(any([0, 1, 2]))   # any()用来测试序列或可迭代对象中是否存在等价于True的元素
print(any([0]))
print()

print("all():")
print(all([1, 2, 3, 4]))    # all()用来测试序列或可迭代对象中是否所有元素都等价于True
print(all([0, 1, 2, 3]))
```

sum()：对数值型列表的元素进行求和运算，对于非数值型列表需要指定第二个参数，适用于元组、集合、range 对象、字典、map 对象、filter 对象等。

```python
a = {
    1: 77,
    2: 88,
    3: 99
}

print(sum(a))       # 对字典的键进行求和
print(sum(a.values()))      # 对字典的值进行求和

print(sum([[1], [2], ["name"]], []))    # 非数值型，第二个参数需要指定
```

zip()方法、enumerate()方法参考 python(一)

### 1.5 列表推导式

列表推导式语法

```python
[exp for variable in iterable if condition]
```

列表推导式使用非常简洁的方式来快速生成满足特定需求的列表。（<b style="color: red">非常简洁，以至于有时候很难看懂，应多看多敲</b>）

几个例子

```python
aList = [x*x for x in range(10)]
print(aList)
print()
'''
相当于
aList = []
for x in range(10):
    aList.append(x * x)

print(aList)
'''

vec = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
print("使用列表推导式实现嵌套列表的平铺")
print(vec)
print([num for elem in vec for num in elem])    # 使用列表推导式实现嵌套列表的平铺
print()

print("在列表推导式中使用多个循环")
print([(x, y) for x in range(3) for y in range(3)])     # 在列表推导式中使用多个循环
```

## 2. 元组

### 2.1 元组的创建和删除

```python
myTuple = ()
print(myTuple)  # 空元组

myTuple = ('1', 'Hello', 2, True)
print(myTuple)

myTuple = 1, 2, 3, True, "Hello"   # 有逗号时可以没有括号
print(myTuple)

myTuple = (2)     # 不是元组，即使元组只有一个元素，元素后面也必须要带逗号，即myTuple = (2, )才对
print(myTuple)

myTuple = tuple('Hello World!')
print(myTuple)       # 通过tuple()函数可以把列表、字符串、集合等可迭代对象转为元组

del myTuple     # 只能通过del命令删除整个元组对象，不能删除只删除特定元素，因为元组属于不可变序列
```

### 2.2 元组和列表的区别

- 列表属于可变序列， 元组属于不可变序列
- 元组没有提供 append()、extend()、insert()和 remove()、pop()方法
- 元组不支持对元组元素进行 del 操作，只能用 del 命令删除整个元组
- 元组也支持切片操作，但只能通过切片来访问元组中的元素
- 元组的访问和处理速度比列表更快

<b style="color: red">元组属于不可变序列，其元素的值是不可改变的，但是元组中含有可变序列的话，可以通过调用可变序列的方法来改变可变序列的值。</b>

```python
myTuple = (1, [77, 88], 8)
print(myTuple)

myTuple[1].append(99)   # 调用元组里的可变序列的方法来改变可变序列的值
print(myTuple)

myTuple[1] += [111]  # 这样子相当于直接给可变序列赋值，即给元组的元素赋值，会报错
print(myTuple)
```

### 2.3 序列解包

```python
x, y, z = 1, 2, 3
print(x, y, z)

myTuple = (True, "Hello", 88)
(x, y, z) = myTuple

print(x, y, z)
```

序列解包用于列表和字典

```python
a, b, c = [4, 5, 6]
print(a, b, c)

# 下面是序列解包用于字典
s = {
    'Hello': 1,
    'World': 2,
    '!': 3,
}

a, b, c = s              # 序列解包对字典使用，默认是对字典"键"操作
print(a, b, c)

a, b, c = s.items()     # 使用字典的items()方法说明，是对字典"键:值"操作
print(a, b, c)

a, b, c = s.values()    # 使用字典的values()方法说明，是对字典"值"操作
print(a, b, c)
```

使用序列解包方便同时遍历多个序列。

```python
keys = ['a', 'b', 'c', 'd']
values = [1, 2, 3, 4]

for k, v in zip(keys, values):
    print(k, v)

```

可以加上一个或两个星号(\*)进行序列解包

```python
print(*[1], *[2, 3], 4, *[5, 6, 7])

print({*range(4), 5, *(7, 8, 9)})

print({
    'x': 1,
    **{'y': 2}
})
```

### 2.4 生成器表达式

```python
g = ((i + 2)**2 for i in range(10))     # 生成器表达式，和列表推导式非常接近
print(g)
print(list(g))      # 转化为列表
print(list(g))      # 元素已经遍历完了，所以输出空数组

g = ((i + 2)**2 for i in range(10))
print(g.__next__())     # 通过生成器对象的__next__()方法获取下一个元素
print(g.__next__())     # 生成器对象中的每个元素只能访问一次，所以两次输出不一样

print(next(g))      # 使用内置函数next()获取下一个元素

g = ((i + 2)**2 for i in range(10))
for i in g:     # 使用循环遍历生成器对象中的元素
    print(i, end=' ')

```

## 3. 字典

### 3.1 字典的创建与删除

```python
mydict = {
    'a': 1,
    'b': 2
}
print(mydict)

keys = ['a', 'b', 'c']
values = [1, 2, 3]
print(dict(zip(keys, values)))      # 通过内置函数dict()创建字典

mydict = dict()   # 空字典
print(mydict)    # 看着像集合，但是是字典
print(type(mydict))

mydict = {}     # 空字典
print(mydict)
print(type(mydict))

mydict = dict(name='clz', age=21, sex='male', address='QQ')   # 通过dict()键值对形式创建字典
print(mydict)

del mydict['age']
print(mydict)

print('----pop()----------')
print(mydict.pop('sex'))     # 删除指定"键"的元素，并返回对应被删除的值
print(mydict)

print('----popitem()------')
print(mydict.popitem())     # 删除并返回字典中的一个元素
print(mydict)

mydict.clear()      # 删除字典中的所有元素,字典变为空字典，不像del"连根拔起"
print(mydict)

```

### 3.2 字典元素的读取

```python
mydict = dict(name='clz', age=21, sex = 'male')   # 通过dict()键值对形式创建字典

print(mydict['name'])   # 直接使用字典的"键"作为下标访问字典元素的值,找不到会报错

print(mydict.get('age'))       # 使用字典对象的get()方法访问字典元素的值，找不到会返回None

print(mydict.keys())        # 返回字典的"键"

print(mydict.values())        # 返回字典的"值"

print(mydict.items())        # 返回字典的键值对

for key, value in mydict.items():      # 序列解包，遍历每个元素的"键"和"值"
    print(key, value, sep=':')
```

### 3.3 字典的添加与修改

```python
mydict = dict(name='clz', age=21, sex = 'male')   # 通过dict()键值对形式创建字典

mydict['name'] = 'czh'      # 当前已经有name这个键了，所以修改这个键的值
print(mydict)

mydict['address'] = 'QQ'    # 当前还没有address这个键，所以添加一个新元素
print(mydict)

mydict2 = dict(name='czh', age=22, sex = 'male', haha = 'Hello')
mydict.update(mydict2)   # 以调用这个方法的字典对象为准，把它没有的键对应的键值对加入到字典中
print(mydict)
```

## 4. 集合

特点：没有重复的元素

### 4.1 集合的创建和删除

```python
myset = {1, 2, 1, 4, 5, 2}
print(myset)

myset = set([1, 2, 3, 5, 2, 3])
print(myset)

myset = set()  # 空集合
print(myset)

myset.add(3)   # 使用add方法添加元素
print(myset)
myset.add(2)   # add()方法是在前面添加元素
myset.add(1)
print(myset)

myset.pop()
print(myset)   # pop()方法是弹出在最前面元素,不接受参数

myset.remove(3)  # remove()删除指定元素
print(myset)

```

### 4.2 集合运算

```python
aset = {1, 2, 3, 4, 5, 6}
bset = {3, 4, 5, 6, 7, 8}

print(aset | bset)      # 并集

print(aset & bset)      # 交集

print(aset - bset)      # 差集

print(aset ^ bset)      # 对称差集

aset = {1, 2, 3}
bset = {1, 2, 4}
print(bset < aset)
print(bset == aset)
print(bset > aset)   # 三个都是false，因为a集合和b集合没有关系

cset = {1, 2, 3, 5}
print(aset < cset)  # a集合是c集合的子集，所以是true
```

## 5. 复杂数据结构

### 5.1 堆

```python
# heapq模块默认建最小堆
import heapq
import random
data = list(range(10))
random.shuffle(data)        # 随机打乱顺序
print(data)

heap = []
for n in data:
    heapq.heappush(heap, n)     # 建堆

print(heap)

heapq.heappush(heap, 0.5)   # 新数据入堆
print(heap)
print(heapq.heappop(heap))  # 弹出最小的元素，堆会自动重建
print(heap)

myheap = [1, 2, 3, 4, 7, 4, 6, 8, 3, 9, 434]
heapq.heapify(myheap)  # 将列表转化为堆
print(myheap)

heapq.heapreplace(myheap, 6)  # 替换堆中的最小元素值，堆会自动重建
print(myheap)

print(heapq.nlargest(4, myheap))  # 返回堆中最大的4个元素(是数值最大，不是层数)，按从大到小排序
print(heapq.nsmallest(5, myheap))  # 返回堆中最小的5个元素，按从小到大排序

```

### 5.2 队列

```python
import queue
q = queue.Queue()  # 创建队列的实例

q.put(1)    # 入队
q.put(5)
q.put(3)
print(q.queue)

print(q.get())   # 队列头元素出队
print(q.queue)

'''
后进先出队列
'''
LiFoQueue = queue.LifoQueue(5)
LiFoQueue.put(1)
LiFoQueue.put(2)
LiFoQueue.put(3)

print(LiFoQueue.queue)
print(LiFoQueue.get())
print(LiFoQueue.queue)

'''
优先级队列
优先级队列,自动依照元素的权值排列。权值最高者排在最前面
没有去权值时，则按建堆来排列
'''
PriQueue = queue.PriorityQueue(5)
PriQueue.put(3)
PriQueue.put(5)
PriQueue.put(1)
PriQueue.put(7)
print(PriQueue.queue)

print(PriQueue.get())
print(PriQueue.queue)
```

### 5.3 栈

可以使用列表来实现栈，append()方法用于实现入栈操作，pop()方法用于实现出栈操作。

```python
mystack = []
mystack.append(1)
mystack.append(3)
mystack.append(5)

print(mystack)
print(mystack.pop())
print(mystack)

mystack.pop()
print(mystack)
mystack.pop()
print(mystack)
```

<b style="color: red">用列表来实现栈不足：1. 当列表为空时，再执行 pop()会抛出异常； 2. 无法限制栈的大小</b>

可以自己自定义栈结构来实现。

### 5.4 链表

可以使用列表来实现链表

```python
linkTable = []
linkTable.append(1)
linkTable.append(3)
print(linkTable)

linkTable.insert(1, 2)  # 正在链表中间插入节点
print(linkTable)

linkTable.remove(linkTable[2])   # 删除节点
print(linkTable)
```

### 5.5 二叉树

自定义二叉树

```python
class BinaryTree:
    def __init__(self, value):
        self.__left = None    # 和c++中的一开始让左孩子指向null类似
        self.__right = None
        self.__data = value

    def buildLeftChild(self, value):
        if self.__left:
            print('左孩子已经存在')
        else:
            self.__left = BinaryTree(value)
            return self.__left

    def buildRightChild(self, value):
        if self.__right:
            print('右孩子已经存在')
        else:
            self.__right = BinaryTree(value)
            return self.__right

    def show(self):
        print(self.__data)

    def preOrder(self):     # 前序遍历
        print(self.__data)
        if self.__left:
            self.__left.preOrder()
        if self.__right:
            self.__right.preOrder()

    def inOrder(self):
        if self.__left:
            self.__left.inOrder()
        print(self.__data)
        if self.__right:
            self.__right.inOrder()

    def postOrder(self):
        if self.__left:
            self.__left.postOrder()
        if self.__right:
            self.__right.postOrder()
        print(self.__data)
```

## 选择结构

之前没有学到的特别用法

```python
x = 5 if 2 > 1 else 6   # 后面不能写成x=6，只能是值的形式
print(x)

# 等价于
'''
if 2 > 1:
    x = 5
else:
    x = 6

print(x)
'''
```
