---
title: start() 与 run()：启动线程与同步调用的本质区别
date: 2025-10-14 14:42:34
category: 后端
tags: 多线程
---


在Java多线程编程中，start() 和 run() 是与线程执行密切相关的两个方法，初学者极易混淆。理解它们之间的区别，是掌握Java线程模型的基础。

角色定位
run() 方法：定义在 java.lang.Runnable 接口中，是线程需要执行的任务逻辑的主体。无论是通过继承 Thread 类还是实现 Runnable
接口创建线程，都必须覆盖或实现此方法。

start() 方法：定义在 java.lang.Thread 类中。它的职责是启动一个新的执行线程。调用此方法后，JVM会在底层创建一个新的操作系统线程（或映射到内核线程），然后由这个新线程去调用
run() 方法。

核心区别：异步 vs 同步
调用 start() 是异步的：主线程（调用者）在发出启动指令后立即返回，继续执行后续代码。新线程被放入“就绪队列”，等待操作系统调度器分配CPU时间片。一旦获得执行权，新线程便开始独立执行其
run() 方法中的代码。主线程和新线程是并发执行的。

调用 run() 是同步的：这仅仅是普通的方法调用。当前线程（通常是主线程）会直接执行 run()
方法体中的代码，并在此方法返回后才继续执行。这并没有创建新的线程，完全是在单线程环境下按顺序执行。

代码示例：对比两者的行为
java
public class StartVsRunDemo {
public static void main(String[] args) {
System.out.println("主线程开始。");

        Thread workerThread = new Thread(() -> {
            System.out.println("子线程开始工作，线程ID: " + Thread.currentThread().getId());
            try {
                Thread.sleep(2000); // 模拟耗时任务
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("子线程工作完成。");
        });

        System.out.println("\n--- 调用 start() ---");
        long startTime = System.currentTimeMillis();
        workerThread.start(); // 异步，主线程立刻返回
        long endTime = System.currentTimeMillis();
        System.out.println("主线程调用start()耗时: " + (endTime - startTime) + " 毫秒");

        // 等待子线程结束，以便观察清楚
        try {
            workerThread.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("\n--- 调用 run() ---");
        startTime = System.currentTimeMillis();
        workerThread.run(); // 同步，主线程在此阻塞2秒
        endTime = System.currentTimeMillis();
        System.out.println("主线程调用run()耗时: " + (endTime - startTime) + " 毫秒");

        System.out.println("\n主线程结束。");
    }

}
典型输出：

text
主线程开始。

--- 调用 start() ---
主线程调用start()耗时: 0 毫秒
子线程开始工作，线程ID: 12
子线程工作完成。

--- 调用 run() ---
子线程开始工作，线程ID: 1 (注意：这是主线程ID)
子线程工作完成。
主线程调用run()耗时: 2002 毫秒

主线程结束。
关键观察点：

start() 调用耗时几乎为0，且子线程的输出与主线程后续输出交错出现（并发）。

run() 调用耗时约2秒（被sleep阻塞），且输出的线程ID是主线程的ID，说明是主线程自己在执行任务。

start() 方法的关键细节
查看 Thread.start() 源码，你会发现：

它是 synchronized 方法，防止一个线程被多次启动。

它会检查线程状态 threadStatus，若非0（即NEW状态以外的状态），则抛出 IllegalThreadStateException。

核心是一个名为 start0() 的私有本地方法（native），该方法负责与操作系统交互，创建真正的执行线程。

总结
简单来说，start() 是 “点火发射” 指令，开启一个新的并行执行流。而 run() 只是 “任务蓝图”
本身，直接调用它就像在原地按照蓝图施工，不会产生新的“施工队”（线程）。在需要实现并发的地方，务必使用 start() 来启动线程。