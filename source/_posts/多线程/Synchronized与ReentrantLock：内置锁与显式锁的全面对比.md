---
title: Synchronized与ReentrantLock：内置锁与显式锁的全面对比
date: 2025-10-14 14:42:34
category: 后端
tags: 多线程
---


在Java并发编程中，synchronized 关键字和 ReentrantLock 类是两种最核心的互斥同步工具。理解它们的异同，对于做出正确的技术选型至关重要。

对比总览
特性维度 synchronized (内置锁/监视器锁)    ReentrantLock (显式锁)
实现层级 JVM层面实现，属于语言原生特性。 JDK层面实现，基于 java.util.concurrent.locks 包。
锁的获取与释放 自动管理。进入同步块自动获取，退出（正常或异常）自动释放。 手动控制。必须显式调用 lock() 和 unlock()，通常将
unlock() 置于 finally 块。
可重入性 支持。同一线程可多次进入。 支持。同一线程可多次锁定，需对应次数的解锁。
锁的公平性 仅支持非公平锁（默认，吞吐量高）。 支持公平与非公平锁（通过构造器参数选择）。公平锁按等待顺序获取，减少“饥饿”，但性能较低。
性能 早期性能较差。JDK 1.6后引入“锁升级”（偏向->轻量级->重量级）优化，在低竞争下性能与ReentrantLock接近。
性能稳定。在高竞争场景下，其可伸缩性通常更好。
功能灵活性 基础。提供基本的互斥和等待/通知（wait/notify）。 丰富。提供诸多高级功能。
中断响应 不支持。线程在等待锁时无法被中断，会一直阻塞。 支持。lockInterruptibly() 方法允许在等待锁时响应中断。
尝试获取锁 不支持。要么获得，要么阻塞。 支持。tryLock() 尝试获取，失败立即返回或等待指定时间。
条件队列 每个锁对象只有一个隐式的等待条件（通过wait/notify）。 可关联多个Condition对象，实现更精细的线程分组唤醒（如生产者-消费者模型）。
锁绑定 与对象头中的Mark Word绑定。 是独立的对象。
ReentrantLock 的三大独有功能详解

1. 公平性选择
   java
   // 非公平锁（默认，吞吐量高，但可能产生线程饥饿）
   ReentrantLock unfairLock = new ReentrantLock();
   // 公平锁（按FIFO顺序获取锁，吞吐量可能下降，但公平）
   ReentrantLock fairLock = new ReentrantLock(true);
2. 可中断的锁获取
   java
   ReentrantLock lock = new ReentrantLock();
   try {
   lock.lockInterruptibly(); // 此方法可被中断
   // ... 访问共享资源
   } catch (InterruptedException e) {
   // 处理中断，执行清理或退出
   Thread.currentThread().interrupt();
   } finally {
   if (lock.isHeldByCurrentThread()) {
   lock.unlock();
   }
   }
3. 尝试锁与超时
   java
   if (lock.tryLock()) { // 尝试立即获取
   try {
   // 获取成功，操作共享资源
   } finally {
   lock.unlock();
   }
   } else {
   // 获取失败，执行替代逻辑
   }

// 或带超时的尝试
if (lock.tryLock(1, TimeUnit.SECONDS)) {
try {
// ...
} finally {
lock.unlock();
}
} else {
// 超时未获取，执行降级策略
}

4. 多个条件变量（Condition）
   这是 ReentrantLock 最强大的特性之一，可以替代 Object.wait/notify，实现更精确的控制。

java
class BoundedBuffer {
final ReentrantLock lock = new ReentrantLock();
final Condition notFull = lock.newCondition(); // 条件：不满
final Condition notEmpty = lock.newCondition(); // 条件：不空

    public void put(Object x) throws InterruptedException {
        lock.lock();
        try {
            while (/* 队列满 */) {
                notFull.await(); // 在“不满”条件上等待
            }
            // ... 入队
            notEmpty.signal(); // 唤醒一个在“不空”条件上等待的消费者
        } finally {
            lock.unlock();
        }
    }

    public Object take() throws InterruptedException {
        lock.lock();
        try {
            while (/* 队列空 */) {
                notEmpty.await(); // 在“不空”条件上等待
            }
            // ... 出队
            notFull.signal(); // 唤醒一个在“不满”条件上等待的生产者
            return item;
        } finally {
            lock.unlock();
        }
    }

}
synchronized 只有一个等待集，调用 notify() 时无法指定唤醒生产者还是消费者。而 Condition 允许将不同的等待线程分组到不同的条件队列，实现精准通知。

选型建议
优先使用 synchronized：对于大多数简单的同步场景，其简洁性和自动管理特性是首选。JVM的持续优化使其性能不再落后。

考虑 ReentrantLock 当需要：

可中断的锁等待。

带超时的锁尝试。

需要实现公平锁策略。

需要多个条件谓词进行复杂的线程协作（如复杂的生产者-消费者模型）。

需要在 try-catch 块外获得锁，并在不同地方释放。

记住：能力越大，责任越大。ReentrantLock 的灵活性带来了手动管理的复杂性，务必在 finally 块中确保锁的释放，否则会导致死锁。