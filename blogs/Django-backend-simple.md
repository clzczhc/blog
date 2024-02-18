---
title: Django实现简单的增删改查
categories: 后端
date: 2023-01-21 11:02:57
tags:
    - Django
    - Python
---

# Django实现简单的增删改查

## 前言

之后的项目可能会用到Django来编写后端，提前学习熟悉一下。

> 项目启动不起来的话，查看下面的文章来配置参数
> 
> [Pycharm项目启动参数配置](https://blog.csdn.net/qq_42701659/article/details/124635133)
> 
> ![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202301241328910.png)

## 创建Django项目

社区的Pycharm没有办法直接创建Django项目，所以需要通过命令行创建，再使用Pycharm打开。

[社区版pycharm创建django项目 - 冰箱喵 - 博客园](https://www.cnblogs.com/keqing1108/p/15986644.html)

## Django ORM

Django自带ORM（对象关系映射）。可以通过描述对象和数据库之间的映射，将程序中的对象自动持久化到数据库中。

![](https://www.runoob.com/wp-content/uploads/2020/05/django-orm1.png)

ORM解析过程：

1. ORM会将Python代码转换成SQL语句

2. pymysql将SQL语句发送到数据库服务端

3. 在数据库中执行SQL语句并返回结果

![](https://www.runoob.com/wp-content/uploads/2020/05/orm-object.png)

## 准备操作

通过上面的链接创建好Django项目后，文件结构应该是类似下面的：

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202301211203466.png)

因为Django的ORM没办法操作数据库，所以需要自己启动数据库，通过命令行来创建数据库。

```shell
create database user default charset=utf8;
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202301211334548.png)

修改设置文件`setting.py`中的`DATABASES`配置项，将信息改成对应的数据库信息

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql', # 数据库引擎
        'NAME': 'User',      # 数据库地址
        'HOST': '127.0.0.1', # 数据库地址
        'PORT': 3306,
        'USER': 'root', # 数据库用户名
        'PASSWORD': '123456'    
    }
}
```

修改`__init__.py`文件，引入`pymysql`模块连接数据库（需要先通过pip之类的工具安装好）

```python
import pymysql

pymysql.install_as_MySQLdb()
```

修改`models.py`文件，创建数据表对应的Models类。

```python
from django.db import models

# Create your models here.
class User(models.Model):
    name = models.CharField(max_length=20)
    age = models.IntegerField()
```

添加对应的`Models`类后，还需要在`setting.py`中的`INSTALLED_APPS`中添加上去。

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'testBackend'
]
```



通过命令行来创建表结构

> * `python manage.py migrate`
> 
> * `python manage.py makemigrations testBackend`
> 
> * `python manage.py migrate testBackend`

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202301241340017.png)



`urls.py`配置路由

```python
from django.urls import path

from . import testdb

urlpatterns = [
    path('testdb/', testdb.testdb),
]
```



新建文件`testdb.py`，该文件就是用来操作数据的。

## 数据库操作

> $\color{red}{记得启动数据库}$ 

### 添加数据

* 从`testBackend.models`中引入对应的`Models`类

* 创建对应的`Models`对象

* 调用`save`方法添加数据



testdb.py

```python
from django.http import HttpResponse

from testBackend.models import User

def testdb(request):
    user = User(name='clz', age=22)
    user.save()
    return HttpResponse("<p>添加数据成功</p>")
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202301241341269.png)



### 获取数据

* 通过`User.objects.all()`获取所有数据

* 之后遍历数据，拿到想要的部分

```python
from django.http import HttpResponse

from testBackend.models import User

def testdb(request):
    result = ""

    users = User.objects.all()

    for user in users:
        result += f'<p>' \
                  f'<span> id: {user.id}</span>' \
                  f'<span> name: {user.name}</span>'\
                  f'<span> age: {user.age}</span>' \
                  f'</p>'

    return HttpResponse("<p>" + result + "</p>")
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202301241444305.png)



当然，获取数据并不是只有`all()`方法，还可以使用`filter()`方法设置过滤条件、`get()`方法获取单个对象、`order_by()`对数据进行排序等。($\color{red}{方法支持链式调用}$)



* `filter()`

```python
users = User.objects.filter(name='czh')
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202301241838159.png)



* `get()`

```python
user = User.objects.get(id=2)  # 如果得到的结果不只是一个对象会报错

result += f'<p>' \
          f'<span> id: {user.id}</span>' \
          f'<span> name: {user.name}</span>'\
          f'<span> age: {user.age}</span>' \
          f'</p>'
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202301241844624.png)



* `order_by()`

```python
users = User.objects.order_by("age")
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202301241846959.png)



* 链式调用

```python
# filter(id__lte = 33)意味着 <=，
# 对应SQL：select * from User where id <= 33
users = User.objects.filter(id__lte = 33).order_by('age')
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202301241855968.png)



还可以配合Python的切片使用

```python
# filter(id__lte = 33)意味着 <=，
# 对应SQL：select * from User where id <= 33
users = User.objects.filter(id__lte = 33).order_by('age')
users = users[2:]
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202301241857339.png)



更多：[django 之 select filter查询](https://blog.csdn.net/YL3126/article/details/103717946)



### 修改数据

修改数据主要有两种方式：

1. > - 通过`get()`方法获取唯一的一个对象实例
   > 
   > - 修改该对象实例的属性
   > 
   > - 调用`save()`方法
   
   ```python
   def testdb(request):
       result = "修改前："
   
       users = User.objects.all()
   
       for user in users:
           result += f'<p>' \
                     f'<span> id: {user.id}</span>' \
                     f'<span> name: {user.name}</span>'\
                     f'<span> age: {user.age}</span>' \
                     f'</p>'
   
       updateuser = User.objects.get(id=1)
       updateuser.age = 999
       updateuser.save()
   
       result += "修改后："
   
       users = User.objects.all()
   
       for user in users:
           result += f'<p>' \
                     f'<span> id: {user.id}</span>' \
                     f'<span> name: {user.name}</span>' \
                     f'<span> age: {user.age}</span>' \
                     f'</p>'
   
       return HttpResponse("<p>" + result + "</p>")
   ```
   
   ![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202301241947984.png)

2. > * 通过`filter()`等方法获取符合修改条件的对象$\color{red}{集合}$
   > 
   > * 调用`update()`方法修改
   
   ```python
   User.objects.filter(id=1).update(age=888)
   ```
   
   ![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202301241948774.png)

### 删除数据

删除数据只需要调用对象的`delete()`方法即可，对象集合调用`delete()`方法也可以

```python
deleteuser = User.objects.get(id=1)
deleteuser.delete()
```



```python
User.objects.filter(id=2).delete()
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202301241952325.png)



## 参考

[Django 模型 | 菜鸟教程](https://www.runoob.com/django/django-model.html)

[社区版pycharm创建django项目 - 冰箱喵 - 博客园](https://www.cnblogs.com/keqing1108/p/15986644.html)

[django 之 select filter查询](https://blog.csdn.net/YL3126/article/details/103717946)
