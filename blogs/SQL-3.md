---
title: 数据库学习笔记(三)
date: 2021-11-06 21:39:32
keywords: SQL 数据库完整性 数据库
categories: "SQL"
tags:
  - SQL
---

# 数据库学习笔记(三)

## 1. 数据库完整性

### 1.1 概述

- **数据的正确性**：指数据符合现实世界语义，反映了当前实际情况
- **数据的相容性**：指数据库同一对象在不同的表中的数据是符合逻辑的

#### 数据的完整性和安全性

是两个不同概念

- **数据的完整性**
  - 防止数据库中存在不符合语义的数据，也就是防止数据库中存在不正确的数据
  - 防范对象：不合语义的、不正确的数据
- **数据的安全性**
  - 保护数据库，防止恶意的破坏和非法的存取
  - 防范对象：非法用户和非法操作

### 1.2 实体参照性

#### 1.2.1 实体完整性定义

```sql
create table Student
( Sno char(9) primary key,		# 在列级定义主码
  Sname char(20) not null,
  Ssex char(2)
);

create table Student
( Sno char(9),
  Sname char(20) not null,
  Ssex char(2),
  primary key(Sno)			   # 在表级定义主码
);

# 如果是要将属性组定义为主码，只能在表级定义主码
create table SC
( Sno char(9),
  Cno char(4),
  Grade smallint,
  primary key(Sno, Cno)		# 属性组定义为主码，只能在表级定义主码
);
```

### 1.3 参照完整性

#### 1.3.1 参照完整性定义

- 在建表时用<b style="color: red">` foreign key`</b>短语定义哪些列是外码
- 用<b style="color: red">` references`</b>短语指明这些外码参照哪些表的主码

```sql
create table SC
( Sno char(9) not null,
  Cno char(4) not null,
  Grade smallint,
  primary key(Sno, Cno),
  foreign key(Sno) references Student(Sno),
  foreign key(Cno) references Course(Cno)	# 在表级定义参照完整性
);
```

参照完整性检查示例：

破坏参照完整性：

**SC 表增加一个元组**，该元组的 Sno 属性的值在表 Student 中找不到一个元组，能让其 Sno 属性的值与之相等

![](https://pic.imgdb.cn/item/617a75162ab3f51d91f54fab.jpg)

### 1.4 用户定义的完整性

**用户定义的完整性**：针对某一具体应用的数据必须满足的语义要求

#### 1.4.1 属性上的约束条件

建表时定义属性上的约束条件

- 列值非空(not null)
- 列值唯一(unique)
- 检查列值是否满足一个条件表达式(check)

1. 不允许取空值

   ```sql
   create table SC
   ( Sno char(9) not null,
     Cno char(4) not null,
     Grade smallint not null
     primary key(Sno, Cno)		# Sno、Cno、Grade属性不允许取空值
   )
   ```

2. 列值唯一

   ```sql
   create table DEPT
   ( Deptno numeric(2),
     Dname char(9) unique not null,	# 要求Dname列值唯一，并且不能取空值
     primary key(Deptno)
   )
   ```

3. 用 check 短语指定列值应该满足的条件

   ```sql
   create table Student
   ( Sno char(9) primary key,
     Sname char(8) not null,
     Ssex char(2) check(Ssex in ('男', '女'))  # Ssex只允许是'男'或'女'
   )
   ```

#### 1.4.2 元组上的约束条件

建表时用**check**短语定义元组上的约束条件，即**元组级的限制**

```sql
create table Student
( Sno char(9),
  Sname char(8) not null,
  Ssex char(2),
  primary key(Sno),
  check(Ssex='女' or Sname not like 'MS.%')		# 当学生的性别是男时，名字不能以MS.开始
)
```

### 1.5 完整性约束命名子句

#### 1.5.1 完整性约束命名子句

` constraint <完整性约束条件名> <完整性约束条件>`

- 完整性约束条件包括 not null、unique、primary key 短语、foreign key 短语、check 短语等

```sql
create table Student
( Sno numeric(6)		# 下面有完整性约束命名子句，所以没有逗号
  constraint C1 check (Sno between 9000 and 9999),	# 要求学号在9000~9999之间
  Sname char(20),
  constraint C2 not null,
  Sage numeric(3)
  constraint C3 check(Sage < 30),
  constraint StudentKey primary key(Sno)		# 主码约束，上面的C1、C2、C3都是列级约束
)
```

#### 1.5.2 修改表中的完整性限制

- 使用 alter table 语句修改表中的完整性限制

```sql
alter table Student
drop constraint C3;		# 删除对年龄的限制
```

- 修改表的约束条件

  ```sql
  # 修改表的约束条件方法：先删除原来的约束条件，再增加新的约束条件
  alter table Student
  drop constraint C1;
  alter table Student
  add constraint C1 check(Sno between 1000 and 9999);
  ```

### 1.6 断言

- 在 SQL 中，可以使用 create assertion 语句，通过声明断点来指定更具一般性的约束
- 断言创建之后，任何对断言中涉及的关系的操作都会触发关系数据库管理系统对断言的检查，任何使断言不为真值的操作都会被拒绝执行

#### 1.6.1 创建断言的语句格式

` create assertion <断言名> <check子句>`

```sql
create assertion ASSE_SC_CNUM1
check(60 >= all(select count(*) from SC
                group by Cno)
     );		# 限制每一门课程最多60名学生选修
```

```sql
# 限制每个学期每一门课程最多60名学生选修
# 先增加一个学期属性
alter table SC add Team;

# 定义断言
create assertion ASSE_SC_CNUM2
check(60 >= all(select count(*) from SC
                group by Cno, Team)
     );
```

#### 1.6.2 删除断言

` drop assertion <断言名>`

### 1.7 触发器

**触发器**：用户定义在关系表上的一类由**事件驱动**的特殊过程

#### 1.7.1 定义触发器

概念太多，建议直接实践

```sql
# 当对表SC的Grade属性进行修改时，若分数增加了10% 则将这次的操作记录下来
create trigger SC-T
after update of Grade on SC		# 分数发生修改后激活触发器
referencing
	old row as OldTuple			# 把引发事件之前的值改名为OldTuple
	new row as NewTuple			# 把引发事件之后的值改名为NewTuple
for each row		# 行级触发器，即每执行一次Grade的更新，下面的规则就会执行一次
when(NewTuple.Grade >= 1.1 * OldTuple.Grade)
	insert into SC_U(Sno, Cno, OldGrade, NewGrade)	# 满足条件，把这次的操作记录下来
	values(OldTuple.Sno, OldTuple.Cno, OldTuple.Grade, NewTuple.Grade)
```

```sql
# 将每次对表Student的插入操作所增加的学生个数记录下来
create trigger Student_Count
after insert on Student
referencing
	new table as Delta
for each statement
	insert into StudentInsertLog(Numbers)
	select count(*) from Delta
```

```sql
# 定义一个BEFORE行级触发器，为教师表Teacher定义完整性规则“教授的工资不得低于4000元，如果低于4000元，自动改为4000元
create trigger Insert_Or_Update_Sal
before insert or update on Teacher		# 插入工资前，或者更改工资前激活触发器
for each row
begin		# 定义触发动作体，是一个PL/SQL过程块
	if (new.Jog = '教授') and (new.Sal < 4000)
	then new.Sal := 4000;
	end if;
end;		# 触发动作体结束
```

#### 1.7.2 激活触发器

- 触发器的执行，是由**触发事件激活**的，并由数据库服务器自动执行

  触发器执行时顺序

  1. 执行该表上的 before 触发器
  2. 激活触发器上的 SQL 语句
  3. 执行该表上的 after 触发器

#### 1.7.3 删除触发器

` drop trigger <触发器名> on <表名>;`
