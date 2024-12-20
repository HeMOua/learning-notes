# MySQL数据库

## 什么是 MySQL

+ 一种关系型数据库
+ 开源
+ 使用结构化查询语言 SQL 进行管理

## 常用的存储引擎

+ MyISAM 5.1 以前的默认引擎【my-z[ei]m】
  + 不支持外键以及事务
  + 默认锁级别为表级
  + 无事务日志
  + 数据和索引分开存储
  + 非聚簇索引
  + 查询性能相比 innodb 较好
+ innodb
  + 支持外检以及事务
  + 可以通过事务日志恢复数据
  + 支持行级锁以及表级锁
  + 数据与索引集中存储
  + 使用聚簇索引
  + 性能稍低

## 三大范式

第一范式：每列字段是原子的，不可再分割

第二范式：每列都和主键相关

第三范式：在第二范式的基础上，要求消除传递依赖。每列都和主键直接相关，而不是间接相关

## 数据类型

+ 整型
  + tinyint（1字节）、smallint（2字节）、mediumint（3字节）、int（4字节）、bigint（64）
+ 浮点型
  + float、double、decimal
+ 字符串
  + char、varchar
  + char 相比 varchar 效率更高，不宜产生碎片
  + 避免使用 text/blob 等类型，因为查询时会用到临时表，造成严重的性能开销
+ 日期
  + year、time、date、datetime、timestamp
  + datetime 使用 8 字节存储，保存 1000 年到 9999 年的时间，精度到秒，与时区无关
  + timestamp 使用 4 字节存储，保存 1970 年到 2038年，与时区相关

## 索引

### 简介

对数据库表一列或多列的值进行排序的一种结构，使用索引可以快速访问数据的特定信息

### 优缺点

+ 优点

  + 大大加快检索的速度

  + 将随机 I/O 变为顺序 I/O

  + 加快表与表的连接

+ 缺点
  + 建立索引需要额外的物理空间
  + 建立和维护索引都需要花费时间

### 索引的数据结构

有 B+树索引和哈希索引

+ *B+树索引*
  + B+树索引可以分为主索引和辅助索引。
  + 主索引为聚簇索引，叶子节点存储着完整的数据信息
  + 辅助索引为非聚簇索引，叶子节点储存着主键值

+ 哈希索引
  + 哈希索引是基于哈希表实现的，对于每一行数据，存储引擎会对索引列计算哈希值，并将哈希值作为哈希表的 key

### hash索引和B+树的区别

+ 哈希索引不支持排序，哈希表是无序的
+ 哈希索引不支持范围查询
+ 哈希索引不支持模糊查询
+ 哈希索引性能不稳定，因为可能存在 hash 值冲突的问题

### 索引类型

+ fulltext：全文索引，用来查找文本中的关键字，不是直接比较是否相等，常用来模糊查询
+ hash：哈希索引，多用于等值查询，但不支持排序、模糊查询、范围查询
+ btree：b+树索引，innodb存储引擎的默认索引，支持排序、分组、范围查询、模糊查询
+ rtree：空间数据索引，多用于地理数据的存储，在范围查询方面存在优势
