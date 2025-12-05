---
title: Thread.join()：线程顺序执行的协调者
date: 2025-10-14 09:53:44
category: 后端
tags: 多线程
---


在多线程编程中，我们经常遇到这样的需求：一个线程（通常是主线程）需要等待另一个或多个线程完成工作后，才能继续执行。Thread.join()
方法正是为满足这种“等待-继续”模式而设计的同步工具。

方法语义
join() 方法使当前线程进入等待状态，直到被调用join()方法的那个线程终止（即执行完 run() 方法）。它有多个重载版本：

join(): 无限期等待，直到目标线程结束。

join(long millis): 最多等待 millis 毫秒。

join(long millis, int nanos): 最多等待指定毫秒+纳秒。

基础用法示例
java
public class JoinDemo {
public static void main(String[] args) throws InterruptedException {
System.out.println("【主线程】启动一个工作线程。");

        Thread worker = new Thread(() -> {
            System.out.println("【工作线程】开始执行任务。");
            try {
                // 模拟耗时任务
                for (int i = 1; i <= 3; i++) {
                    System.out.println("【工作线程】正在处理步骤 " + i + "...");
                    Thread.sleep(1000);
                }
            } catch (InterruptedException e) {
                System.out.println("【工作线程】被中断。");
                Thread.currentThread().interrupt();
            }
            System.out.println("【工作线程】任务执行完毕。");
        });

        worker.start(); // 启动工作线程

        System.out.println("【主线程】调用 worker.join()，等待工作线程结束。");
        long start = System.currentTimeMillis();
        worker.join(); // 主线程在此阻塞，等待worker线程结束
        long end = System.currentTimeMillis();

        System.out.println("【主线程】工作线程已结束，等待耗时: " + (end - start) + "ms");
        System.out.println("【主线程】继续执行后续逻辑。");
    }

}
输出：

text
【主线程】启动一个工作线程。
【主线程】调用 worker.join()，等待工作线程结束。
【工作线程】开始执行任务。
【工作线程】正在处理步骤 1...
【工作线程】正在处理步骤 2...
【工作线程】正在处理步骤 3...
【工作线程】任务执行完毕。
【主线程】工作线程已结束，等待耗时: 3012ms
【主线程】继续执行后续逻辑。
可以看到，主线程在 worker.join() 处被阻塞了约3秒（工作线程执行时间），直到工作线程结束后才继续。