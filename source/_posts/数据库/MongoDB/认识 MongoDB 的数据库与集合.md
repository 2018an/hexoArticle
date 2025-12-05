---
title: 认识 MongoDB 的数据库与集合
date: 2025-10-30 14:42:34
category: 数据库
tags: MongoDB
---

https://docs.mongodb.com/manual/_images/crud-annotated-collection.bakedsvg.svg

数据库（Databases）
在 MongoDB 中，数据库是一组集合的逻辑容器，类似于关系型数据库中的“数据库”概念，用于组织和管理相关的数据集合。

创建与切换数据库
使用 use 命令可以创建或切换数据库。如果数据库不存在，则会先创建再切换；如果已存在，则直接切换。

javascript
use my_database;
集合（Collections）
集合相当于关系型数据库中的“表”，用于存储一组文档。但与传统表不同，MongoDB 的集合是“无模式”的，意味着其中的文档可以拥有不同的结构。

动态创建集合
MongoDB 采用“懒创建”机制，在以下两种常见操作时，如果集合不存在，会自动创建它：

向集合插入第一条文档：

javascript
db.myNewCollection1.insertOne( { x: 1 } )
为集合创建第一个索引：

javascript
db.myNewCollection2.createIndex( { y: 1 } )
此外，也可以使用 db.createCollection() 方法显式创建集合，该方法允许指定更多选项，如设置集合的最大容量、启用文档验证规则等。对于大多数简单场景，上述的自动创建方式已足够。

文档验证（Document Validation）
自 MongoDB 3.2 起，引入了文档验证功能。默认情况下，同一个集合中的文档可以拥有完全不同的字段和数据类型，这提供了极大的灵活性。

然而，在某些场景下，我们需要对数据的结构进行约束。可以通过定义验证规则来实现：

javascript
db.createCollection( "contacts",
{ validator: { $or:
[
{ phone: { $type: "string" } },
{ email: { $regex: /@mongodb\.com$/ } },
{ status: { $in: [ "Unknown", "Incomplete" ] } }
]
}
} )
验证规则可以在插入或更新文档时执行。可以配置为拒绝不符合规则的写入操作，也可以配置为仅记录违规日志但仍允许写入。

了解更多关于文档验证的细节，请参考：官方文档验证指南

修改文档结构
MongoDB 允许灵活地更新已有文档的结构，例如：

添加新的字段。

删除现有的字段。

修改某个字段的数据类型。
这种灵活性使得应用能够随着需求演进而平滑地调整数据模型。