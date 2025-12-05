---
title: 重入锁（ReentrantLock）深度解析：不只是可重入
date: 2025-10-14 14:42:34
category: 后端
tags: 多线程
---


ReentrantLock 是 java.util.concurrent.locks 包中的核心类，自JDK 1.5引入。它常被称为“重入锁”，但其内涵远不止“可重入”这么简单。它提供了比传统
synchronized 关键字更灵活、更强大的锁控制能力。

何为“重入”？
“重入”意味着同一个线程可以多次获取同一把锁，而不会造成自我死锁。

java
ReentrantLock lock = new ReentrantLock();

public void outer() {
lock.lock();
try {
inner(); // 在持有锁的情况下调用另一个需要同一把锁的方法
} finally {
lock.unlock();
}
}

public void inner() {
lock.lock(); // 同一线程，可以再次获取已经持有的锁
try {
// 访问共享资源
} finally {
lock.unlock();
}
}
如果锁不是可重入的，当线程在 outer() 中调用 inner() 时，inner() 会因无法获取锁（已被自己持有）而永久等待，导致死锁。ReentrantLock
和 synchronized 都具备可重入性。锁内部维护一个计数器，记录重入次数，每次 lock() 递增，每次 unlock() 递减，计数器归零时锁才真正释放。

ReentrantLock的核心能力
作为 Lock 接口的核心实现，ReentrantLock 提供了一系列丰富的方法：

1. 基础锁操作
   void lock(): 获取锁。若锁被其他线程持有，则当前线程休眠等待。

void unlock(): 释放锁。必须在 finally 块中调用，以确保异常时也能释放。

2. 高级获取方式
   void lockInterruptibly() throws InterruptedException: 可中断地获取锁。在等待锁的过程中，线程可以响应中断请求，避免无限期阻塞。

boolean tryLock(): 尝试获取锁，成功返回 true，失败立即返回 false，线程不会阻塞。

boolean tryLock(long time, TimeUnit unit) throws InterruptedException: 带超时的尝试获取锁。在指定时间内尝试，超时或中断则失败。

3. 条件变量支持
   Condition newCondition(): 创建一个与该锁绑定的 Condition 对象。Condition 提供了类似 Object.wait()/notify()
   的等待/通知机制，但功能更强大：一个锁可以关联多个 Condition，实现分组、精准的线程唤醒。

标准使用模式
java
Lock lock = new ReentrantLock();
// ...
lock.lock();
try {
// 访问或修改共享状态
} finally {
lock.unlock(); // 确保释放锁
}
这是必须遵循的模板，以防止锁泄漏导致死锁。

synchronized也是重入锁吗？
是的。synchronized 关键字实现的锁也是可重入的。

java
public class Widget {
public synchronized void doSomething() {
// ...
doSomethingElse(); // 可以调用，因为锁可重入
}

    public synchronized void doSomethingElse() {
        // ...
    }

}
因此，“重入”是 ReentrantLock 的基本属性，而非其独有的特性。它的名字 ReentrantLock 更多是强调其“可重入的锁实现”。

总结：为什么选择ReentrantLock？
选择 ReentrantLock，本质上是为了获取 synchronized 所不具备的高级控制能力：

对中断的响应：使线程在等待锁时能优雅退出。

尝试获取锁：避免死锁或实现更灵活的锁获取策略。

公平性选择：在需要严格顺序的场景下使用公平锁。

多个条件变量：构建复杂的线程协调逻辑（如多个等待队列的生产者-消费者模型）。

如果你的需求仅仅是简单的互斥，那么简洁的 synchronized 通常是更好的选择。但当你的并发控制逻辑变得复杂时，ReentrantLock 及其
Condition 提供的工具箱将展现出不可替代的价值。