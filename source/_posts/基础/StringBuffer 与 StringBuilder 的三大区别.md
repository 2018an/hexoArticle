---
title: StringBuffer 与 StringBuilder 的三大区别
date: 2025-10-14 14:42:34
category: 后端
tags: 基础
---


虽然两者都用于可变字符串操作，但在线程安全、内部缓存和性能上存在差异。

1. 线程安全性
   StringBuffer 所有公开方法都使用 synchronized 修饰，线程安全。

StringBuilder 未同步，线程不安全。

java
// StringBuffer 中的方法
public synchronized StringBuffer append(String str) {
// ...
}

2. 内部缓存机制
   StringBuffer 内部维护了一个 toStringCache 数组，在 toString() 时复用，避免重复拷贝。

java
// StringBuffer
public synchronized String toString() {
if (toStringCache == null) {
toStringCache = Arrays.copyOfRange(value, 0, count);
}
return new String(toStringCache, true);
}

// StringBuilder
public String toString() {
return new String(value, 0, count); // 每次创建新数组
}

3. 性能差异
   由于 StringBuffer 的同步开销，单线程环境下 StringBuilder 性能更优。

使用建议
多线程共享字符串时用 StringBuffer。

单线程或局部变量场景用 StringBuilder。