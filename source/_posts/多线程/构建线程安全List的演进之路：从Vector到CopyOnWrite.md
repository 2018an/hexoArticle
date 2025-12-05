---
title: 构建线程安全List的演进之路：从Vector到CopyOnWrite
date: 2025-10-14 14:42:34
category: 后端
tags: 多线程
---


当面试官追问如何解决 ArrayList 的线程安全问题，仅仅回答 Vector 是远远不够的。这展现了知识面的狭窄。从早期的同步容器到现代的并发容器，Java提供了多种方案，各有其适用场景。

第一代方案：完全同步的容器

1. Vector
   Vector 是Java早期（JDK 1.0）提供的线程安全动态数组。其实现线程安全的方式简单粗暴：在几乎所有公开方法上都加上了
   synchronized 关键字。

java
// Vector内部方法示例
public synchronized boolean add(E e) {
modCount++;
ensureCapacityHelper(elementCount + 1);
elementData[elementCount++] = e;
return true;
}
public synchronized E get(int index) {
if (index >= elementCount)
throw new ArrayIndexOutOfBoundsException(index);
return elementData(index);
}
优点：绝对的线程安全。
缺点：性能差。无论是读还是写操作，都需要获取锁，在高并发读多写少的场景下，会造成大量不必要的线程阻塞。

2. Collections.synchronizedList
   这是一个工厂方法，可以将任何 List 实现包装成一个线程安全的版本。

java
List<Integer> syncList = Collections.synchronizedList(new ArrayList<>());
其内部实现原理与 Vector 类似，持有一个原始的 list 引用和一个互斥锁对象 (mutex)，所有方法都通过 synchronized(mutex) 块进行包装。
优点：更灵活，可以将任何 List（如 ArrayList, LinkedList）转为线程安全，具有良好的兼容性。
缺点：与 Vector 一样，存在严重的读写锁竞争，读性能不佳。

第二代方案：写时复制（Copy-On-Write）并发容器
为应对读多写少的并发场景，Java 5在 java.util.concurrent 包中引入了基于“写时复制”思想的并发集合。

1. CopyOnWriteArrayList
   核心思想：读操作完全无锁，写操作通过复制整个底层数组来实现。

写操作（add, set, remove）：

获取独占锁（ReentrantLock）。

将当前数组复制到一个新的、长度+1（或-1）的数组中。

在新数组上执行修改操作。

将内部数组引用指向新数组。

释放锁。
整个过程在锁保护下进行，但非常快（只做数组拷贝和引用替换）。

读操作（get, iterator）：
直接访问当前内部数组的引用（一个“快照”），无需任何锁。由于读和写操作的是不同的数组，因此不存在读写冲突。

java
// 读操作源码示意
private E get(Object[] a, int index) {
return (E) a[index];
}
public E get(int index) {
return get(getArray(), index); // getArray() 返回当前数组引用
}
优点：极高的读性能，完全不受写线程影响，适合“读多写极少”的场景（如监听器列表、配置快照）。
致命缺点：

1. 内存消耗大：每次写操作都会复制整个数组，如果数组很大，频繁写入会导致频繁的GC，甚至内存溢出（OOM）。
2. 数据弱一致性：读取操作拿到的是写操作发生前的快照，不能立即看到其他线程的最新写入。迭代器也不支持修改操作。

2. CopyOnWriteArraySet
   其内部就是封装了一个 CopyOnWriteArrayList，利用后者的 addIfAbsent 方法来实现去重添加。因此，它继承了
   CopyOnWriteArrayList 的所有优缺点，适用于需要保证元素唯一性的、读多写少的集合场景。

总结与选型建议
向面试官清晰地阐述这条技术演进路径，能体现你的深度思考：

Vector：已过时，不推荐使用。除非在遗留系统中。

Collections.synchronizedList：适用于写多，或者读写操作都频繁，且对数据强一致性要求高的场景。它是通用解决方案。

CopyOnWriteArrayList/CopyOnWriteArraySet：适用于读非常多，写非常少，且能容忍短暂数据不一致性的场景。例如，系统配置、黑/白名单、事件监听器列表等。

额外加分项：提及 ConcurrentLinkedQueue（非阻塞队列）或 LinkedBlockingQueue（阻塞队列），它们提供了不同的线程安全列表语义。