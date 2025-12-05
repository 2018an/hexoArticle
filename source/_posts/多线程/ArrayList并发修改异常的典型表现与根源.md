---
title: ArrayList并发修改异常的典型表现与根源
date: 2025-10-14 14:42:34
category: 后端
tags: 多线程
---


ArrayList 作为Java中最常用的集合类之一，其非线程安全的特性是并发编程中的经典考点。理解其在多线程环境下可能出现的各种异常现象，有助于我们加深对线程安全必要性的认识。

并发问题复现
java
import java.util.ArrayList;
import java.util.List;

public class UnsafeArrayListDemo {
private static List<Integer> numberList = new ArrayList<>();

    public static void main(String[] args) throws InterruptedException {
        // 模拟10次并发测试
        for (int testRound = 0; testRound < 10; testRound++) {
            testConcurrentAdd();
            numberList.clear(); // 清空为下一轮测试准备
        }
    }

    private static void testConcurrentAdd() throws InterruptedException {
        Runnable addTask = () -> {
            for (int i = 0; i < 1000; i++) {
                numberList.add(i); // 并发添加元素
            }
        };

        Thread t1 = new Thread(addTask);
        Thread t2 = new Thread(addTask);
        Thread t3 = new Thread(addTask);

        t1.start();
        t2.start();
        t3.start();

        t1.join();
        t2.join();
        t3.join();

        // 理论上，3个线程各加1000次，总数应为3000
        System.out.println("本轮最终列表大小: " + numberList.size());
    }

}
多次运行上述程序，你可能会观察到以下几种不同的结果：

并发问题的三种典型现象

1. 抛出 ArrayIndexOutOfBoundsException 异常
   这是最直接的错误表现。异常栈通常会指向 ArrayList.add 方法内部与数组扩容或索引赋值相关的代码行。其根本原因是：多个线程同时进行添加操作时，对
   ArrayList 内部维护的数组 elementData 和表示大小的 size 变量的修改出现了竞争。

线程A和B同时检测到需要扩容，但扩容逻辑（创建新数组并复制）可能被交叉执行，导致索引计算错误。

线程A增加了size，但线程B在其更新数组元素之前使用了相同的size作为索引，导致数组越界。

2. 程序“正常”结束，但最终元素数量少于预期（例如20332, 16100）
   这是“数据覆盖”或“更新丢失”的体现。两个线程可能读取到相同的数组尾部索引位置，然后都向该位置写入新元素。后写入的线程会覆盖前一个线程写入的值，并且
   size 可能只被递增了一次，导致元素总数减少。

3. 程序“正常”结束，且得到预期数量（30000）
   这只是一种幸运情况，依赖于特定的线程执行时序。在多核处理器和高并发场景下，这种“正确”结果的概率极低，完全不可依赖。

核心源码剖析（基于OpenJDK简化逻辑）
ArrayList.add(E e) 方法的关键步骤：

java
public boolean add(E e) {
ensureCapacityInternal(size + 1); // 步骤1：检查并可能扩容
elementData[size] = e; // 步骤2：在size位置赋值
size = size + 1; // 步骤3：递增size
return true;
}
在无同步保护下，上述三个步骤（尤其是2和3）不是原子的。多个线程交叉执行，就会导致前述的数组越界或数据覆盖问题。

结论与启示
ArrayList 的设计初衷并非用于并发场景。其内部状态（数组和大小）在多线程下的非原子性修改必然导致不确定的行为。同样，HashMap、HashSet
等大多数标准集合框架实现也都不是线程安全的。
解决方案方向：

使用同步包装器：Collections.synchronizedList(new ArrayList<>())。

使用并发集合：如 CopyOnWriteArrayList（读多写少场景）、ConcurrentLinkedQueue 等。

在方法内部使用局部 ArrayList，避免共享。