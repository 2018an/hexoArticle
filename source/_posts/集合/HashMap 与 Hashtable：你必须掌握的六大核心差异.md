---
title: HashMap 与 Hashtable：你必须掌握的六大核心差异
date: 2025-10-15 11:36:33
category: 后端
tags: 集合
---

作为 Java 开发者，HashMap 和 Hashtable 是面试与工作中绕不开的话题。但你是否能清晰地说出它们的主要区别呢？本文将为你系统梳理两者的六大关键不同点。

1. 线程安全性
   这是最根本的区别。

Hashtable 是线程安全的。其几乎所有公开方法（如 put, get, remove）都使用 synchronized 关键字修饰，这意味着在多线程环境下，同一时刻只有一个线程能操作该实例。

HashMap 是非线程安全的。在多线程环境下直接使用可能导致数据不一致等问题。

2. 性能表现
   线程安全性的代价是性能。

由于 synchronized 锁住了整个实例，Hashtable 在高并发场景下竞争激烈，性能较差。

HashMap 没有同步开销，因此在单线程或线程安全已由外部保障的情况下，性能远优于 Hashtable。

替代方案：如需兼顾线程安全与性能，应优先选择 java.util.concurrent.ConcurrentHashMap。

3. 对 Null 键和 Null 值的支持
   Hashtable：既不支持 key 为 null，也不支持 value 为 null。若尝试存入 null，会直接抛出 NullPointerException。

HashMap：允许 key 为 null（唯一），也允许多个 value 为 null。其 hash() 方法对 null key 做了特殊处理，将其哈希值定义为 0。

源码对比：
Hashtable 的 put 方法中明确检查 value 不为 null，并在计算 key.hashCode() 时，若 key 为 null 也会抛出 NPE。
HashMap 的 hash() 方法包含三元运算符：return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);

4. 继承体系与诞生背景
   Hashtable：继承自一个古老的 Dictionary 类。它是 JDK 1.0 的产物，属于陈旧的 API。

HashMap：继承自较新的 AbstractMap 类。它是 Java 集合框架（Java Collections Framework, JCF）的一部分，自 JDK 1.2 引入，设计更现代。

5. 初始容量与扩容策略
   默认初始容量：

Hashtable: 11

HashMap: 16

负载因子：默认均为 0.75。

扩容公式（当 size > threshold 时）：

Hashtable: newCapacity = oldCapacity * 2 + 1

HashMap: newCapacity = oldCapacity * 2

HashMap 的容量始终保持为 2 的 n 次方，这与其使用 (n-1) & hash 计算下标的方式（位运算替代取模，效率更高）密切相关。

6. 迭代器的 fail-fast 行为
   HashMap 的 Iterator 是 fail-fast 的。如果在迭代过程中，其他线程（或本线程的其他操作）通过非迭代器自身的 remove 等方法修改了
   Map 的结构（增删元素），迭代器会立即抛出 ConcurrentModificationException。

Hashtable 的 Enumerator（其旧式迭代器）不是 fail-fast 的。在迭代过程中进行结构性修改可能不会立即引发异常，但行为是未定义的，可能导致不可预料的结果。

这实际上也反映了较新的 Iterator 与陈旧的 Enumerator 在故障感知上的设计差异。

示例演示（单线程环境，模拟结构性修改）：

text
// Hashtable 使用 Enumerator，移除元素后迭代继续（但结果可能混乱）
Hashtable<String, String> ht = new Hashtable<>();
// ... 添加元素
Enumeration<String> en = ht.elements();
ht.remove(someKey); // 在迭代外部移除
while (en.hasMoreElements()) { // 可能继续迭代，但行为不确定
System.out.println(en.nextElement());
}

// HashMap 使用 Iterator，移除元素后立即抛出异常
HashMap<String, String> hm = new HashMap<>();
// ... 添加元素
Iterator<String> it = hm.values().iterator();
hm.remove(someKey); // 在迭代外部移除
while (it.hasNext()) { // 抛出 ConcurrentModificationException
System.out.println(it.next());
}
了解这些区别，不仅有助于在面试中从容应对，更能指导我们在实际开发中做出正确的技术选型。