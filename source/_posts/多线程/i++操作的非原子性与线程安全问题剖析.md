---
title: i++操作的非原子性与线程安全问题剖析
date: 2025-10-14 09:33:33
category: 后端
tags: 多线程
---


i++（自增操作）是一个在Java面试中频繁出现的问题，其核心在于探讨这一看似简单的操作在并发环境下的安全性。许多开发者可能会直觉地认为这是一个原子操作，但事实恰恰相反。

并发环境下的问题演示
java
public class IncrementDemo {
private static int counter = 0;
private static final int THREAD_COUNT = 1000;
private static final int PER_THREAD_INCREMENT = 1000;

    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(THREAD_COUNT);
        Runnable incrementTask = () -> {
            for (int j = 0; j < PER_THREAD_INCREMENT; j++) {
                counter++; // 非原子操作！
            }
            latch.countDown();
        };

        for (int i = 0; i < THREAD_COUNT; i++) {
            new Thread(incrementTask).start();
        }
        latch.await(); // 等待所有线程完成
        System.out.println("预期结果: " + (THREAD_COUNT * PER_THREAD_INCREMENT));
        System.out.println("实际结果: " + counter);
    }

}
运行此程序多次，几乎不可能得到预期的 1000000。结果通常小于该值，且每次运行都可能不同。这直观地证明了 i++ 不是线程安全的。

根源分析：Java内存模型与操作非原子性

1. i++ 的真实步骤
   i++ 并非单一指令，它对应三个独立的步骤：

读取：从主内存（或工作内存缓存）读取变量 i 的当前值到线程的工作内存。

增加：在工作内存中将读取到的值加1。

写回：将增加后的新值写回主内存。

2. 并发冲突场景
   假设 i 初始值为5，两个线程A和B几乎同时执行 i++：

时刻1：线程A和B都从主内存读取到 i=5。

时刻2：线程A计算得6，并写回主内存，此时主内存 i=6。

时刻3：线程B（仍基于旧值5）计算得6，并写回主内存，再次将 i 设置为6。
结果，尽管执行了两次自增，i 的最终值却是6，而不是7。这就是 更新丢失 问题。

3. volatile 关键字能解决吗？
   将 counter 声明为 volatile 只能保证可见性和有序性，无法保证原子性。

可见性：当一个线程修改了 volatile 变量，新值会立即对其他线程可见。

原子性：i++ 的“读-改-写”复合操作本身依然不是原子的。volatile 无法阻止上述“读取旧值-计算-写回”过程中的交错执行。

解决方案
方案一：使用同步锁
java
synchronized (lockObject) {
counter++;
}
或使用 ReentrantLock。这确保了同一时刻只有一个线程能执行自增操作，但可能引入性能开销。

方案二：使用原子类 (推荐)
java.util.concurrent.atomic 包提供了基于CAS（Compare-And-Swap）乐观锁的原子类，性能通常优于悲观锁。

java
import java.util.concurrent.atomic.AtomicInteger;

private static AtomicInteger atomicCounter = new AtomicInteger(0);

// 在线程中
atomicCounter.incrementAndGet(); // 原子自增
AtomicInteger.incrementAndGet() 方法通过CPU底层的原子指令保证了自增操作的原子性，是解决此类问题的首选方案。