---
title: MongoDB 数据查询操作详解
date: 2025-10-30 14:42:34
category: 数据库
tags: MongoDB
---

准备示例数据
首先向 inventory 集合插入一批示例文档：

javascript
db.inventory.insertMany([
{ item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
{ item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "A" },
{ item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
{ item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
{ item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" }
]);
基础查询操作
查询所有文档
以下两种方式均可返回集合中的所有文档：

javascript
db.inventory.find()
db.inventory.find({})
等值条件查询
查询 status 字段等于 “D” 的所有文档：

javascript
db.inventory.find( { status: "D" } )
范围查询（使用 $in）
查询 status 字段值为 “A” 或 “D” 的文档：

javascript
db.inventory.find( { status: { $in: [ "A", "D" ] } } )
多条件 AND 查询
查询同时满足 status 为 “A” 且 qty 小于 30 的文档：

javascript
db.inventory.find( { status: "A", qty: { $lt: 30 } } )
多条件 OR 查询
查询满足 status 为 “A” 或 qty 小于 30 中任意一个条件的文档：

javascript
db.inventory.find( { $or: [ { status: "A" }, { qty: { $lt: 30 } } ] } )
混合 AND 与 OR 查询
查询 status 为 “A”，并且同时满足 qty 小于 30 或 item 字段以字母 “p” 开头的文档：

javascript
db.inventory.find( {
status: "A",
$or: [ { qty: { $lt: 30 } }, { item: /^p/ } ]
} )
扩展学习
查看所有查询操作符的完整列表与用法：MongoDB 查询操作符

了解如何在查询中使用正则表达式：正则表达式查询