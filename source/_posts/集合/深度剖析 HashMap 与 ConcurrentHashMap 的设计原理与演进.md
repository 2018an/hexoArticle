---
title: 深度剖析 HashMap 与 ConcurrentHashMap 的设计原理与演进
date: 2025-10-15 11:36:33
category: 后端
tags: 集合
---

关于 HashMap 和 ConcurrentHashMap 的文章很多，但往往不够详尽，尤其对 Java 8 中 ConcurrentHashMap
的复杂机制阐述不清。本文旨在清晰、透彻地解析其核心细节，特别是 Java 8 的 ConcurrentHashMap，希望能帮助读者降低学习成本，构建完整的知识体系。

阅读建议： 以下四个部分可独立阅读，建议初学者按顺序进行，以降低理解难度：

Java7 HashMap → Java7 ConcurrentHashMap → Java8 HashMap → Java8 ConcurrentHashMap

预备知识： 本文聚焦源码分析，假设读者已熟悉基本接口使用，并对 CAS、ReentrantLock、Unsafe 操作以及红黑树有基本了解。

第一部分：Java 7 的 HashMap
HashMap 结构简单且不支持并发，是理解后续内容的基础。其核心结构如下图所示：

http://qianniu.javastack.cn/18-12-4/65827660.jpg

（示意图未考虑扩容）
整体上，HashMap 内部维护一个数组，数组的每个元素是一个单向链表的头部。上图中每个绿色方块代表一个 Entry 实例，它包含四个属性：key,
value, hash 值以及指向下一个节点的 next 引用。

capacity: 数组容量，始终保持为 2 的 n 次方，可扩容。

loadFactor: 负载因子，默认 0.75。

threshold: 扩容阈值，等于 capacity * loadFactor。

put 过程分析
数组初始化：插入第一个元素时，会初始化数组，容量取大于等于指定初始值的 2 的 n 次方，并计算阈值。

计算数组下标：通过 hash(key) & (length-1) 确定元素应存放的桶位置。

遍历链表：若该位置已有元素（哈希冲突），则遍历链表，判断是否有相同 key（== 或 equals）。若存在，则覆盖旧值并返回。

添加新节点：若未找到相同 key，则创建新 Entry，并采用头插法将其插入链表头部。插入前会检查是否达到扩容阈值。

get 过程分析
根据 key 计算 hash 值。

通过 hash & (length-1) 定位数组下标。

遍历该下标处的链表，通过 == 或 equals 比较 key，直到找到目标节点。

扩容机制
当元素数量超过 threshold 且发生哈希冲突时，会触发扩容。新容量为旧容量的 2 倍。扩容时，会新建一个数组，并将所有原有元素重新哈希（rehash）
分配到新数组中。对于链表中的元素，其在新数组中的位置要么是原索引 i，要么是 i + oldCapacity。

第二部分：Java 7 的 ConcurrentHashMap
为了支持高并发，ConcurrentHashMap 的实现比 HashMap 复杂得多。它采用了分段锁（Segment） 的机制。

http://qianniu.javastack.cn/18-12-4/75121838.jpg

整个 ConcurrentHashMap 由一个 Segment 数组构成。每个 Segment 继承自 ReentrantLock，本身就是一个独立的、类似 HashMap
的结构（数组+链表）。每次加锁只锁住一个 Segment，不同 Segment 的操作可以并行，从而提升了并发度。

concurrencyLevel: 并发级别，默认为 16，即默认有 16 个 Segment。一旦初始化，Segment 数量不可变。

put 过程分析（简略）
计算 key 的 hash 值。

根据 hash 值的高位定位到对应的 Segment。

在 Segment 内部进行加锁，然后执行类似 HashMap 的 put 操作（包括检查扩容）。

Segment 内部的扩容只针对它自己的哈希表（HashEntry 数组），不影响其他 Segment。

get 过程分析
get 操作通常不需要加锁（除非读到的是一个正在扩容过程中迁移的中间状态）。它通过 Unsafe 提供的 getObjectVolatile
方法保证读到的是最新值。其步骤与 HashMap 类似，只是在定位到 Segment 和链表后，直接遍历读取。

并发安全保证
put/remove：通过 Segment 锁保证对同一个桶的写操作互斥。

get：利用 volatile 关键字修饰的 table 数组引用和链表节点的 next 引用，结合 Unsafe 的原子读操作，保证了可见性，实现了无锁读的高性能。

第三部分：Java 8 的 HashMap
Java 8 对 HashMap 进行了重大优化，引入了红黑树，结构变为 数组 + 链表 + 红黑树。

http://qianniu.javastack.cn/18-12-4/75398751.jpg

（示意图，实际数据量早于达到此状态前就会扩容）
当链表的长度超过一定阈值（默认为 8）且数组容量达到一定规模（默认为 64）时，链表会转换为红黑树，将查找时间复杂度从 O(n) 降至 O(
log n)。反之，当红黑树节点数小于等于 6 时，会退化为链表。

核心变化
节点类型：链表节点使用 Node，红黑树节点使用 TreeNode。

put 逻辑：先插入，再判断是否需要扩容（Java 7 是先判断扩容再插入）。链表插入改为尾插法。

扩容优化：扩容时，无需重新计算所有元素的 hash 值。通过判断 (e.hash & oldCap) == 0，可将原链表拆分为两条链表，一条留在原索引
j，一条移到新索引 j + oldCap，保持了元素的相对顺序，避免了 Java 7 中可能造成的死链问题。

第四部分：Java 8 的 ConcurrentHashMap
Java 8 的 ConcurrentHashMap 放弃了分段锁，采用了与 HashMap 更相似的结构（数组+链表+红黑树），但通过 CAS + synchronized
实现了更细粒度的并发控制。

http://qianniu.javastack.cn/18-12-4/335817.jpg

核心改进
锁粒度更细：锁的对象从 Segment 变为每个数组桶（bucket）的头节点（Node）。

数据结构同步：同样使用链表和红黑树解决冲突。

并发控制：

初始化数组、创建空桶等操作用 CAS 实现。

对桶内链表/红黑树的操作，使用 synchronized 锁住头节点。

put 过程简述
计算 hash，定位到桶。

若桶为空，则用 CAS 尝试放入新节点。

若桶不为空（hash == MOVED），则帮助进行数据迁移（扩容过程的一部分）。

否则，使用 synchronized 锁住头节点，然后在链表或红黑树上执行插入操作。

插入后判断是否需要树化或扩容。

扩容机制（关键难点）
Java 8 的 ConcurrentHashMap 扩容设计非常精巧，支持多线程协同扩容。

触发：元素总数超过阈值，或单个链表长度超过 8 但数组容量小于 64。

过程：维护一个 nextTable 作为新数组。通过一个 transferIndex 指针将迁移任务划分成多个小任务段（stride）。多个线程可以并发地领取不同段的任务进行数据迁移，迁移完成的桶会用一个特殊的
ForwardingNode 节点标记。

协助：其他线程在 put 时若发现桶被 ForwardingNode 占据，则会加入帮助迁移的行列。

get 过程
与 HashMap 的 get 逻辑几乎一致，全程无锁。得益于 volatile 修饰的 table 和节点的 val、next，保证了读到的总是最新数据。

总结
从 Java 7 到 Java 8，HashMap 和 ConcurrentHashMap 在追求更高性能的道路上不断演进。HashMap
通过引入红黑树优化了极端情况下的查询效率；ConcurrentHashMap 则通过缩小锁粒度、利用 CAS 和 synchronized
的优化，以及设计巧妙的多线程协作扩容机制，在保证线程安全的同时，大幅提升了并发性能。理解这些设计思想的演变，比单纯记忆源码细节更为重要。