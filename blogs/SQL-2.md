---
title: 数据库学习笔记(二)
date: 2021-10-29 10:10:35
categories: "SQL"
tags:
  - SQL
---

# 数据库学习笔记(二)

## 1. 数据库安全性

### 1.1 TCSEC/TDI 安全级别划分

![](https://pic.imgdb.cn/item/6170227f2ab3f51d91e74c50.jpg)

<b style="color: red">B1 级："安全"或"可信的"产品 ，真正意义上的安全产品</b>

### 1.2 数据库管理系统安全性控制模型

![](https://pic.imgdb.cn/item/617023a82ab3f51d91e8829f.jpg)

**存取控制流程**：

1. 数据库管理系统对提出 SQL 访问请求的数据库用户进行身份鉴别，防止不可信用户使用系统
2. 在 SQL 处理层进行自主存取控制和强制存取控制，甚至还可以进行推理控制
3. 还可以对用户访问行为和系统关键操作进行审计，对异常用户行为进行简单入侵检测

### 1.3 数据库安全性控制的常用方法

1. 用户标识和鉴定
2. 存取控制
3. 视图
4. 审计
5. 数据加密

### 1.4 数据库安全性控制

#### 1.4.1 用户身份鉴别

- 静态口令鉴别
- 动态口令鉴别
- 生物特征鉴别
- 智能卡鉴别

#### 1.4.2 存取控制

<b style="color: red">自主存取控制</b>:

- C2 级
- 用户对不同的数据对象有不同的存取权限
- 不同的用户对同一对象有不同的权限
- 用户可以将其拥有的存取权限授予其他用户

<b style="color: red">强制存取控制</b>:

- B1 级
- 每一个数据对象被标以一定的密级
- 每个用户也会被授予某个级别的许可证
- 对于每一个对象，只有具有合法许可证的用户才可以存取

#### 1.4.3 授权: 授予与回收

1. grant

   格式：

   ```sql
   grant <权限>[, <权限>]...
   on <对象类型> <对象名>[, <对象类型> <对象名>]...
   to <用户>[, <用户>]...
   [with grant option]	# 指定这个的话，表示权限可以再授予，没有指定的话，则不行
   ```

   例子：

   ```sql
   grant select
   on table Student
   to U1;	# 把查询Student表的权限授予用户U1

   grant all priviliges
   on table Student, Course
   to U2, U3; # 把Student表和Course表的全部权限授予用户U2和U3

   grant select
   on table Student
   to public;	# 把查询Student表的权限授予所有用户

   grant update(Sno), select
   on table Student
   to U4	# 把查询Student表和修改学号的权限授予用户U4
   with grant option; # 允许用户U4再把此权限授予其他用户

   grant select
   on table Student
   to U5;	# 用户U4可以把自己的select权限授予U5, U5不能再传播此权限
   ```

2. revoke

   格式：

   ```sql
   revoke <权限>[, <权限>]...
   on <对象类型> <对象名>[, <对象类型> <对象名>]...
   from <用户>[, <用户>]...[cascade | restrict]
   ```

   例子：

   ```sql
   revoke select
   on table Student
   from public； # 收回所有用户对标Student的查询权限

   revoke select
   on table Student
   from U4 cascade; # 将用户U5的select权限收回的时候，需要的使用cascade，否则拒绝执行该语句， 因为用户U4已经把select权限授予给U5了，所有收回U4的select权限的时候需要级联，收回用户U5的权限，因为U5的权限来自U4,U4没了，U5自然也会没
   ```

   **数据库管理员**：

   - 拥有所有对象的所有权限
   - 根据实际情况将不同的权限授予不同用户

   **用户**：

   - 拥有自己建立的对象的全部的操作权限
   - 可以使用 grant，把权限授予其他用户

   **被授权的用户**：

   - 如果具有"继续授权"的许可，则可以把获得的权限再授予其他用户

3. 创建数据库模式的权限

   - 数据库管理员在创建用户时实现

   - 语法：

     ```sql
     create user <username>
     [with][DBA|RESOURCE|CONNECT];
     ```

     **CONNECT 权限**：拥有 CONNECT 权限的用户不能创建新用户、模式、基本表，只能登录数据库

     **RESOURCE 权限**：能创建基本表和视图，但不能创建模式、新用户

     **DBA 权限**：可以创建新用户、创建模式、创建基本表和视图。DBA 拥有对所有数据库对象的存取权限，还可以吧这些权限授予一般用户

#### 1.4.4 数据库角色

**数据库角色**：被命名的一组与数据库操作相关的权限

- 角色是权限的集合
- 可以为一组具有相同权限的用户创建一个角色，用来简化授权的过程

1. **角色的创建**

   ```sql
   create role <角色名>;
   ```

2. 给角色授权

   ```sql
   grant <权限>[, <权限>]...
   on <对象类型> <对象名>
   to <角色>[, <角色>]...
   ```

3. 将一个角色授予其他的角色或用户

   ```sql
   grant <角色1>[, <角色2>]...
   to <角色3>[, <用户1>]...
   [with admin option] # 指定with admin option的话，可以把权限授予其他角色或用户
   ```

4. 角色权限的收回

   ```sql
   revoke <权限>[, <权限>]...
   on <对象类型> <对象名>
   from <角色>[, <角色>]...
   ```

例子：

```sql
create role R1;  # 创建一个角色R1

grant select, update, insert
on table Student
to R1;			# 给角色R1授予Student表的select、update、insert权限

grant R1
to clz, czh;	# 将角色R1授予clz、czh，使他们具有角色R1所包含的全部权限

revoke R1
from clz;		# 通过R1一次性收回clz的这3个权限

grant delete
on table Student
to R1;			# 使角色R1在原来的基础上增加了Student表的delete权限

revoke select
on table Student
from R1;		# 收回R1的select权限
```

#### 1.4.4 强制存取控制方法

**强制存取控制(MAC)**:

- 保证更高程度的安全性
- 用户不能直接感知或进行控制
- 适用于对数据有严格而固定密级分类的部门

在强制存取控制中，数据库管理系统所管理的全部实体被分为主体和客体两大类

**主体**是系统中的活动实体

- 包括数据库管理系统所管理的实际用户
- 也包括代表用户的各个进程

**客体**是系统中的被动实体，受主体控制

- 文件、基本表、索引、视图等

**敏感度标记**

- 对于主体和客体，DBMS 为它们每一个实例指派一个敏感度标记

- 敏感度标记可分为若干级别

  - 绝密(TS)
  - 机密(S)
  - 可信(C)
  - 公开(P)

  **TS>=S>=C>=p**

- 主体的敏感度标记称为许可证级别

- 客体的敏感度标记称为密级

强制存取控制规则：

- 仅当主体的许可证级别<b style="color: red">大于或等于</b>客体的密级时，该主体才能<b style="color: red">读</b>相应的客体
- 仅当主体的许可证级别<b style="color: red">小于或等于</b>客体的密级时，该主体才能<b style="color: red">写</b>相应的客体

**自主存取控制与强制存取控制共同构成数据库管理系统的安全机制**

- 先进行自主存取控制检查(DAC 检查)，通过自主存取控制检查的数据对象再由系统进行强制存取控制检查(MAC 检查)，只有通过 MAC 检查的数据对象才可以存取。

### 1.5 视图机制

- 把要保密的数据对无权存取这些数据的用户隐藏，对数据提供一定程度的安全保护
- 间接实现支持存取谓词的用户权限定义

例子：

```sql
create view CS_Student
as
select * from Student
where Sdept = 'CS';	# 建立计算机系学生的视图

grant select
on CS_Student
to clz;

grant all priviliges
on CS_Student
to czh;				# 在视图上进一步定义存取权限
```

### 1.6 审计

概念：

- 启用一个专用的审计日志，将用户对数据库的所有操作记录在上面
- 审计员可以利用审计记录，监控数据库中的各种行为，找出非法存取数据的人、时间和内容
- **C2 以上安全级别的 DBMS 必须具有审计功能**

<b style="color: red">应该只有保密级别高的才需要设置审计功能，因为审计很费时间和空间，会增加很多开销</b>

**用户级审计**：

- 任何用户可设置的审计
- 主要是用户针对自己创建的数据库表和视图进行审计

**系统级审计**：

- 只能由数据库管理员设置
- 可以监测成功或失败的登录请求、监测授权和收回操作以及其他数据库权限下的操作

#### 1.6.1 audit 语句

**audit 语句**：设置审计功能

```sql
audit alter, update
on Student;		# 对修改Student表结构或修改表数据的操作进行审计
```

#### 1.6.2 noaudit 语句

**noaudit 语句**：取消审计功能

```sql
noaudit alter, update
on Student;		# 取消对Student表的审计
```

### 1.7 数据加密

#### 1.7.1 存储加密

**透明存储加密**：

- 内核级加密保护方式对用户完全透明
- 将数据在写到磁盘时对数据进行，授权用户读取数据时再对其进行解密
- 数据库的管理应用程序不需要做任何修改，只需要在创建表语句中说明需加密的字段就可以

<b style="color: red">性能较好，安全完备性较高</b>

**非透明存储加密**：

- 通过多个加密函数实现

#### 1.7.2 传输加密

**链路加密**：

- 在链路层进行加密
- 传输信息由报头和报文两部分组成
- 报文和报头都加密

**端对端加密**：

- 在发送端加密，接收端解密
- 只加密报文，不加密报头
- 容易被非法监听者发现并从中获取敏感信息

![](https://pic.imgdb.cn/item/6170cfe42ab3f51d915fbfe3.jpg)
