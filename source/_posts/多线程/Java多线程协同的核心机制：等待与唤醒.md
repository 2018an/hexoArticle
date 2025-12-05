---
title: Java多线程协同的核心机制：等待与唤醒
date: 2025-10-14 14:42:34
category: 后端
tags: 多线程
---


在Java多线程编程中，线程间的协调与通信是构建稳定并发程序的关键。除了共享内存与锁机制外，Java还提供了一组基于对象监视器（Monitor）的通信原语：wait(),
notify(), 以及 notifyAll()。这三个方法共同构成了线程间“等待-通知”模式的基础。

方法定义与归属
wait()：使当前线程暂停执行并进入等待状态，同时释放其持有的对象锁。线程将停留在对象的等待集中，直到被其他线程唤醒。

notify()：从在该对象上等待的线程中，任意选择一个并将其移出等待集，使其重新进入锁竞争状态。

notifyAll()：唤醒在该对象上等待的所有线程，使它们全部移出等待集并重新竞争锁。

这三个方法并非定义在 Thread 类中，而是位于 Object
类。这是因为锁是关联于对象实例的（每个对象都有一个内置监视器锁），线程通过获取对象的锁来进入同步区域。因此，让线程等待或唤醒的操作自然应该由锁的持有者——对象来提供。

http://qianniu.javastack.cn/18-6-1/82637503.jpg

wait(long timeout) 方法允许设置一个最大等待时长。若超时后仍未被唤醒，线程会自动恢复并尝试重新获取锁。

关键使用原则
必须在同步上下文中调用：调用这些方法的线程必须已经获得了该对象的监视器锁，即只能在 synchronized 方法或同步代码块内部使用。

wait() 会释放锁：调用 wait() 后，线程不仅暂停，还会释放其持有的对象锁，这是实现有效协调的前提。

优先使用 notifyAll()：notify() 随机唤醒一个线程，可能导致某些线程“饥饿”或死锁。通常更安全的选择是使用 notifyAll()
，唤醒所有等待线程，让它们公平竞争。

实践示例
java
public class WaitNotifyDemo {
public static void main(String[] args) {
final Object coordinator = new Object();

        Thread waiter = new Thread(() -> {
            synchronized (coordinator) {
                for (int i = 1; i <= 5; i++) {
                    System.out.println("Waiter: " + i);
                    if (i == 3) {
                        try {
                            System.out.println("Waiter: 即将等待...");
                            coordinator.wait(); // 释放锁并等待
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
                System.out.println("Waiter: 继续完成剩余工作。");
            }
        });

        Thread notifier = new Thread(() -> {
            synchronized (coordinator) {
                try {
                    System.out.println("Notifier: 休眠2秒，模拟工作。");
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("Notifier: 工作完成，唤醒等待者。");
                coordinator.notifyAll(); // 唤醒所有等待线程
            }
        });

        waiter.start();
        notifier.start();
    }

}
输出可能类似于：

text
Waiter: 1
Waiter: 2
Waiter: 3
Waiter: 即将等待...
Notifier: 休眠2秒，模拟工作。
Notifier: 工作完成，唤醒等待者。
Waiter: 4
Waiter: 5
Waiter: 继续完成剩余工作。
线程 waiter 在输出到3时进入等待，释放锁。线程 notifier 获取锁后执行任务，最后通过 notifyAll() 唤醒 waiter，使其继续执行。

