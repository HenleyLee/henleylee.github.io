---
title: SQL 之基础操作
categories: 学习笔记
tags:
  - 学习笔记
abbrlink: 38865d3b
date: 2019-03-28 18:56:25
---

**`SQL`** 是用于访问和处理数据库的标准的计算机语言。

## SQL 简介 ##
### SQL 是什么？ ###
 - SQL 是指结构化查询语言，全称是 Structured Query Language。
 - SQL 是一种 ANSI(American National Standards Institute 美国国家标准化组织)标准的计算机语言。

### SQL 能做什么？ ###
 - SQL 面向数据库执行查询
 - SQL 可从数据库取回数据
 - SQL 可在数据库中插入新的记录
 - SQL 可更新数据库中的数据
 - SQL 可从数据库删除记录
 - SQL 可创建新数据库
 - SQL 可在数据库中创建新表
 - SQL 可在数据库中创建存储过程
 - SQL 可在数据库中创建视图
 - SQL 可以设置表、存储过程和视图的权限

虽然 SQL 是一门 ANSI 标准的计算机语言，但是仍然存在着多种不同版本的 SQL 语言。然而，为了与 ANSI 标准相兼容，它们必须以相似的方式共同地来支持一些主要的命令(比如 `SELECT`、`UPDATE`、`DELETE`、`INSERT`、`WHERE` 等等)。

> 注释：除了 SQL 标准之外，大部分 SQL 数据库程序都拥有它们自己的专有扩展！

### 在网站上如何使用 SQL？ ###
要创建一个显示数据库中数据的网站，需要：
 - RDBMS 数据库程序(比如 MS Access、SQL Server、MySQL)
 - 使用服务器端脚本语言，比如 PHP 或 ASP
 - 使用 SQL 来获取您想要的数据
 - 使用 HTML / CSS

### RDBMS ###
 - RDBMS(Relational Database Management System) 指关系型数据库管理系统。
 - RDBMS 是 SQL 的基础，同样也是所有现代数据库系统的基础，比如 MS SQL Server、IBM DB2、Oracle、MySQL 以及 Microsoft Access。
 - RDBMS 中的数据存储在被称为表的数据库对象中。表是相关的数据项的集合，它由列和行组成。

## SQL 语法 ##
### 数据库表 ###
一个数据库通常包含一个或多个表。每个表有一个名字标识(例如 Customer、order)，表包含带有数据的记录(行)。

### SQL 语句 ###
在数据库上执行的大部分工作都由 SQL 语句完成。

 - SQL 语句的大小写问题：
**SQL 对大小写不敏感**：`SELECT` 与 `select` 是相同的。

 - SQL 语句后面的分号问题：
某些数据库系统要求在每条 SQL 语句的末端使用分号。分号是在数据库系统中分隔每条 SQL 语句的标准方法，这样就可以在对服务器的相同请求中执行一条以上的 SQL 语句。

### SQL 命令 ###
SQL 是指结构化查询语言，是用于执行查询的语法。但是 SQL 语言也包含用于更新、插入和删除记录的语法。

可以把 SQL 分为两个部分：`数据定义语言(DDL)`和`数据操作语言(DML)`。

#### 数据定义语言(DDL) ####
数据定义语言(Data Definition Language，简称 DDL)用于改变数据库结构，包括创建/更改/删除数据库对象或表、定义索引(键)、规定表之间的链接以及施加表间的约束。SQL 中最重要的 DDL 语句:
 - **CREATE DATABASE** - 创建新数据库
 - **ALTER DATABASE** - 修改数据库
 - **DROP DATABASE** - 删除数据库
 - **CREATE TABLE** - 创建新表
 - **ALTER TABLE** - 修改表
 - **TRUNCATE TABLE** - 删除表中数据
 - **DROP TABLE** - 删除表
 - **CREATE INDEX** - 创建索引(搜索键)
 - **DROP INDEX** - 删除索引

#### 数据操作语言(DML) ####
数据操作语言(Data Manipulation Language，简称 DML)用于检索、插入、修改和删除数据。SQL 中最重要的 DML 语句:
 - **SELECT** - 从数据库表中查询数据
 - **INSERT** - 向数据库表中插入数据
 - **UPDATE** - 更新数据库表中的数据
 - **DELETE** - 从数据库表中删除数据

## SQL CREATE DATABASE ##
`CREATE DATABASE` 用于创建数据库。SQL CREATE DATABASE 语法如下：
```sql
CREATE DATABASE database_name;
```
## SQL DROP DATABASE ##
`DROP DATABASE` 语句用于删除数据库。SQL DROP DATABASE 语法如下：
```sql
DROP DATABASE database_name;
```

## SQL CREATE TABLE ##
`CREATE TABLE` 语句用于创建数据库中的表。表由行和列组成，每个表都必须有个表名。SQL CREATE TABLE 语法如下：
```sql
CREATE TABLE table_name
(
column_name1 data_type(size),
column_name2 data_type(size),
column_name3 data_type(size),
....
);
```

 - column_name：规定表中列的名称。
 - data_type：规定列的数据类型(例如 varchar、integer、decimal、date 等等)。
 - size：规定表中列的最大长度。

## SQL ALTER TABLE ##
`ALTER TABLE` 语句用于在已有的表中添加、修改或删除列。

如需在表中添加列，请使用下列语法:
```sql
ALTER TABLE table_name
ADD column_name data_type
```

要删除表中的列，请使用下列语法(请注意，某些数据库系统不允许这种在数据库表中删除列的方式)：
```sql
ALTER TABLE table_name
DROP COLUMN column_name
```

要改变表中列的数据类型，请使用下列语法：
```sql
ALTER TABLE table_name
ALTER COLUMN column_name data_type
```

## SQL TRUNCATE TABLE ##
`TRUNCATE TABLE` 语句用于删除表中的所有行，但表结构及其列、约束、索引等保持不变。SQL TRUNCATE TABLE 语法如下：
```sql
TRUNCATE TABLE table_name;
```

## SQL DROP TABLE ##
`DROP TABLE` 语句用于删除表。SQL DROP TABLE 语法如下：
```sql
DROP TABLE table_name;
```

## SQL CREATE INDEX ##
`CREATE INDEX` 语句用于在表中创建索引。在不读取整个表的情况下，索引使数据库应用程序可以更快地查找数据。。SQL CREATE INDEX 语法如下：
 - 在表上创建一个简单的索引，允许使用重复的值：
```sql
CREATE INDEX index_name
ON table_name (column_name);
```

 - 在表上创建一个唯一的索引，不允许使用重复的值(唯一的索引意味着两个行不能拥有相同的索引值)：
```sql
CREATE UNIQUE INDEX index_name
ON table_name (column_name);
```

用户无法看到索引，它们只能被用来加速搜索/查询。如果希望索引不止一个列，可以在括号中列出这些列的名称，用逗号隔开。

虽然索引的目的在于提高数据库的性能，但这里有几个情况需要避免使用索引。使用索引时，应重新考虑下列准则：
 - 索引不应该使用在较小的表上。
 - 索引不应该使用在有频繁的大批量的更新或插入操作的表上。
 - 索引不应该使用在含有大量的 NULL 值的列上。
 - 索引不应该使用在频繁操作的列上。

> 注意：更新一个包含索引的表需要比更新一个没有索引的表更多的时间，这是由于索引本身也需要更新。因此，理想的做法是仅仅在常常被搜索的列(以及表)上面创建索引。

## SQL DROP INDEX ##
`DROP INDEX` 语句用于删除表中的索引。

 - 用于 MS Access 的语法如下：
```sql
DROP INDEX index_name ON table_name;
```

 - 用于 MS SQL Server 的语法如下：
```sql
DROP INDEX table_name.index_name;
```

 - 用于 DB2/Oracle 的语法如下：
```sql
DROP INDEX index_name;
```

 - 用于 MySQL 的语法如下：
```sql
ALTER TABLE table_name DROP INDEX index_name;
```

> 当删除索引时应特别注意，因为性能可能会下降或提高。

## SQL SELECT ##
`SELECT` 语句用于从数据库中查询数据，结果被存储在一个结果表中，称为结果集。SQL SELECT 语法如下：
```sql
SELECT column_name,column_name
FROM table_name;
```
与
```sql
SELECT * FROM table_name;
```

在表中，一个列可能会包含多个重复值，有时也许希望仅仅列出不同(distinct)的值。`DISTINCT` 关键词用于返回唯一不同的值。SQL SELECT DISTINCT 语法如下：
```sql
SELECT DISTINCT column_name,column_name
FROM table_name;
```

## SQL WHERE ##
`WHERE` 子句用于提取那些满足指定标准的记录。SQL WHERE 语法如下：
```sql
SELECT column_name,column_name
FROM table_name
WHERE column_name operator value;
```

下面的运算符可以在 `WHERE` 子句中使用：

| 运算符    | 描述                                                   |
| :-------: | ------------------------------------------------------ |
| `=`       | 等于                                                   |
| `<>`      | 不等于(注释：在 SQL 的一些版本中，该操作符可被写成 !=) |
| `>`       | 大于                                                   |
| `<`       | 小于                                                   |
| `>=`      | 大于等于                                               |
| `<=`      | 小于等于                                               |
| `BETWEEN` | 在某个范围内                                           |
| `LIKE`    | 搜索某种模式                                           |
| `IN`      | 指定针对某个列的多个可能值                             |

## SQL AND & OR ##
`AND` 和 `OR` 运算符用于基于一个以上的条件对记录进行过滤。
SQL AND 语法如下：
```sql
SELECT * FROM table_name
WHERE column_name operator value
AND column_name operator value;
```
> 如果第一个条件和第二个条件都成立，则 `AND` 运算符显示该条记录。

SQL OR 语法如下：
```sql
SELECT * FROM table_name
WHERE column_name operator value
OR column_name operator value;
```
> 如果第一个条件和第二个条件中只要有一个成立，则 `OR` 运算符显示该条记录。

SQL AND 和 OR 结合语法如下：
```sql
SELECT * FROM table_name
WHERE column_name operator value
AND (column_name operator value OR column_name operator value);
```

## SQL ORDER BY ##
`ORDER BY` 关键字用于对结果集按照一个列或者多个列进行排序。ORDER BY 关键字默认按照升序对记录进行排序。如果需要按照降序对记录进行排序，可以使用 `DESC` 关键字。SQL ORDER BY 语法如下：
```sql
SELECT column_name,column_name
FROM table_name
ORDER BY column_name [ASC|DESC],column_name [ASC|DESC];
```

 - `ORDER BY` 多列的时候，先按照第一个列名排序，在按照第二个列名排序；
 - `ORDER BY` 排列时，不写明 `ASC|DESC` 的时候，默认是 `ASC`，`DESC` 需要写明；
 - `ASC` 或 `DESC`只对它紧跟着的第一个列名有效，其他列名不受影响，仍然是默认的升序。

> `ASC` 是指定列按`升序`排列，`DESC` 则是指定列按`降序`排列。

## SQL INSERT ##
`INSERT INTO` 语句用于向表中插入新记录。INSERT INTO 语句可以有两种编写形式：
 - 第一种形式无需指定要插入数据的列名，只需提供被插入的值即可：
```sql
INSERT INTO table_name
VALUES (value1,value2,value3,...);
```
> 该种形式需要列出插入行的每一列数据。

 - 第二种形式需要指定列名及被插入的值：
```sql
INSERT INTO table_name (column1,column2,column3,...)
VALUES (value1,value2,value3,...);
```
> 该种形式没有指定的列将被自动填充为 null。

## SQL UPDATE ##
`UPDATE` 语句用于更新表中已存在的记录。SQL UPDATE 语法如下：
```sql
UPDATE table_name
SET column1=value1,column2=value2,...
WHERE some_column=some_value;
```

> **`请注意 SQL UPDATE 语句中的 WHERE 子句！`** WHERE 子句规定哪条记录或者哪些记录需要更新。如果省略了 WHERE 子句，所有的记录都将被更新！执行没有 WHERE 子句的 UPDATE 要慎重，再慎重。

## SQL DELETE ##
`DELETE` 语句用于删除表中的行。SQL DELETE 语法如下：
```sql
DELETE FROM table_name
WHERE some_column=some_value;
```

> **`请注意 SQL DELETE 语句中的 WHERE 子句！`** WHERE 子句规定哪条记录或者哪些记录需要删除。如果您省略了 WHERE 子句，所有的记录都将被删除！

可以在不删除表的情况下，删除表中所有的行。这意味着表结构、属性、索引将保持不变：
```sql
DELETE FROM table_name;
```
或
```sql
DELETE * FROM table_name;
```
> **注意：**在删除记录时要格外小心！因为不能重来！


## 参考 ##
[W3Cschool SQL 教程](http://www.w3school.com.cn/sql/index.asp)

