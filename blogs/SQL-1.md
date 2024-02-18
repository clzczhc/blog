---
title: 数据库学习笔记(一)
date: 2021-10-20 14:26:17
categories: "SQL"
tags:
  - SQL
---

# 数据库学习笔记(一)

<b style="color: red">个人习惯，SQL 都直接用小写字母，SQL 不区分大小写，大写个人看着别扭。</b>

## 1. SQL 概述

- SQL(Structured Query Language)：结构化查询语言，是关系数据库的标准语言。

- SQL 是一个通用的、功能极强的关系数据库语言

- SQL 以同一种语法结构提供多种使用方式
  1. SQL 是一门独立的语言，能够独立地用于联机交互的使用方式
  2. SQL 也是一门嵌入式语言，能够嵌入到高级语言(如 C、Java)中

## 2. 数据定义

### 2.1 模式的定义与删除

#### 2.1.1 定义模式

```sql
create schema "S-T" authorization Wang   # 为用户Wang定义一个模式S-T
```

如果没有指定模式名，则模式名隐含为用户名

#### 2.1.2 删除模式

```sql
drop schema <模式名> <cascade|restrict>
```

- cascade(级联)：删除模式的同时把该模式中的所有数据库对象(如表等)全部删除
- restrict(限制)：如果该模式中定义了数据库对象(如表、视图等)，则会拒绝该删除语句的执行

### 2.2 基本表的定义、删除与修改

#### 2.2.1 基本表定义

语法：

```sql
create table <表名>
	(<列名> <数据类型> [<列级完整性约束条件>]
     [,<列名> <数据类型> [<列级完整性约束条件>]]
     ...
     [,<表级完整性约束条件>]);
```

基本表定义：

```sql
create table Student
	(Sno char(9) primary key,	# 列级完整性约束条件，Sno是主码
     Sname char(20) unique, 	# Sname取唯一值
     Sage smallint
    );
```

有外码的基本表定义

```sql
create table Course
	(Cno char(4) primary key,
     Cname char(40),
     foreign key(Cpno) references Course(Cno) # 表级完整性约束条件
     # Cpno是外码，被参照表是Course,被参照列是Cno
    );
```

#### 2.2.2 模式与表

- 每一个基本表都属于某一个模式

- 一个模式包含多个基本表

- 定义基本表所属模式

  1. 在表名中给出模式名

     ```sql
     create table "S-T".Student(...);   # 模式名为S-T
     ```

  2. 在创建模式语句中同时创建表

     ```sql
     create schema Test authorization Zhang
     create table tab(...);
     ```

  3. 设置所属的模式

- 创建基本表(以及其他数据库对象)，如果没有指定模式，系统会根据**搜索对象**来确定该对象所属的模式

- 关系数据库管理系统会使用模式列表中**第一个存在的模式**作为数据库对象的模式名

显示当前的搜索路径: ` show search_path;`

#### 2.2.3 修改基本表

- 增加列

  ```sql
  alter table Student add S_entrance Date;
  /* 向Student表中增加列S_entrance, 数据类型为日期型 */
  ```

  **新增的列为空值**

- 修改列的数据类型

  ```sql
  alter table Student alter column Sage int;
  /* 将年龄那一列地数据类型变为整型 */
  ```

- 增加列的约束条件

  ```sql
  alter table Course add unique(Cname)
  /* 为表Course的Cname列增加必须取唯一值的约束条件 */
  ```

#### 2.2.4 删除基本表

```sql
drop table <表名> [cascade|restrict]  # 参考删除模式
```

### 2.3 索引的建立与删除

建立索引的目的：加快查询速度

#### 2.3.1 建立索引

语法：

```sql
create [unique] [cluster] index <索引名> on <表名>(<列名>[<次序>]);
```

- 次序：指定索引值的排列次序，升序：asc，降序：desc，默认值是 asc
- unique：该索引的每一个索引值只对应唯一的数据记录
- cluster：表示要建立的索引是聚簇索引

```sql
create unique index Stusno on Student(Sno); # Student表按学号升序建唯一索引
create unique index SCno on SC(Sno, asc, Cno, desc);
/* SC表按学号升序和课程号降序建唯一索引 */
```

#### 2.3.2 修改索引

语法：

```sql
alter index <旧索引名> rename to <新索引名>
```

```sql
alter index SCno rename to SCSno; # 将SC表的SCno索引名改为SCSno
```

#### 2.3.3 删除索引

```sql
drop index SCno; # 删除SCno索引
```

## 3. 数据查询

### 3.1 单表查询

**查询只涉及一个表**

- 选择表中的若干列

  ```sql
  /* 1. 查询指定列 */
  selete Sno, Sname from Student;

  /* 2. 查询全部列 */
  select * from Student;

  /* 3. 查询经过计算的值 */
  select Sname, 2021-Sage from Student;
  ```

- 选择表中的若干元组

  ```sql
  /* 如果没有指定distinct关键词，则默认为all，all不会去重 */
  select Sno from SC;
  # 等价于
  select all Sno from SC;

  select distinct Sno from SC; # 会去重

  /* 查询满足条件的元组 */
  /* 1. 比较大小 */
  select distinct Sno from SC
  where Grade<60;  # 查询考试成绩有不及格的学生的学号

  /* 2. 确定范围 */
  select Sname from Student
  where Sage between 20 and 23; # 用谓词between ... and ... 来确定范围

  /* 3. 确定集合 */
  select Sname from Student
  where Sdept in ('CS', 'MA', 'Is'); # 查询CS系、MA系、IS系的学生的姓名

  /* 4. 字符匹配 */
  select * from Student
  where Sno like '201211123'; # 查询学号为201211123的学生的信息

  select Sname from Student
  where Sname like '_阳%'  # 查询名字中第二个字为"阳"的学生的姓名
  /* 通配符%代表任意长度的(包括长度为0)的字符串，而通配符_代表匹配任意单个字符 */

  select Cno from Course
  where Cname like 'DB/_Design' escape "/"; /*使用换码字符"/"将通配符转义为普通字符*/

  /* 5. 涉及空值的查询 */
  select Sno from SC
  where Grade is NULL;  # 查询没有成绩的学生的学号

  ```

- order by 子句

  ```sql
  select Sno from SC
  where Cno='3'
  order by Grade DESC; # 查询选修了3号课程的学生的学号，结果按分数降序排列
  ```

- 聚集函数

  ```sql
  select count(*) from Student;   # 查询学生总人数

  select avg(Grade) from SC
  where Cno='1';  # 查询1号课程的学生的平均成绩
  ```

- group by 子句

  ```sql
  select Cno, count(Sno) from SC
  group by Cno; # 根据Cno分组，会去重

  select Cno, count(Sno) from SC
  group by Cno, Sno; # 根据Cno,Sno分组，只有Cno和Sno相同的会分为一组

  /* 增加条件表达式 */
  select Sno from Sc
  group by Sno
  having avg(Grade)>=90;
  ```

  **having 短语与 where 自居的区别**：

  - 作用对象不同
  - where 子句作用于基表或视图，从中选择满足条件的元组
  - having 短语作用于组，从中选择满足条件的元组

  <b style="color: red">where 子句中不能使用聚集函数作为条件表达式</b>

### 3.2 连接查询

连接查询：同时涉及两个以上的表的查询

- 等值与非等值连接查询

  ```sql
  /* 等值连接 */
  select Student.*, SC.* from Studentm SC
  where Student.Sno = SC.Sno   # 查询每个学生及其选修课程的情况

  /*  自然连接 */
  select Student.Sno, Sname, Sage, Cno, GRade from Student, SC
  where Student.Sno = SC.Sno  # 上面的查询用自然连接完成
  ```

- 自身连接

  ```sql
  select First.Cno, Second.Cpno
  from Course First, Course Second # 因为所有属性名都是同名的，所以需要使用别名前缀
  where First.Cpno = Second.Cno;  # 查询每一门课的间接先修课
  ```

- 外连接

- 多表连接

  ```sql
  select Student.Sno, Sname, Cname
  from Student, SC, Course
  where Student.Sno = SC.Sno and SC.Cno = Course.Cno;
  ```

### 3.3 嵌套查询

```sql
/*  1. 带有in谓词的子查询 */
select Sno, Sname, Sdept from Student
where Sdept in
	(select Sdept from Student
     where Sname='刘成'
    );   # 查询和"刘成"在同一个系的学生


/* 2. 带有比较运算符的子查询 */
select Sno, Cno from SC x
where Grade >= (select avg(Grade) from SC y
			   where y.Sno = x.Sno);

/* 3. 带有any(some)或all谓词的子查询 */
# 使用any或all谓词时需要同时使用比较运算
# any表示子查询结果中的某个值, all表示子查询结果中的所有值

select Sname, Sage from Student
where Sage < any(select Sage from Student
                 where Sdept='CS')
and Sdept <> 'CS';   # 父查询块中的条件，'<>'作用和'!='一样
/* 子查询先把所有CS系的学生的年龄找出来，然后使用any谓词找出比找出来的结果中任意一个年龄小的 */

/* 4. 带有exists谓词的子查询 */
select Sname from Student
where exists   # 带有exists谓词的子查询不返回任何数据，若内层查询结果为空，则外层的where子句返回假值
	(select * from SC
     where Sno = Student.Sno and Cno = '1'
    );

```

### 3.4 集合查询

- 并集 union

  ```sql
  select * from Student
  where Sdept='CS'
  union
  select * from Student
  where Sage<=19  # 查询CS系以及年龄不超过19岁的学生
  /* union: 将多个查询结果合并起来，系统会自动去重 */
  /* union all: 将多个查询结果合并起来后，不去重 */
  ```

- 交集 intersect

  ```sql
  select * from Student
  where Sdept='CS'
  intersect
  select * from Student
  where Sage<=19   # 查询CS系而且年龄不超过19岁的学生
  ```

- 差集 except

  ```sql
  select * from Student
  where Sdept='CS'
  except
  select * from Student
  where Sage<=19   # 查询CS系中年龄超过19岁的学生
  ```

## 4. 数据更新

### 4.1 插入数据

两种插入方式：

- 插入元组

  语法：

  ```sql
  insert into <表名> [(<属性列1>, <属性列2>...)]
  values(<常量1>, <常量2>...);
  ```

  作用：将元组插入到指定表中

  into 子句：

  1. 指定要插入的表名及属性列
  2. 属性列的顺序和表定义的顺序可以不一致
  3. 没有指定属性列：表示要插入的是完整的元组，而且属性列顺序和表定义的顺序需要一致
  4. 指定部分属性列：插入的元组在其余属性列上取空值

  values 子句：提供的值必须和 into 子句匹配，包括值的个数和值的类型

  ```sql
  insert into SC(Sno, Cno)
  values('20121112', '1');   # 增加选课记录

  insert into Student
  values('2012322221', 'ttt', '男', 111, 'CS');  # into子句没有给出指定属性列，所以插入的是完整的元组
  ```

- 插入子查询结果

  ```sql
  /* 1. 建表 */
  create table Dept_age
  	(Sdept char(15)
       Avg_age smallint);

  /* 2. 插入数据 */
  insert into Dept_age(Sdept, Avg_age)
  	select Sdept, avg(Sage) from Student
  	group by Sdept;
  ```

### 4.2 修改数据

语句格式：

```sql
update <表名>
set <列名>=<表达式>[, <列名>=<表达式>]...
[where <条件>]  # 省略where子句，表示要修改表中的所有元组
```

#### 4.2.1 修改某一个元组的值

```sql
update Student
set Sage = 22
where Sno = '201123012';
```

#### 4.2.2 修改多个元组的值

```sql
update Student
set Sage = Sage + 1;
```

#### 4.2.3 带子查询的修改语句

```sql
update SC
set Grade = 0
where Sno in
	(select Sno from Student
     where Sdept = 'CS');
```

### 4.3 删除数据

#### 4.3.1 删除某一个元组的值

```sql
delete from Student
where Sno = '201233215';
```

#### 4.3.2 删除多个元组的值

```sql
delete from SC;
```

#### 4.3.3 带子查询的删除语句

```sql
delete from SC
where Sno in
	(select Sno from Student
     where Sdept='CS');
```

## 5. 空值的处理

**空值**：不知道或不存在或无意义的值，有以下几种情况

- 该属性应该有值，但是目前不知道它的具体值
- 该属性不应该有值
- 由于某种原因不便于填写

```sql
# 产生空值
insert into SC(Sno, Cno, Grade)
values('2012345632', '1', null);

insert into SC(Sno, Cno)
values('2012345621', '1');	# 没有赋值的属性，其值为空值

update Student
set Sdept = null
where Sno = '20123245656';

# 空值的判断
select * from Student
where Sname is null;
```

**不能取空值的情况**：

- 有 not null 约束条件不能取空值
- 加了 unique 限制的属性不能取空值
- 码属性不能取空值

**空值与另一个值(包括空值)的算术运算的结果是空值**

**空值与另一个值(包括空值)的比较运算结果是 unknown**

## 6. 视图

视图的特点：

- 虚表，是从一个或几个基本表导出的表
- 只存放视图的定义，不存放视图对应的数据
- 基表中的数据发生变化，从试图中查询出来的数据也会改变

### 6.1 定义视图

#### 6.1.1 建立视图

语法格式：

```sql
create view <视图名> [<列名>, <列名>...]
as <子查询>
[with check option];
# with check option：对视图进行update、insert和delete操作时要保证更新、插入和删除的行满足视图定义中的谓词条件
```

```sql
create view IS_Student
as
select Sno from Student
where Sdept = 'IS'
```

#### 6.1.2 删除视图

语法格式：

```sql
drop view <视图名> [cascade];	# 使用cascade级联删除语句，会把该视图和由它导出的所有的所有试图一起删除
```

**删除基表时，需要显式的使用 drop view 语句删除，或者使用 cascade 级联删除**

```sql
drop view IS_Student;
```

### 6.2 查询视图

```sql
select Sno, Sage from IS_Student
where Sage < 20;

# 视图消解转换后的查询语句：
select Sno, Sage from Student
where Sdept = 'IS' and Sage < 20;

/* 视图消解法：1. 进行有效性检查
			2. 转换成等价的对基本表的查询
			3. 执行修正后的查询
```

### 6.3 更新视图

```sql
update IS_Student
set Sname = 'clz'
where Sno = '20213114565';

# 转换后的语句：
update Student
set Sname = 'clz'
where Sno = '20213114565' and Sdept = 'IS';

insert into IS_Student
values('202132456', 'clz', 21);

# 转换后的语句：
insert into Student(Sno, Sname, Sage, Sdept)
values('202132456', 'clz', 21, 'IS');
```

### 6.4 视图的作用

- 视图能够简化用户的操作
- 视图使用户能以多种角度看待同一数据
- 视图对重构数据库提供了一定程度的逻辑独立性
- 视图能够对机密数据提供安全保护
- 适当的利用视图可以更清晰的表达查询
