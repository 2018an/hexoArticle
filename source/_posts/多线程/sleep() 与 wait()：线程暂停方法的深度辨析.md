---
title: sleep() 与 wait()：线程暂停方法的深度辨析
date: 2025-10-14 14:42:34
category: 后端
tags: 多线程
---


sleep() 和 wait() 是Java多线程中用于暂停线程执行的两种核心方法。虽然它们都能让线程“停下来”，但其设计目的、使用机制和底层行为存在本质区别。

五大核心区别详解
对比维度 Thread.sleep(long millis)    Object.wait(long timeout)

1. 方法与归属 Thread 类的静态本地方法。 Object 类的实例本地方法。
2. 调用限制 可在任何地方调用，无需获取锁。但需捕获 InterruptedException。 必须在同步代码块或同步方法中调用，且调用线程必须已获得该对象的监视器锁（monitor
   lock）。同样需捕获 InterruptedException。
3. 锁的行为 不会释放 任何已持有的锁。线程休眠时仍持有锁。 会释放 调用该方法的对象锁。这是实现线程间协调的关键。
4. 唤醒机制 休眠指定时间后自动恢复，或可被中断。 1) 超时后自动恢复；2) 被中断；3) 必须由其他线程调用同一对象的 notify() 或
   notifyAll() 来唤醒。唤醒后需重新竞争锁。
5. 设计目的 单纯地让当前线程暂停执行一段时间，用于计时、节流等。 专为线程间通信设计，是等待-通知（Wait-Notify）模式的基础。通常用于条件不满足时让线程等待。
   源码与设计哲学
   为何sleep在Thread类，而wait在Object类？

sleep 作用于线程本身，与特定对象无关。它让当前执行线程进入“定时等待”状态，不涉及对象锁的交互，因此作为 Thread 的静态方法很合理。

wait 作用于对象监视器。它的语义是：“当前线程暂时停止，并释放这个对象的锁，直到某个条件发生”。这个“条件”通常由另一线程通过操作同一个对象来改变。因此，它是对象级别的行为，定义在
Object 中。

代码示例：对比锁的释放行为
java
public class SleepVsWaitDemo {
private static final Object LOCK = new Object();

    public static void main(String[] args) throws InterruptedException {
        Thread sleepThread = new Thread(() -> {
            synchronized (LOCK) {
                System.out.println("SleepThread 获取了锁，即将sleep 2秒。");
                try {
                    Thread.sleep(2000); // 休眠，但不释放LOCK
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("SleepThread 从sleep中醒来，并释放锁。");
            }
        });

        Thread waitThread = new Thread(() -> {
            synchronized (LOCK) {
                System.out.println("WaitThread 获取了锁，即将wait 2秒。");
                try {
                    LOCK.wait(2000); // 等待，并释放LOCK
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("WaitThread 从wait中被唤醒或超时，并重新获取锁。");
            }
        });

        // 先启动sleepThread，它持有锁去sleep
        sleepThread.start();
        Thread.sleep(100); // 确保sleepThread先启动
        // 再启动waitThread，它将无法立即获取锁，因为锁被sleepThread持有且不释放
        waitThread.start();

        sleepThread.join();
        waitThread.join();
    }

}
输出分析：

text
SleepThread 获取了锁，即将sleep 2秒。
// 等待2秒...
SleepThread 从sleep中醒来，并释放锁。
WaitThread 获取了锁，即将wait 2秒。
// 可能等待2秒（超时）...
WaitThread 从wait中被唤醒或超时，并重新获取锁。
可以看到，waitThread 在 sleepThread 持有锁并 sleep 期间，根本无法进入同步块。而如果两个线程都使用 wait，则它们可以先后释放锁，允许对方执行。

总结与选用建议
需要单纯延迟：用 sleep()。

需要实现线程间协作，等待某个条件：用 wait()/notify() 机制，并配合 synchronized。

切记 wait() 必须在同步上下文中使用，且通常应放在检查条件的循环中，以防止“虚假唤醒”。

