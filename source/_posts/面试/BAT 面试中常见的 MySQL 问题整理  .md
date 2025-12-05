---
title: BAT 面试中常见的 MySQL 问题整理
date: 2025-10-30 14:42:34
category: 程序人生
tags: 面试
---

若某表具有自增主键 ID，在插入 17 条记录后，删除第 15、16、17 条记录，重启 MySQL 后再插入一条记录，这条记录的 ID 会是 15 还是
18？

MySQL 的技术特点有哪些？

请解释什么是 Heap 表。

MySQL 服务器的默认端口号是多少？

与 Oracle 相比，MySQL 有哪些优势？

如何区分 FLOAT 和 DOUBLE 类型？

CHAR_LENGTH 和 LENGTH 函数有什么区别？

简述 InnoDB 存储引擎支持的四类事务隔离级别及其区别。

ENUM 类型在 MySQL 中如何使用？

如何定义并使用 REGEXP 进行正则匹配？

CHAR 与 VARCHAR 的区别是什么？

列的字符串类型可以有哪些？

如何查看当前 MySQL 的版本信息？

MySQL 常用的存储引擎有哪些？

什么是 MySQL 驱动程序？

TIMESTAMP 类型字段设置为 ON UPDATE CURRENT_TIMESTAMP 时会有什么效果？

主键和候选键的区别是什么？

如何使用 Unix shell 登录 MySQL？

myisamchk 工具的作用是什么？

有哪些命令可用于分析 MySQL 数据库服务器的性能？

如何控制 HEAP 表的最大容量？

MyISAM Static 和 MyISAM Dynamic 有何不同？

什么是 federated 表？

如果表中有一个字段定义为 TIMESTAMP，会有什么特性？

当自增列达到最大值后继续插入，会发生什么？

如何获取最后一条插入记录的自增 ID 值？

如何查看某个表上定义的所有索引？

LIKE 语句中的 % 和 _ 分别代表什么？

如何在 Unix 时间戳和 MySQL 时间格式之间进行转换？

什么是列对比运算符？

如何获取受查询影响的行数？

MySQL 查询是否区分大小写？

LIKE 和 REGEXP 在匹配方式上有何不同？

BLOB 与 TEXT 类型有哪些区别？

mysql_fetch_array 和 mysql_fetch_object 的区别是什么？

如何在批处理模式下运行 MySQL 脚本？

MyISAM 表的存储位置和存储格式是怎样的？

MySQL 支持哪些表类型（存储引擎）？

ISAM 是什么？

InnoDB 是什么？

MySQL 如何对 DISTINCT 查询进行优化？

如何输入十六进制形式的字符？

如何显示查询结果的前 50 行？

一个索引最多可以包含多少列？

NOW() 和 CURRENT_DATE() 函数有什么区别？

可以使用 CREATE 语句创建哪些数据库对象？

一个 MySQL 表中最多可以创建多少个触发器？

什么是非标准字符串类型？

常见的 SQL 函数有哪些？

请解释 MySQL 中的访问控制列表（ACL）。

MySQL 是否支持事务？

在 MySQL 中存储金额数据，推荐使用什么字段类型？

什么情况下 MySQL 数据表容易损坏？

MySQL 中与权限管理相关的系统表有哪些？

MySQL 中常见的锁类型有哪些？