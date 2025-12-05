---
title: 向 MongoDB 中插入数据的多种方式
date: 2025-10-30 14:42:34
category: 数据库
tags: MongoDB
---

核心插入方法
MongoDB 提供了以下三种主要的数据插入方法：

db.collection.insertOne()：插入单条文档（自 MongoDB 3.2 起支持）。

db.collection.insertMany()：插入多条文档（自 MongoDB 3.2 起支持）。

db.collection.insert()：通用插入方法，可插入单条或多条文档。

插入单条文档
使用 insertOne() 方法插入单个文档：

javascript
db.inventory.insertOne(
{ item: "canvas", qty: 100, tags: ["cotton"], size: { h: 28, w: 35.5, uom: "cm" } }
)
语法说明：

javascript
db.collection.insertOne(
<document>,
{
writeConcern: <document> // 可选，写关注配置
}
)
注意：insertOne() 方法不支持 db.collection.explain() 执行计划分析，若需分析，请使用 insert() 方法。

批量插入文档
使用 insertMany() 方法一次性插入多个文档：

javascript
db.inventory.insertMany([
{ item: "journal", qty: 25, tags: ["blank", "red"], size: { h: 14, w: 21, uom: "cm" } },
{ item: "mat", qty: 85, tags: ["gray"], size: { h: 27.9, w: 35.5, uom: "cm" } },
{ item: "mousepad", qty: 25, tags: ["gel", "blue"], size: { h: 19, w: 22.85, uom: "cm" } }
])
语法说明：

javascript
db.collection.insertMany(
[ <document 1>, <document 2>, ... ], // 文档数组
{
writeConcern: <document>, // 可选，写关注配置
ordered: <boolean>        // 可选，是否按顺序插入，默认为 true
}
)
重要注意事项
集合自动创建：执行插入操作时，如果目标集合不存在，MongoDB 会自动创建它。

主键 _id：每个文档必须拥有唯一的 _id 字段作为主键。如果插入时未指定，MongoDB 会自动生成一个 ObjectId。

原子性保证：所有针对单条文档的写操作（包括 insertOne）都是原子性的。