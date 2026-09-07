---
title: MySql基础
source: https://www.yuque.com/mook-mvqcp/viz8gx/dqz38pwycx29fc4v
created: 2026-05-15
updated: 2026-05-15
tags:
  - 数据库
---

# SQL语法

## 一、数据查询语言DQL

核心是SELECT查询语句，功能是从表中查找数据并返回。

### （一）基础语法

SELECT  columnname  FROM tablename [LIMIT n1[,n2]] ;  指定要查询的列和表。

可以查询一个列，也可以查询多个列，但是不同列之间用逗号分隔，并且最后一个列的后面不能带有逗号。如果要查询表的所有列，可以使用 ***** 指代所有的列。

LIMIT关键字可以指定返回查询结果的多少行数据或者从什么位置开始取多少行数据并返回，如果只写n1表示从第1行开始返回n1行数据，如果同时写了n1和n2，则表示从n1行开始返回n2行数据。OFFSET关键字可以用于指定返回多少行数据。

### （二）排序

ODER BY columnname1 [DESC or ASC] [,columnname2 [DESC or ASC]]...

ODER BY子句可以声明查询数据根据什么列来排序以及排序方向。

DESC表示降序，ASC表示升序。

先用columnname1的值排序，如果多个结果的该列的值相同，再用columnname2的值排序，以此类推。

### （三）过滤

WHERE子句实现对查询数据的过滤。

#### 1.单条件过滤

WHEREfilterconditon

过滤出列满足filtercondition的哪些行（也就是要返回的）。

filtercondition是条件表达式，可以使用关系运算符、between ... and 、is等等这些运算符

检查空值的过滤条件是IS NULL。

#### 2.多条件过滤

AND或者OR可以连接多个filtercondition。AND的优先级比OR高，如果要改变优先级可以使用()将条件包裹。

NOT关键字表示取反，表示排除掉符合NOT后面的条件的行。

IN表示在一个集合内取值，作用类似于BETWEEN ... AND ...

## 二、数据操纵语言DML

## 三、数据定义语言DDL

## 四、数据控制语言DCL

## 五、事务控制语言TCL
