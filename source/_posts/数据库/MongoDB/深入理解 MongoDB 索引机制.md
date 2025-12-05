---
title: 深入理解 MongoDB 索引机制
date: 2025-10-30 14:42:34
category: 数据库
tags: MongoDB
---

MongoDB 中的索引概念
与关系型数据库类似，MongoDB 中的索引也是用于加速数据查询的数据结构。索引建立在集合级别，可以在任意字段甚至嵌套子字段上创建。

如果没有索引，查询将不得不执行全集合扫描（Collection Scan），效率低下；而合理使用索引则可以大幅缩小需要检查的数据范围，从而显著提升查询性能。

索引本质上是一种特殊的数据结构（通常为
B-Tree），它存储了数据集的一小部分，并按照特定顺序组织，使得数据易于遍历。索引存储指定字段的值并按值排序，既能支持高效的等值匹配查询，也能快速处理范围查询，还能直接利用索引的顺序返回排序后的结果。

下图展示了基于索引的匹配与排序查询原理，其中数字 1 表示升序，-1 表示降序。

https://docs.mongodb.com/manual/_images/index-for-sort.bakedsvg.svg

常见索引类型
MongoDB 提供了多种索引类型，以适应不同的数据模型与查询需求。

1. 默认的 _id 索引
   每个集合在创建时都会自动为 _id 字段建立一个唯一索引，其主要目的是防止 _id 值重复。该索引无法被删除。

2. 单字段索引
   用户可以在单个字段上创建自定义顺序的索引，无论是升序（1）还是降序（-1）均可。

https://docs.mongodb.com/manual/_images/index-ascending.bakedsvg.svg

3. 复合索引
   支持在多个字段上联合创建索引。字段的顺序至关重要，例如索引 { userid: 1, score: -1 } 表示首先按 userid 升序排列，在
   userid 相同的情况下再按 score 降序排列。

4. 多键索引
   当索引的字段值为数组时，MongoDB 会自动为数组中的每个元素创建独立的索引条目，这种索引称为多键索引。它使得查询可以高效地匹配包含特定数组元素的文档。MongoDB
   会自动检测数组字段并创建多键索引，无需显式指定。

https://docs.mongodb.com/manual/_images/index-multikey.bakedsvg.svg

5. 地理空间索引
   为了高效查询地理坐标数据，MongoDB 提供了两种特殊索引：使用平面几何模型的 2d 索引 和使用球面几何模型的 2dsphere 索引。

6. 全文索引
   用于支持对集合中字符串内容的文本搜索。全文索引会过滤掉特定语言的停用词（如 “the”, “a”, “or”），并只存储词干，以减少索引大小并提升搜索相关性。

7. 哈希索引
   主要用于支持基于哈希的分片。哈希索引存储的是字段值的哈希值，其值分布较为随机，但仅支持精确匹配查询，不支持范围查询。

索引的常用操作命令
db.collection.createIndex()：创建索引。

db.collection.dropIndex()：删除指定索引。

db.collection.dropIndexes()：删除集合上的所有索引。

db.collection.getIndexes()：查看集合的所有索引信息。

db.collection.reIndex()：重建集合的所有索引。