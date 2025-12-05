---
title: 在 MongoDB 中更新文档的操作指南
date: 2025-10-30 14:42:34
category: 数据库
tags: MongoDB
---

核心更新方法
MongoDB 提供了多种更新文档的方法，以适应不同场景：

db.collection.updateOne()：更新匹配到的第一条文档（3.2 版本引入）。

db.collection.updateMany()：更新所有匹配到的文档（3.2 版本引入）。

db.collection.replaceOne()：替换匹配到的第一条文档（仅替换整个文档，3.2 版本引入）。

db.collection.update()：经典的更新方法，默认只更新第一条匹配文档，通过配置选项可更新多条。

准备示例数据
首先插入一组用于演示的文档：

javascript
db.inventory.insertMany( [
{ item: "canvas", qty: 100, size: { h: 28, w: 35.5, uom: "cm" }, status: "A" },
{ item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
{ item: "mat", qty: 85, size: { h: 27.9, w: 35.5, uom: "cm" }, status: "A" },
{ item: "mousepad", qty: 25, size: { h: 19, w: 22.85, uom: "cm" }, status: "P" },
{ item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "P" },
{ item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
{ item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
{ item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" },
{ item: "sketchbook", qty: 80, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
{ item: "sketch pad", qty: 95, size: { h: 22.85, w: 30.5, uom: "cm" }, status: "A" }
]);
更新单条文档
以下操作将 item 为 “paper” 的文档的尺寸单位改为 “cm”，状态改为 “P”，并自动更新 lastModified 时间戳：

javascript
db.inventory.updateOne(
{ item: "paper" }, // 查询条件
{
$set: { "size.uom": "cm", status: "P" }, // 更新操作：设置字段值
$currentDate: { lastModified: true } // 更新操作：将指定字段设为当前日期
}
)
$set：用于设置或更新特定字段的值，支持点号访问嵌套字段。

$currentDate：将指定字段的值设置为当前日期或时间。如果字段不存在，则会创建它。

批量更新文档
以下操作将所有库存数量 qty 小于 50 的文档的尺寸单位更新为 “in”，状态更新为 “P”：

javascript
db.inventory.updateMany(
{ "qty": { $lt: 50 } },
{
$set: { "size.uom": "in", status: "P" },
$currentDate: { lastModified: true }
}
)
替换整个文档
replaceOne 方法用于完全替换匹配到的第一条文档（仅保留 _id 不变）：

javascript
db.inventory.replaceOne(
{ item: "paper" }, // 查询条件
// 新文档内容，将完全替换旧文档（除了 _id）
{ item: "paper", instock: [ { warehouse: "A", qty: 60 }, { warehouse: "B", qty: 40 } ] }
)
关键注意事项
原子性：所有针对单条文档的写操作（包括上述更新）都是原子性的。

不可变性：文档的 _id 字段一旦生成，便不可修改。

存储重组：如果更新操作导致文档体积显著增大，MongoDB 可能会在磁盘上重新分配存储空间。

字段顺序：更新操作通常会保持文档最初插入时的字段顺序，但 _id 字段始终位于首位。重命名字段可能导致字段顺序重排。

更多操作符
MongoDB 提供了丰富的更新操作符以满足各种需求，完整列表请参考：官方更新操作符文档