---
title: python(三)
date: 2021-10-29 10:09:13
categories: "Python"
tags:
  - Python
---

# python(三) 字符串与正则表达式

## 1 字符串

### 1.1 字符串格式化

语法：`'格式字符' %要格式化的内容`

```python
x = 16
print('%o' %x)
print('%x' %x)
print('%e' %x)
print('%d, %c' %(65, 65))   # 如果要求格式化的对象多于一个，要放在元组中
print('%s' %[1, 2, 3])
```

使用 format()方法进行格式化

```python
print('The number {0} in hex is: {0:#x}, the number {1:,} in oct is {1:#o}'.format(16, 1234))
# 字符串里的{}相当于占位符，把后面format()的参数放进去，按索引值放，即0代表放第一个参数，{0:#x}代表把第一个参数转换为16进制

print('My name is {name}, my age is {age}'.format(name = 'clz', age = 21))

position = (1, 2, 3)
print('x: {0[0]}, y:{0[1]}, z: {0[2]}'.format(position))

print('{0:.4f}'.format(1.23456))    # 指定小数位数

print('{0:.2%}'.format(1/3))        # 格式化为百分数

print('{0:^10.2%}'.format(1/3))     # ^表示居中对齐，<表示左对齐，>表示右对齐

```

从 Python 3.6.x 开始支持一种新的字符串格式化方法，官方叫做 Formatted String Literals, 简称为 f-字符串,在 Python 3.8 之后的版本中，支持 print(f'{width=}')形式的用法++9+++

```python
w = 1
h = 2
print(f'width is {w}, height is {h}')
```

### 1.2 字符串常用方法

**find()、rfind()、index()、rindex()、count()**：

```python
s = 'apple, peach, banana, peach'
print(s.find('peach'))  # 返回第一次出现的位置
print(s.find('peach', 8))  # 从指定位置开始查找
print(s.find('peaach', 6, 24))  # 在指定范围中查找， 查找的字符串要完全在范围内才可以找到，只有开始或结束在范围内也会返回-1

print(s.rfind('p'))   # rfind()是从后往前找

print(s.index('p'))     # 返回首次出现的位置
print(s.index('pe'))

# print(s.index('ppp'))      # 找不到会抛出异常

print(s.count('p'))     # 计算子字符串出现的次数
```

**split()、rsplit()、partition()、rpatition()**：

```python
s = 'a, b, c, d, e, hello'
li = s.split(',')   # 把字符串按指定分隔符分割，变成列表
print(li)

li = s.partition(',')   # 将字符串按指定的分隔符分割成三部分，分隔符前的字符串，分隔符，和分割符后的字符串，以元组形式
print(li)
li = s.rpartition(',')
print(li)

# split()和rsplit()方法，如果不指定分隔符，则字符串中的任何空白符号(包括空格、换行符，制表符等)都会被认为是分隔符
s = 'hello\t\tworld   I am\n\n\nclz'
print(s.split())
print(s.split(None, 2)) # 指定最大分隔次数， None相当于不指定分隔符，即分隔符会是所有的空白字符
```

**join()**：效率比直接使用'+'拼接高

```python
# join()方法是将列表或其他可迭代对象中的字符串以指定的字符串连接， 如果当中含有非字符串的元素，则会抛出异常
li = ['a', 'b', 'c', 'apple', 'True']
sep = '::'
print(sep.join(li))
```

**lower()、upper()、capitalize()、title()、swapcase()**：

```python
s = 'Hello World, hello python'
print(s.lower())    # 转换成小写
print(s.upper())    # 转化成大写
print(s.capitalize())   # 字符串首字母大写
print(s.title())        # 每个单词首字母大写
print(s.swapcase())     # 大小写互换
```

**replace()**：

```python
words = ('非法', '话', '暴力')
text = '这句话有非法内容'

for word in words:
    if word in text:
        text = text.replace(word, '**')

print(text)
```

**maketrans()、translate()**

```python
table = ''.maketrans('0123456789', '零一二三四五六七八九')    # 生成字符映射表，可以搭配translate()使用，把字符串中第一个参数内的字符转换为对应第二个参数中的字符(转换过程是单向的)
s = '2021年10月1日'
print(s.translate(table))
s = '二零二一年一零月一日'
print(s.translate(table))
```

**strip()、rstrip()、lstrip()**：

```python
s = '  hello \n'
print(len(s))
print(s)

print(len(s.strip()))   # 删除两端空白字符
print(s.strip())

print(len(s.rstrip()))
print(s.rstrip())       # 删除右端空白字符

s = 'aabbccddffee'
print(s.lstrip('ab'))   # 删除左端指定字符, 是按字符依次进行删除的，即先删除左端字符a，然后再删除左端字符b
print(s.lstrip('ac'))   # 删除左端字符a后，因为c不在左端，所以不会删除c
```

**eval()**：

```python
print(eval('1 + 1'))    # eval()把任意字符串转换为Python表达式并求值

a = 3
b = 4
print(eval('a + b'))
```

使用 eval()需要注意的问题：它可以计算任何合法表达式的值，即用户可以用特殊的字符串进行攻击

**in**：

使用关键字来判断一个字符串是否在另一个字符串中

```python
print('abc' in 'aabbcc')    # False
print('abc' in 'abcabc')    # True
```

**startswith()、endswith()**：

```python
s = "Hello World!"
print(s.startswith('He'))   # 用来判断字符串是否以指定字符串开始
print(s.startswith('He', 5))    # 可以接收整数参数来限定字符串的检测范围
print(s.startswith('He', 0, 3))
```

其他方法：

```python
print('1234abcd'.isalnum())
print('1234abcd'.isalpha())
print('1234abcd'.isdigit())
print('abcd'.isalpha())
print('1234.053'.isdigit())     # 用来判断字符串是不是数字字符，小数点.不是数字字符，所以返回False
print('00123400'.isdigit())
print()

print('Hello world!'.center(20))
print('Hello world!'.center(20, '='))
print('Hello world!'.ljust(20, '='))
print('Hello world!'.rjust(20, '='))
```

### 1.3 字符串常量

```python
import string
print('数字: ', string.digits)
print('字母: ', string.ascii_letters)
print('特殊符号: ', string.punctuation)

# 下面是生成8位长度的随机密码的例子

x = string.digits + string.ascii_letters + string.punctuation

import random
print(''.join([random.choice(x) for i in range(8)]))   # random.choice() 方法返回一个列表，元组或字符串的随机项

print(''.join(random.sample(x, 8)))
```

## 2 正则表达式

正则表达式使用预定义的特定模式去匹配一类具有共同特征的字符串，主要用于字符串处理，可以快速、准确地完成复杂的查找、替换等处理任务。

### 2.1 直接使用 re 模块函数

```python
import re       # 在Python中，主要使用re模块来实现正则表达式的操作
text = 'alpha,beta,gamma,delta'
print(re.split('[,]+', text))       # 根据匹配项分割字符串

pat = '[a-zA-Z]+'
print(re.findall(pat, text))    # 在text中找符合pat的单词

pat = '{name}'
text = 'Dear {name}'
print(re.sub(pat, 'clz', text))    # 将text中的pat匹配项用第二个参数替换
s = 'a s d'
print(re.sub('a|s|d', 'test', s))   # 将s中的a、s、d替换成test
print(s)                            # 不会改变原字符串

print(re.escape('http://www.python.org'))   # 字符串转义

print(re.match('a|b', 'acgs'))     # re.match()在字符串的开始处匹配模式，匹配成功
print(re.match('a|b', 'cde'))       # 匹配不成功，返回None

print(re.match('done|quit', 'testdone'))    # 是在字符串的开始处进行匹配模式，所以会匹配不成功
print(re.search('done|quit', 'testdone'))   # re.search()在整个字符串中寻找模式，所以会匹配成功

text = '''good
bad
test
clz'''
print(re.findall(r'\w+', text))			# r用来阻止转义

print(re.findall(r'^\w+$', text))   # \w不能匹配换行符,即有换行就会返回空列表，包括\n形式的换行
print(re.findall(r'^.+$', text, re.S))      # 单行模式，此时.可以匹配换行符，会把换行符变为\n
print(re.findall(r'^.+$', text, re.M))      # 多行模式，会把每一行变为列表中的元素

```

### 2.2 使用正则表达式对象

使用正则表达式对象的用法和正常使用 re 模块基本一样，首先通过 re 模块的 compile()函数将正则表达式编译生成正则表达式对象，然后再使用正则表达式对象提供的方法进行字符串处理。

```python
import re
example = 'ShanDong Institute of Business and Technology is a very beautiful school.'
pattern = re.compile(r'\bB\w+\b')   # 以B开头的单词
'''
\b表示匹配单词头或单词尾
\w表示匹配任何字母、数字以及下划线
+表示匹配位于+之前的字符或子模式的1次或多次重复
'''
print(pattern.findall(example))

pattern = re.compile(r'\w+g\b')     # 以g结尾的单词
print(pattern.findall(example))

pattern = re.compile(r'\b[a-zA-Z]{3}\b')    # 3个字母长的单词
print(pattern.findall(example))

pattern = re.compile(r'\b\w*a\w*\b')        # 查找所有含有字母a的单词
print(pattern.findall(example))
```

### 2.3 子模式与 Match 对象

```python
import re
telNumber = '''Suppose my Phone No. is 0535-1234567,
yours is 010-12344567, his is 025-123456789'''
pattern = re.compile(r'(\d{3,4})-(\d{7,8})')    # 这里的字符串格式要注意：如,后不能跟空格，否则匹配不上。看了好久都没发现问题，最后才发现是自己的代码规范导致的
print(pattern.findall(telNumber))

email = 'tony@tiremove_thisger.net'
m = re.search('remove_this', email)     # 利用re.search()返回的Match对象来删除字符串指定内容
print(email[:m.start()] + email[m.end():])
'''
Match对象的start()方法: 返回指定子模式内容的起始位置
Match对象的end()方法: 返回指定子模式内容的结束位置的下一个位置
'''

m = re.match(r'(\w+) (\w+)', 'Isaac Newton, physicist')
print(m.group(0))       # 返回整个模式内容
print(m.group(1))       # 返回第1个子模式内容
print(m.group(2))       # 返回第2个子模式内容
print(m.group(1, 2))    # 返回指定的多个子模式的内容，元组形式
'''
group(): 返回匹配的一个或多个子模式内容
'''

m = re.match(r'(\d+)\.(\d+)', '12.3456')
print(m.groups())
'''
groups(): 返回一个包含匹配的所有子模式内容的元组
'''

m = re.match(r'(?P<first_name>\w+) (?P<last_name>\w+)', 'Malcolm Reynolds')     # ?P<groupname>: 为子模式命名
print(m.groupdict())
```

练习理解：使用正则表达找出 AABC 和 ABAC 类型的成员

![](https://pic.imgdb.cn/item/616e4cdb2ab3f51d918f17bd.jpg)

首先，直接给出两个做法的答案

**做法 1**：

```python
pattern = r'((.).\2.|(.)\3..)'
for item in findall(pattern, text):
    print(item[0])
```

**做法 2**：

```python
pattern = r'(\b(?P<F>\w)\w(?P=F)\w\b|\b(?P<D>\w)(?P=D)\w\w\b)'
for item in findall(pattern, text):
    print(item[0])
```

做法 1 讲解：

1. 首先，有一个会用到的重要概念，<b style="color: red">使用括号表示一个子模式</b>

2. 先 ABAC 类型(实际上 ABAB 也能匹配到，书上的也是这样，将错就错了，简单点)，小数点表示能匹配到除换行符的任意单个字符，所以先构建

   正则表达式` pattern = r'(..)'`

3. 第三个应该要和第一个相同，所以不能直接`pattern = r'(...)'`, 这个时候就要用上正则表达式的复制粘贴功能了，首先，做好复制工作--**用括号把要复制的部分包住**，` pattern = r'((.).)'`, 然后是粘贴工作--反斜线加要粘贴的内容是第几个子模式，` pattern = r'((.).\2)'`,这里是 2 的原因就是上面说的重点了，<b style="color: red">使用括号表示一个子模式</b>，我们要把第二个括号里的东西复制粘贴，所以自然就是 2 了

4. 第四个随便，所以正则表达式变成` pattern = r'((.).\2.)'`

5. 再 AABC 类型的，先加上'|'，`pattern = r'((.).\2.|)'`,<b style="color: red">这里的|左右不能有空格</b>

6. 之后的原理和上面的相同，需要注意的是：<b style="color: red">括号并不会重置为 0，才开始算，而是前面的括号也算</b>，所以变成` pattern = r'((.).\2.|(.)\3..)'`

7. 之后通过循环即可得到结果，因为 findall()是找出所有的匹配项，所以只需要 item[0]就行了

做法 2 讲解：

1. 首先原理和做法 1 一样，不同的是复制粘贴的形式，做法 2 是先通过` (?P=<groupname>)`给要复制的内容命名，然后通过` (?P=groupname)`把内容粘贴过去
