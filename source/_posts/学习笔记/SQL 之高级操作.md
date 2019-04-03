---
title: SQL 之高级操作
categories: 学习笔记
tags:
  - 学习笔记
abbrlink: c55cd1a6
date: 2019-03-31 10:36:55
---

**`SQL`** 是用于访问和处理数据库的标准的计算机语言。

## SQL 约束 ##
SQL 约束用于规定表中的数据规则。如果存在违反约束的数据行为，行为会被约束终止。

约束可以在创建表时规定(通过 CREATE TABLE 语句)，或者在表创建之后规定(通过 ALTER TABLE 语句)。

SQL CREATE TABLE + CONSTRAINT 语法如下：
```sql
CREATE TABLE table_name
(
column_name1 data_type(size) constraint_name,
column_name2 data_type(size) constraint_name,
column_name3 data_type(size) constraint_name,
....
);
```

在 SQL 中，我们有如下约束：
 - `NOT NULL` - 强制某列不能接受 NULL 值(默认情况下，表的列接受 NULL 值)。NOT NULL 约束强制字段始终包含值。这意味着，如果不向字段添加值，就无法插入新记录或者更新记录。
 - `UNIQUE` - 唯一标识数据库表中的每条记录，保证某列的每行必须有唯一的值。
 - `PRIMARY KEY` - 唯一标识数据库表中的每条记录，确保某列或多列组合有唯一标识，相当于 NOT NULL 和 UNIQUE 的结合。每个表都应该有一个主键，并且每个表只能有一个主键。
 - `FOREIGN KEY` - 一个表中的 FOREIGN KEY 指向另一个表中的 UNIQUE KEY(唯一约束的键)，保证一个表中的数据匹配另一个表中的值的参照完整性。
 - `CHECK` - 用于限制列中的值的范围，保证列中的值符合指定的条件。如果对单个列定义 CHECK 约束，那么该列只允许特定的值。如果对一个表定义 CHECK 约束，那么此约束会基于行中其他列的值在特定的列中对值进行限制。
 - `DEFAULT` - 用于向列中插入默认值，规定没有给列赋值时的默认值。如果没有规定其他的值，那么会将默认值添加到所有的新记录。

> 请注意，每个表可以有多个 UNIQUE 约束，但是每个表只能有一个 PRIMARY KEY 约束。

## SQL AUTO INCREMENT ##
`AUTO INCREMENT` 关键字会在新记录插入表中时生成一个唯一的数字，用于表中的字段值自动递增。

默认地，AUTO_INCREMENT 的开始值是 1，每条新记录递增 1。

## SQL LIMIT ##
`LIMIT` 子句用于限制由 SELECT 语句返回的数据数量。对于拥有数千条记录的大型表来说，LIMIT 子句是非常有用的。

 - 用于 SQL Server / MS Access 语法如下：
```sql
SELECT TOP number|percent column_name(s)
FROM table_name;
```

 - 用于 MySQL 语法如下：
```sql
SELECT column_name(s)
FROM table_name
LIMIT number;
```

 - 用于 Oracle 语法如下：
```sql
SELECT column_name(s)
FROM table_name
WHERE ROWNUM <= number;
```

> 注意：并非所有的数据库系统都支持 SELECT LIMIT 语句。

## SQL LIKE ##
`LIKE` 操作符用于在 WHERE 子句中搜索列中的指定模式。SQL LIKE 操作符语法如下：
```sql
SELECT column_name(s)
FROM table_name
WHERE column_name LIKE pattern;
```

## SQL 通配符 ##
在搜索数据库中的数据时，SQL 通配符可以替代一个或多个字符。SQL 通配符必须与 `LIKE` 运算符一起使用。

在 SQL 中，可使用以下通配符：

| 通配符   	               | 描述                                                   |
| :--------------------------: | ------------------------------------------------------ |
| `%` 	  	               | 替代一个或多个字符                                     |
| `_` 	  	               | 仅替代一个字符                                         |
| `[charlist]` 	               | 字符列中的任何单一字符                                 |
| `[^charlist]`或`[!charlist]` | 不在字符列中的任何单一字                               |

搭配以上通配符可以让 LIKE 命令实现多种技巧：
 - `LIKE 'Mc%'`：将搜索以字母 Mc 开头的所有字符串(如 McBadden)。
 - `LIKE '%inger'`：将搜索以字母 inger 结尾的所有字符串(如 Ringer、Stringer)。
 - `LIKE '%en%'`：将搜索在任何位置包含字母 en 的所有字符串(如 Bennet、Green、McBadden)。
 - `LIKE '_heryl'`：将搜索以字母 heryl 结尾的所有六个字母的名称(如 Cheryl、Sheryl)。
 - `LIKE '[CK]ars[eo]n'`：将搜索下列字符串：Carsen、Karsen、Carson 和 Karson(如 Carson)。
 - `LIKE '[M-Z]inger'`：将搜索以字符串 inger 结尾、以从 M 到 Z 的任何单个字母开头的所有名称(如 Ringer)。
 - `LIKE 'M[^c]%'`：将搜索以字母 M 开头，并且第二个字母不是 c 的所有名称(如 MacFeather)。

## SQL IN ##
`IN` 操作符允许在 WHERE 子句中规定多个值。SQL IN 语法如下：
```sql
SELECT column_name(s)
FROM table_name
WHERE column_name IN (value1,value2,...);
```

如果需要选取不在规定的多个值内的记录，可以使用 `NOT IN`：
```sql
SELECT column_name(s)
FROM table_name
WHERE column_name NOT IN (value1,value2,...);
```

`IN` 与 `=` 的异同：
 - 相同点：均在 `WHERE` 中作为筛选条件之一使用，均是等于的含义。
 - 不同点：`IN` 可以规定多个值，`=` 只能规定一个值。

下面两种语法等价：
 - **IN**
```sql
SELECT column_name(s)
FROM table_name
WHERE column_name IN (value1,value2,...);
```

 - **=**
```sql
SELECT column_name(s)
FROM table_name
WHERE column_name=value1 or column_name=value2;
```

## SQL BETWEEN ##
`BETWEEN` 操作符选取介于两个值之间的数据范围内的记录。这些值可以是数值、文本或者日期。SQL BETWEEN 语法如下：
```sql
SELECT column_name(s)
FROM table_name
WHERE column_name BETWEEN value1 AND value2;
```

如果需要选取不介于两个值之间的数据范围内的记录，可以使用 `NOT BETWEEN`：
```sql
SELECT column_name(s)
FROM table_name
WHERE column_name NOT BETWEEN value1 AND value2;
```

请注意，在不同的数据库中，BETWEEN 操作符会产生不同的结果！
 - 在某些数据库中，BETWEEN 选取介于两个值之间但不包括两个测试值的字段。
 - 在某些数据库中，BETWEEN 选取介于两个值之间且包括两个测试值的字段。
 - 在某些数据库中，BETWEEN 选取介于两个值之间且包括第一个测试值但不包括最后一个测试值的字段。

因此，请检查您的数据库是如何处理 BETWEEN 操作符！

## SQL 别名 ##
通过使用 SQL，可以为表名称或列名称指定别名。基本上，创建别名是为了让表名称或列名称的可读性更强。

 - 表的别名语法如下：
```sql
SELECT column_name(s)
FROM table_name AS alias_name;
```

 - 列的别名语法如下：
```sql
SELECT column_name AS alias_name
FROM table_name;
```

在下面的情况下，使用别名很有用：
 - 在查询中涉及超过一个表
 - 在查询中使用了函数
 - 列名称很长或者可读性差
 - 需要把两个列或者多个列结合在一起

## SQL JOIN ##
`JOIN` 子句用于把来自两个或多个表的行结合起来，基于这些表之间的共同字段。

可以使用的不同的 SQL JOIN 类型：
 - `INNER JOIN`：如果表中存在至少一个匹配，则返回行。
 - `LEFT JOIN`：即使右表中没有匹配，也从左表返回所有的行。如果右表中没有匹配，则结果为 NULL。
 - `RIGHT JOIN`：即使左表中没有匹配，也从右表返回所有的行。如果左表中没有匹配，则结果为 NULL。
 - `FULL JOIN`：只要左表和右表其中一个表中存在匹配，则返回行。结合了 LEFT JOIN 和 RIGHT JOIN 的结果。

## SQL UNION ##
`UNION` 操作符用于合并两个或多个 SELECT 语句的结果集。UNION 内部的每个 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每个 SELECT 语句中的列的顺序必须相同。

SQL UNION 语法如下：
```sql
SELECT column_name(s) FROM table1
UNION
SELECT column_name(s) FROM table2;
```

默认地，UNION 操作符选取不同的值。如果允许重复的值，请使用 `UNION ALL`。SQL UNION ALL 语法如下：
```sql
SELECT column_name(s) FROM table1
UNION ALL
SELECT column_name(s) FROM table2;
```

> 注意：UNION 结果集中的列名总是等于 UNION 中第一个 SELECT 语句中的列名。

## SQL SELECT INTO ##
`SELECT INTO` 语句从一个表复制数据，然后把数据插入到另一个新表中(要求目标表不存在)。SQL SELECT INTO 语法如下：
 - 可以复制所有的列插入到新表中：
```sql
SELECT *
INTO newtable [IN externaldb]
FROM table1;
```

 - 只复制希望的列插入到新表中：
```sql
SELECT column_name(s)
INTO newtable [IN externaldb]
FROM table1;
```

> 注意：新表将会使用 `SELECT` 语句中定义的列名称和类型进行创建。您可以使用 `AS` 子句来应用新名称。

## SQL INSERT INTO SELECT ##
`INSERT INTO SELECT` 语句从一个表复制数据，然后把数据插入到一个已存在的表中(要求目标表存在)。目标表中任何已存在的行都不会受影响。SQL INSERT INTO SELECT 语法如下：
 - 可以从一个表中复制所有的列插入到另一个已存在的表中：
```sql
INSERT INTO table2
SELECT * FROM table1;
```

 - 可以只复制希望的列插入到另一个已存在的表中：
```sql
INSERT INTO table2
(column_name(s))
SELECT column_name(s)
FROM table1;
```

## SQL GROUP BY ##
`GROUP BY` 语句用于结合聚合函数，根据一个或多个列对结果集进行分组。SQL GROUP BY 语法如下：
```sql
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name;
```

## SQL 函数 ##
SQL 拥有很多可用于计数和计算的内建函数。

### SQL Aggregate 函数 ###
SQL Aggregate 函数计算从列中取得的值，返回一个单一的值。有用的 Aggregate 函数：
 - `AVG()` - 返回数值列的平均值。
 - `COUNT()` - 返回匹配指定条件的行数。
 - `FIRST()` - 返回指定的列中第一个记录的值。
 - `LAST()` - 返回指定的列中最后一个记录的值。
 - `MAX()` - 返回指定列的最大值。
 - `MIN()` - 返回指定列的最小值。
 - `SUM()` - 返回数值列的总数。

### SQL Scalar 函数 ###
SQL Scalar 函数基于输入值，返回一个单一的值。有用的 Scalar 函数：
 - `UCASE()` - 把某个字段的值转换为大写。
 - `LCASE()` - 把某个字段的值转换为小写。
 - `MID()` - 从某个文本字段中提取字符。
 - `LEN()` - 返回某个文本字段中值的长度。
 - `ROUND()` - 对某个数值字段进行指定小数位数的四舍五入。
 - `NOW()` - 返回当前的系统的日期和时间。
 - `FORMAT()` - 用于对字段的显示方式进行格式化。

## 参考 ##
[W3Cschool SQL 教程](http://www.w3school.com.cn/sql/index.asp)

