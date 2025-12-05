---
title: Synchronized关键字的五种应用场景详解
date: 2025-10-14 14:42:34
category: 后端
tags: 多线程
---


synchronized 是Java中最基本的线程同步关键字，其核心思想是通过互斥锁来保证同一时刻只有一个线程能访问特定代码或对象。根据锁定的范围不同，synchronized
的用法主要可分为以下五类。

1. 同步实例方法
   将关键字直接修饰在普通成员方法上。

java
public class Counter {
private int value = 0;

    // 锁住整个当前实例对象 (this)
    public synchronized void increment() {
        value++;
    }

    public synchronized int getValue() {
        return value;
    }

}
锁对象：当前实例对象 (this)。
作用范围：同一实例的所有同步实例方法互斥。不同实例间的同步不互相影响。
注意事项：若使用单例模式，则能达到全局互斥效果；否则，需确保多线程操作的是同一个实例。

2. 同步静态方法
   将关键字直接修饰在静态方法上。

java
public class StaticCounter {
private static int value = 0;

    // 锁住类的Class对象 (StaticCounter.class)
    public static synchronized void increment() {
        value++;
    }

    public static synchronized int getValue() {
        return value;
    }

}
锁对象：当前类的 Class 对象（如 StaticCounter.class）。
作用范围：该类的所有同步静态方法之间互斥。这是类级别的锁，所有实例共享，因此能实现跨实例的全局同步。

3. 同步代码块（指定类对象）
   java
   public void doSomething() {
   // 其他非同步代码...
   synchronized (MyClass.class) { // 或 synchronized (this.getClass())
   // 临界区代码
   // 锁对象：MyClass.class
   }
   }
   锁对象：指定的类对象（XXX.class）。
   作用范围：与同步静态方法等效，是类级别的锁。同一时刻，只有一个线程能进入任何以 MyClass.class 为锁的同步块或同步静态方法。

4. 同步代码块（指定当前实例）
   java
   public void doSomething() {
   // 其他非同步代码...
   synchronized (this) {
   // 临界区代码
   // 锁对象：当前实例 (this)
   }
   }
   锁对象：当前实例对象 (this)。
   作用范围：与同步实例方法等效。用于更灵活地控制需要同步的代码段，而非整个方法。

5. 同步代码块（指定任意对象实例）
   java
   public class BankAccount {
   private final Object lock = new Object(); // 专用于锁的私有对象
   private double balance;

   public void transfer(BankAccount to, double amount) {
   // 为了预防死锁，按固定顺序获取锁（例如按hashCode）
   BankAccount first = this.hashCode() < to.hashCode() ? this : to;
   BankAccount second = first == this ? to : this;

        synchronized (first.lock) {
            synchronized (second.lock) {
                if (this.balance >= amount) {
                    this.balance -= amount;
                    to.balance += amount;
                }
            }
        }
   }
   }
   锁对象：任意对象实例（通常是私有、final的成员变量）。
   作用范围：锁定该特定对象。这种方式提供了最细粒度的控制，可以避免用 this 或类对象作为锁时可能导致的无关方法互斥，减少锁竞争，提升性能。

锁的互斥关系总结
同一个锁对象：互斥。线程A持有锁L时，线程B尝试获取锁L会被阻塞。

不同锁对象：不互斥。例如，实例锁 (this) 和类锁 (MyClass.class) 是两个独立的锁，线程可以同时持有它们。

特殊注意：在继承关系中，子类覆盖父类的 synchronized 方法时，synchronized 关键字不会被继承，锁对象依然是子类实例（或子类Class对象）。

选择哪种同步方式，取决于你需要保护的共享资源范围：是单个实例的状态，还是所有实例共享的静态状态，亦或是更复杂的、由特定对象守护的资源。