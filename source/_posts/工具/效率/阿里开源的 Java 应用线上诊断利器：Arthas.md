---
title: 阿里开源的 Java 应用线上诊断利器：Arthas
date: 2025-10-30 14:42:34
category: 工具
tags: 效率
---

什么是 Arthas？
Arthas 是阿里巴巴开源的一款面向 Java 应用的在线诊断与调试工具。它以其强大的问题排查能力而闻名，能够在无需重启服务、无需修改代码的情况下，对运行中的
Java 进程进行深度探测，解决许多令开发者头疼的线上疑难杂症。

Arthas 采用命令行交互模式，支持 JDK 6+ 及主流操作系统（Linux/macOS/Windows），并提供了友好的命令自动补全功能，使得诊断过程高效顺畅。

官方网站：https://alibaba.github.io/arthas/

GitHub 仓库：https://github.com/alibaba/arthas

它能解决哪些典型问题？
根据官方描述，当您遇到以下场景时，Arthas 或许能提供关键帮助：

无法确定某个类是从哪个 JAR 包加载的，或遭遇各类与类加载相关的异常。

确认代码已修改但未生效，需要验证是分支问题、部署问题还是其他原因。

线上环境无法进行常规 Debug，又不想仅为了添加日志而重新发布应用。

需要追踪线上特定用户请求的数据处理路径，但线下环境难以复现。

期望有一个全局视角来实时观察应用的整体运行健康状况。

需要监控 JVM 内部的实时运行状态，如内存、线程、GC 等。

其核心亮点在于支持在线反编译类、动态方法调用跟踪等功能，实现了“线上 Debug”的梦想。

快速安装与启动
推荐使用 arthas-boot 这个启动器，过程非常简洁（以下以 Linux 环境为例）：

下载启动器：

bash
wget https://alibaba.github.io/arthas/arthas-boot.jar
启动并选择目标进程：

bash
java -jar arthas-boot.jar
执行后，控制台会列出当前所有 Java 进程。输入对应序号并回车，Arthas 便会自动下载必要组件并附加到目标进程上。

连接成功：出现 Arthas 的 Logo 及版本信息，即表示已成功连接，可以开始输入诊断命令。

常用命令实战演示
连接到目标进程后，便进入了 Arthas 控制台。以下是一些最常用命令的示例：

dashboard：查看当前系统的实时数据面板，包含线程、内存、GC、运行时信息等。
https://img/20190730154952.png

thread：查看线程信息。thread id 可查看指定线程栈；thread -n 3 可显示最繁忙的 3 个线程。
https://img/20190730163759.png

jad：反编译指定类的源代码。例如：jad com.example.DemoService。
https://img/20190730155942.png

watch：观测方法执行的入参、返回值或异常。例如：watch com.example.DemoService queryUser '{params, returnObj}' -x 2。
https://img/20190730171659.png

trace：追踪方法内部调用链路，并统计每个节点的耗时，用于性能分析。
https://img/20190730165157.png

monitor：对方法调用进行定时监控，统计调用次数、成功率、平均耗时等指标。
https://img/20190730170442.png

sc / sm：分别用于查看 JVM 已加载的类详情和类中的方法信息。

总结
Arthas 为 Java 开发者提供了一套极其强大的线上问题排查工具箱。从查看 JVM
状态、反编译代码到监控方法执行，它都能在不侵入业务代码的前提下提供关键信息。尽管在分布式系统跟踪方面存在局限，但针对单 JVM
进程的深度诊断，它无疑是目前最优秀的工具之一。掌握 Arthas，能让线上问题定位的效率得到质的提升。