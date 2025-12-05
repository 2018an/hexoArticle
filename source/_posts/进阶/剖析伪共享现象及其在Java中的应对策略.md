---
title: 剖析伪共享现象及其在Java中的应对策略
date: 2025-10-15 11:36:33
category: 后端
tags: 进阶
---

1. 伪共享概念
   CPU缓存以缓存行（通常64字节）为单位存储数据。当多个线程修改位于同一缓存行的不同变量时，会无意中导致缓存行无效，引发频繁的缓存同步，这种现象称为伪共享（False
   Sharing）。

2. 缓存体系结构
   现代CPU通常具备三级缓存（L1、L2、L3），速度逐级递减，容量逐级增大。L1、L2为核内私有，L3为多核共享。数据访问时依次查找各级缓存，未命中则访问主内存。

https://img/18-5-31-91078691.jpg

3. MESI缓存一致性协议
   MESI定义了缓存行的四种状态：

M（Modified）：已修改，与主存不一致，仅存在于当前缓存。

E（Exclusive）：独占，与主存一致，仅存在于当前缓存。

S（Shared）：共享，与主存一致，可能存在于多个缓存。

I（Invalid）：无效。

当某核修改共享数据时，其他核中对应缓存行状态将变为I，需重新从主存加载。

https://img/18-5-31-66429246.jpg

4. Java中的传统解决方案
   通过填充无用字段，使单个对象独占一个缓存行，避免多个变量共享同一缓存行。

text
public final static class VolatileLong {
public long p1, p2, p3, p4, p5, p6, p7; // 填充
public volatile long value = 0L;
public long p8, p9, p10, p11, p12, p13, p14; // 填充
}

5. Java 8的官方支持
   Java 8引入 @sun.misc.Contended 注解，标注的类会自动进行缓存行填充。需在JVM启动参数中添加 -XX:-RestrictContended 启用该功能。

text
@sun.misc.Contended
public final static class VolatileLong {
public volatile long value = 0L;
}
理解伪共享对编写高性能并发程序至关重要，尤其在频繁修改共享变量的场景中。