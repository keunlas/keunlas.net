---
title: 初试MySQL
comments: true
tags:
  - mysql
categories: 随手记
---

# 展示全部数据库

```sql
show databases;
```

# 检查权限

```sql
show grants for 'user'@'%';
```

# 创建数据库

```sql
creat database test;
```

# 删除数据库

```sql
drop database test;
```

# 查询可用数据库

```sql
select database();
```

# 进入数据库

```sql
use test;
```

# 创建表

```sql
create table pet (
  name varchar(20),
  owner varchar(20),
  species varchar(20),
  sex char(1),
  birth date,
  death date
);
```

# 查看表的结构信息

```sql
describe pet;
+---------+-------------+------+-----+---------+-------+
| Field   | Type        | Null | Key | Default | Extra |
+---------+-------------+------+-----+---------+-------+
| name    | varchar(20) | YES  |     | <null>  |       |
| owner   | varchar(20) | YES  |     | <null>  |       |
| species | varchar(20) | YES  |     | <null>  |       |
| sex     | char(1)     | YES  |     | <null>  |       |
| birth   | date        | YES  |     | <null>  |       |
| death   | date        | YES  |     | <null>  |       |
+---------+-------------+------+-----+---------+-------+
```

# 查看这个表如何被创建出来

```sql
show create table pet;
+-------+--------------------------------------------------------------------+
| Table | Create Table                                                       |
+-------+--------------------------------------------------------------------+
| pet   | CREATE TABLE `pet` (                                               |
|       |   `name` varchar(20) DEFAULT NULL,                                 |
|       |   `owner` varchar(20) DEFAULT NULL,                                |
|       |   `species` varchar(20) DEFAULT NULL,                              |
|       |   `sex` char(1) DEFAULT NULL,                                      |
|       |   `birth` date DEFAULT NULL,                                       |
|       |   `death` date DEFAULT NULL                                        |
|       | ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci |
+-------+--------------------------------------------------------------------+
```

可以加上\G, 不以表格显示。

```sql
show create table pet\G;
```

# 插入操作

```sql
insert into pet
  values
  ("puffball", "Tome", 'hamster ', 'f', '1999-03-30', NULL);
```

```sql
insert into pet
  (name, `owner`, species, birth)
  values
  ('White', 'Lukas', 'dog', '1997-9-18');
```

# 查询表中所有数据

```
select * from pet;
```

# 查询表中部分行

```
select name,owner from pet;
```

# 配合筛选查询

```
select name,owner from pet where sex = 'f';
```

# 进行排序

默认升序，desc降序。

```
SELECT name, birth FROM pet ORDER BY birth;

SELECT name, birth FROM pet ORDER BY birth DESC;
```

# 多次排序

```
SELECT name, birth FROM pet ORDER BY species,birth DESC;
```

# 空值NULL相关的操作

空值NULL是一种特殊的值，一般情况下它描述了未确定的值的含义，因此对空值的处理有别于其他类型
的值。

使用is null和is not null运算符可以判断某个值是否为NULL。

空值无法进行比较。

# 简单的模式匹配

- `_` 匹配单个字符
- `%` 匹配任意字符串

```sql
SELECT * FROM pet WHERE name LIKE 'b%';
```

# 聚集函数和统计行数

- 对表整体进行统计

```sql
SELECT owner, COUNT(*) FROM pet GROUP BY owner;
```

- 分组统计

```sql
SELECT owner, COUNT(*) FROM pet GROUP BY owner;
SELECT species, COUNT(*) FROM pet GROUP BY species;
SELECT species,COUNT(species) FROM pet GROUP BY species HAving COUNT(species) >= 2;
```















