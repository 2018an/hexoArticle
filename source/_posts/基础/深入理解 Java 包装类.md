---
title: 深入理解 Java 包装类
date: 2025-10-14 14:42:34
category: 后端
tags: 基础
---


包装类（Wrapper Classes）将基本类型封装为对象，使基本类型能参与面向对象操作。

对应关系
基本类型 包装类
byte Byte
short Short
int Integer
long Long
float Float
double Double
char Character
boolean Boolean
主要用途
集合泛型
集合不能存储基本类型，必须使用包装类。

允许空值
成员变量或方法参数可能需要表示“无值”状态。

java
private Integer score; // 可为 null，表示“未评分”
数值边界处理
避免基本类型的默认值（如 0）被误认为有效值。

自动装箱与拆箱
Java 5 引入自动转换机制：

java
Integer a = 100; // 自动装箱：Integer.valueOf(100)
int b = a; // 自动拆箱：a.intValue()

// 实际等价于：
Integer a = Integer.valueOf(100);
int b = a.intValue();
注意事项
缓存范围：Integer 默认缓存 -128 ~ 127 的对象，该范围内 == 比较可能为 true，范围外建议用 equals。

性能影响：大量装箱拆箱可能产生额外对象，影响性能。