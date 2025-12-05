---
title: UUID：通用唯一标识符
date: 2025-10-14 14:42:34
category: 后端
tags: 基础
---


UUID（Universally Unique Identifier）是一个 128 位的全局唯一标识符，常用于分布式系统中生成唯一 ID。

格式与示例
标准 UUID 形如：
550e8400-e29b-41d4-a716-446655440000
共 32 位十六进制数，分为 5 组：8-4-4-4-12。

生成方式
java
import java.util.UUID;

public class UUIDDemo {
public static void main(String[] args) {
UUID uuid = UUID.randomUUID();
System.out.println(uuid.toString());
}
}
特点与用途
唯一性：理论重复概率极低。

无需中心协调：各节点可独立生成。

应用场景：数据库主键、文件命名、会话标识、分布式追踪等。

注意
字符串形式较长（36 字符），存储和传输有一定开销。

无序性，不适合直接作数据库索引（可考虑有序 UUID 变种）。