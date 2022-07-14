---
layout: post
title: "Codd's Rule"
tags: ["mysql","RDBMS","数据库"]
---

由计算机科学家Edgar Frank Codd提出的13条规则（也被称为科德12定律，从0到12共13条），满足这13条规则的数据库管理系统才能称之为
关系型数据库管理系统。

### 前言
科德定律中一部分表达比较晦涩，不太好翻译，但是大部分都只是常识上的数据库的基本功能以及数据库实现中的一些要求，真正需要我们记住的只有
一条：数据库应该支持用访问普通数据的语言（通常是SQL）来访问数据库的元数据（数据库名、表名、列名等）。

### 规则0
>The system must qualify as relational, as a database, and as a management system. For a system to qualify as a 
relational database management system (RDBMS), that system must use its relational facilities (exclusively) to
manage the database.

首先需要满足关系型（relational）、数据库（database）、管理系统（management system）这三个基本要点，并且能够使用它的关系能力来
管理数据库，这样的系统才能称之为关系型数据库管理系统。

### 规则1 信息法则
>Rule 1 : The information rule: All information in the database is to be represented in one and only one way, 
> namely by values in column positions within rows of tables.

所有数据库中的信息都只应该通过表行列中的值表示。
### 规则2 保证访问准则
>Rule 2 : The guaranteed access rule: All data must be accessible. This rule is essentially a restatement of the 
fundamental requirement for primary keys. It says that every individual scalar value in the database must be logically 
addressable by specifying the name of the containing table, the name of the containing column and the primary key 
value of the containing row.

所有表中的数据都可以通过表面、主键以及所在的列名访问。
### 规则3 null值处理
>Rule 3 : Systematic treatment of null values: The DBMS must allow each field to remain null (or empty). Specifically, 
it must support a representation of "missing information and inapplicable information" that is systematic, distinct 
from all regular values (for example, "distinct from zero or any other number", in the case of numeric values), and 
independent of data type. It is also implied that such representations must be manipulated by the DBMS in a systematic way.

应该支持null值，null值不同于其他所有类型的值（例如在表示数字时，它不同于0和其他所有数字），它表示信息缺失
### 规则4 基于关系模型的动态目录
>Rule 4 : Active online catalog based on the relational model: The system must support an online, inline, 
relational catalog that is accessible to authorized users by means of their regular query language. That is,
users must be able to access the database's structure (catalog) using the same query language that they use to access
the database's data.

用户可以使用访问普通数据库数据的查询语言来访问数据库结构（目录）
### 规则5
>Rule 5 : The comprehensive data sub language rule: The system must support at least one relational 
language that:
>>1. Has a linear syntax
>>2. Can be used both interactively and within application programs,
>>3. Supports data definition operations (including view definitions), data manipulation operations (update as well 
   as retrieval), security and integrity constraints, and transaction management operations (begin, commit, and rollback).

至少支持一种满足以下条件的语言：1、具有线性语法。2、可以用于交互也可以用于编程使用。3、支持ddl、dml、安全性和完整性约束、事务
### 规则6
>Rule 6 : The view updating rule: All views those can be updated theoretically, must be updated by the system.

所有理论上可以更新的视图，都应该由系统更新
### 规则7
>Rule 7 : High-level insert, update, and delete: The system must support set-at-a-time insert, update, and delete 
operators. This means that data can be retrieved from a relational database in sets constructed of data from multiple 
rows and/or multiple tables. This rule states that insert, update, and delete operations should be supported for 
any retrievable set rather than just for a single row in a single table.


### 规则8
>Rule 8 : Physical data independence: Changes to the physical level (how the data is stored, whether in arrays or 
linked lists etc.) must not require a change to an application based on the structure.

独立于物理数据，更改物理数据的存储结构不需要修改应用程序
### 规则9
>Rule 9 : Logical data independence: Changes to the logical level (tables, columns, rows, and so on) must not
require a change to an application based on the structure. Logical data independence is more difficult to achieve than 
physical data independence.

独立于逻辑数据，修改逻辑层不需要修改应用程序，这个比规则8要困难
### 规则10
>Rule 10 : Integrity independence: Integrity constraints must be specified separately from application programs and
stored in the catalog. It must be possible to change such constraints as and when appropriate without unnecessarily
affecting existing applications.

约束应该独立于应用程序并且存储在目录中
### 规则11
>Rule 11 : Distribution independence: The distribution of portions of the database to various locations should be
invisible to users of the database. Existing applications should continue to operate successfully :
>>1. when a distributed version of the DBMS is first introduced; and
>>2. when existing distributed data are redistributed around the system.

分布式中数据库部分的移动不应该被用户感知，即不管是引入分布式，或者分布式数据库重新分配，都不应该影响用户原本应该成功的操作
### 规则12
>Rule 12: The non-subversion rule: If the system provides a low-level (record-at-a-time) interface, then that 
interface cannot be used to subvert the system, for example, bypassing a relational security or integrity constraint.

系统如果提供了比较底层的接口，这个接口不应该破坏数据关系和约束
