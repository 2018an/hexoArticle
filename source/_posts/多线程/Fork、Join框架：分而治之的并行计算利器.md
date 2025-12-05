---
title: Fork/Join框架：分而治之的并行计算利器
date: 2025-10-14 14:42:34
category: 后端
tags: 多线程
---


面对大规模计算任务，单线程顺序处理往往力不从心。Java 7引入的Fork/Join框架，正是为了简化此类问题的并行化处理而生。它将“分而治之”（Divide
and Conquer）的算法思想与线程池相结合，自动将大任务递归分解为小任务并行执行，最后合并结果，特别适合处理可递归分解的计算密集型任务。

核心概念与原理

1. 核心思想
   Fork（分叉）：将一个大任务递归地分割（fork）成若干个互不依赖的子任务。

Join（合并）：等待所有子任务执行完毕后，将它们的结果合并（join）起来，得到最终结果。

2. 工作窃取算法
   这是Fork/Join框架高效的关键。每个工作线程维护一个双端队列来存放分配给它的任务。

线程正常从自己队列的队头取出任务执行。

当某个线程自己的队列为空时，它会随机从其他线程队列的队尾“窃取”一个任务来执行。
优势：减少了线程因等待任务而产生的空闲时间，充分利用了CPU资源，实现了负载均衡。

https://res.infoq.com/articles/fork-join-introduction/zh/resources/image3.png

核心组件
ForkJoinPool
“调度器”，是特殊的线程池，用于执行ForkJoinTask。它除了具备普通线程池的功能，还内置了工作窃取调度逻辑。

常用方法：execute()（异步执行）、invoke()（同步执行并等待结果）、submit()（提交返回Future）。

ForkJoinTask
抽象任务类，有两个常用子类：

RecursiveAction：用于没有返回值的任务（例如并行排序、数组填充）。

RecursiveTask<V>：用于有返回值的任务（例如并行求和、查找）。

实战：使用Fork/Join计算大型数列之和
我们来对比传统串行求和与Fork/Join并行求和的性能差异。任务：计算从1到10亿（1,000,000,000）的所有整数之和。

java
import java.util.concurrent.*;

public class ForkJoinSumCalculator extends RecursiveTask<Long> {
// 任务处理的数列范围 [start, end]
private final long start;
private final long end;
// 分割阈值：当任务大小小于此值时，不再分割，直接计算
private static final long THRESHOLD = 10_000L;

    public ForkJoinSumCalculator(long start, long end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        long length = end - start + 1;
        // 如果任务足够小，直接计算（基本情况）
        if (length <= THRESHOLD) {
            long sum = 0;
            for (long i = start; i <= end; i++) {
                sum += i;
            }
            return sum;
        } else { // 任务太大，继续分割（递归情况）
            long middle = (start + end) / 2;
            // 创建左半部分子任务
            ForkJoinSumCalculator leftTask = new ForkJoinSumCalculator(start, middle);
            leftTask.fork(); // 异步执行，将其压入当前线程的队列
            // 创建右半部分子任务
            ForkJoinSumCalculator rightTask = new ForkJoinSumCalculator(middle + 1, end);
            Long rightResult = rightTask.compute(); // 同步执行右半部分（当前线程）
            Long leftResult = leftTask.join(); // 等待左半部分子任务完成，获取结果
            // 合并结果
            return leftResult + rightResult;
        }
    }

    public static void main(String[] args) {
        System.out.println("--- 串行计算 ---");
        long startTime = System.currentTimeMillis();
        long serialSum = sequentialSum(1, 1_000_000_000L);
        long endTime = System.currentTimeMillis();
        System.out.println("结果: " + serialSum);
        System.out.println("耗时: " + (endTime - startTime) + " ms");

        System.out.println("\n--- Fork/Join 并行计算 ---");
        startTime = System.currentTimeMillis();
        ForkJoinPool pool = new ForkJoinPool(); // 使用公共ForkJoinPool
        ForkJoinSumCalculator task = new ForkJoinSumCalculator(1, 1_000_000_000L);
        long parallelSum = pool.invoke(task); // 提交任务并等待结果
        endTime = System.currentTimeMillis();
        System.out.println("结果: " + parallelSum);
        System.out.println("耗时: " + (endTime - startTime) + " ms");
        pool.shutdown();
    }

    // 传统的串行求和
    private static long sequentialSum(long start, long end) {
        long sum = 0;
        for (long i = start; i <= end; i++) {
            sum += i;
        }
        return sum;
    }

}
运行结果与分析
在多数多核机器上运行，可能得到类似输出：

text
--- 串行计算 ---
结果: 500000000500000000
耗时: 3200 ms

--- Fork/Join 并行计算 ---
结果: 500000000500000000
耗时: 450 ms
并行版本取得了显著的加速（约7倍），这得益于Fork/Join框架自动将任务分解到多个CPU核心上并行执行。

适用场景与注意事项
适用：计算密集型、可递归分解的任务，如图像处理、大规模数据分析、模拟等。
注意：

任务开销：任务分割和结果合并本身有开销。如果任务本身很轻量（例如只做几次加法），使用Fork/Join可能得不偿失。阈值（THRESHOLD）的设置至关重要。

避免阻塞：子任务应避免进行I/O操作或同步阻塞，否则会拖慢整个池。

递归深度：过深的递归可能导致栈溢出或产生海量子任务，消耗大量内存。

结果依赖性：子任务之间应尽量独立，避免共享可变状态，否则仍需额外的同步。