---
title: SQL基础
date: 2016-10-12 11:31:38
tags:
---
#### SQL语句种类
1. DDL（Data Definition Language，数据定义语言）
包含指令有：CREATE、DROP、ALTER
2. DML（Data Manipulation Language，数据操作语言）
包含指令有：SELECT、INSERT、UPDATE、DELETE
3. DCL（Data Control Language，数据控制语言）
包含指令有：COMMIT、ROLLBACK、GRANT、REVOKE


#### 数据库的创建
```sql
CREATE DATABASE <数据库名称>
```

#### 表的创建
```sql
CREATE TABLE <表名> (
<列名1> <数据类型> <该列所需约束>,
<列名2> <数据类型> <该列所需约束>,
...
...
<该表约束1>,<该表约束2>,...);
```

#### 表的删除
```sql
DROP TABLE <表名>
```

#### 表定义的更新
1. 添加列
```sql
ALTER TABLE <表名> ADD COLUMN <列的定义>
```
2. 删除列
```sql
ALTER TABLE <表名> DROP COLUMN <列名>
```


#### SELECT

#### INSERT
1.从其它表复制数据
```
INSERT INTO <表名> (<列名1>,<列名2>,...) SELECT语句
```

#### DELETE
1.删除数据
```
DELETE FROM <表名>
```

#### UPDATE

#### 事务
```
开始语句(MYSQL:START TRANSACTION);
DML语句;
    .
    .
    .
结束语句(COMMIT或者ROLLBACK);
```

#### ACID
原子性、一致性、隔离性、持久性