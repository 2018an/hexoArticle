---
title: Java 基本数据类型转换规则
date: 2025-10-14 14:42:34
category: 后端
tags: 基础
---


Java 中的 8 种基本数据类型在一定条件下可相互转换，分为自动转换和强制转换。

自动类型转换
范围小的类型可自动转换为范围大的类型，例如：

java
int a = 100;
long b = a; // 自动转换

double d = 3.14;
float f = (float) d; // 需强制转换
注意数值溢出：

java
int x = 1_000_000_000;
int y = 2_000_000_000;
long sum = (long) x + y; // 先转型再计算，避免溢出
强制类型转换
将范围大的类型转换为范围小的类型时，需显式强制转换，可能丢失精度或溢出。

java
double pi = 3.14159;
int intPi = (int) pi; // 结果为 3

int big = 300;
byte small = (byte) big; // 溢出，结果不可预期
类型提升
在表达式中，所有操作数会自动提升为范围最大的类型：

java
long count = 1000L;
int price = 50;
long total = price * count; // price 自动提升为 long
建议
进行算术运算时注意类型范围，防止溢出。

强制转换前确保值在目标类型范围内。

使用 Math 类方法进行安全的数值处理。

switch 支持的数据类型
switch 语句在 Java 中可用于多种类型的条件匹配，不同版本支持的类型有所扩展。

支持的数据类型
基本类型：byte、short、char、int

包装类型：Byte、Short、Character、Integer

枚举类型：Enum

字符串类型：String（JDK 7+）

示例
java
// 包装类型
Integer code = 200;
switch (code) {
case 200:
System.out.println("成功");
break;
case 404:
System.out.println("未找到");
break;
}

// 枚举类型
enum Status { NEW, PROCESSING, DONE }
Status s = Status.PROCESSING;
switch (s) {
case NEW:
break;
case PROCESSING:
break;
case DONE:
break;
}
注意事项
case 标签必须是常量或字面量。

每个 case 应包含 break 防止穿透。

default 分支可选，最多一个。

JDK 12+ 支持更简洁的语法（case 1, 2 -> ...）。