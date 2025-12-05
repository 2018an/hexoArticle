---
title: Java 8 实战：十种姿势轻松生成 Stream 数据流
date: 2025-10-15 11:36:33
category: 后端
tags: 新特性
---

https://img/20190613135450.png
https://img/20190613135537.png

Stream 是 Java 8 函数式编程的核心之一，而构建 Stream 则是所有流操作的第一步。本文为你梳理十种常见且实用的 Stream
创建方法，助你从容应对各种数据源。

方法 1 & 2：基于直接值的快速创建 —— Stream.of
Stream.of 方法是最直观的创建方式，它接受可变参数或单个数组。

```java
// 1. 可变参数
Stream<String> streamFromVarargs = Stream.of("Java", "Python", "Go");
System.out.println(streamFromVarargs.collect(Collectors.joining(", ")));
// 输出：Java, Python, Go

// 2. 数组
String[] langArray = {"C++", "Rust", "JavaScript"};
Stream<String> streamFromArray = Stream.of(langArray);
System.out.println(streamFromArray.collect(Collectors.joining("-")));
// 输出：C++-Rust-JavaScript
```
查看源码可知，Stream.of(T... values) 内部其实是调用了 Arrays.stream(values)。

方法 3：数组专用通道 —— Arrays.stream
对于数组类型，直接使用 Arrays.stream() 是标准做法，它还可以指定起始和结束索引。

```java
Integer[] numbers = {10, 20, 30, 40, 50};
Stream<Integer> numStream = Arrays.stream(numbers, 1, 4); // 截取索引[1,4)的元素
numStream.forEach(n -> System.out.print(n + " "));
```
// 输出：20 30 40
方法 4 & 5 & 6：集合框架的流化 —— collection.stream()
所有 Collection 的实现类（List, Set）及其派生视图（如 Map 的 values）都支持直接获取流。

```java
// 4. List 转流
List<String> list = Arrays.asList("Apple", "Banana");
list.stream().forEach(System.out::println);

// 5. Set 转流
Set<Integer> set = new HashSet<>(Arrays.asList(1, 2, 2, 3));
set.stream().forEach(System.out::println); // 输出：1 2 3 (去重)

// 6. Map 转流
Map<Integer, String> map = Map.of(1, "One", 2, "Two");
// 获取键、值或条目的流
map.keySet().stream().forEach(System.out::println);
map.values().stream().forEach(System.out::println);
map.entrySet().stream().forEach(e -> System.out.println(e.getKey() + ":" + e.getValue()));
```
方法 7：生成有序无限流 —— Stream.iterate
iterate 适合生成有规律（尤其是递推关系）的数据序列，记得用 limit 截断，否则是无限流。

```java
// 生成前5个偶数：0, 2, 4, 6, 8
Stream<Integer> evenNumbers = Stream.iterate(0, n -> n + 2).limit(5);
evenNumbers.forEach(n -> System.out.print(n + " "));
```
方法 8：正则切分字符串为流 —— Pattern.splitAsStream
处理文本数据时，可以利用正则表达式将字符串直接拆分为流。

```java
String sentence = "Welcome to the stream world";
Pattern pattern = Pattern.compile("\\s+"); // 按空白字符分割
Stream<String> wordStream = pattern.splitAsStream(sentence);
wordStream.forEach(System.out::println);
// 依次输出：Welcome / to / the / stream / world
```
方法 9：逐行读取文件为流 —— Files.lines
这是处理文本文件的大杀器，可自动关闭资源（配合 try-with-resources），并充分利用流的惰性求值特性。

```java
Path filePath = Paths.get("data.log");
try (Stream<String> lines = Files.lines(filePath, StandardCharsets.UTF_8)) {
long errorCount = lines.filter(line -> line.contains("ERROR")).count();
System.out.println("错误日志行数：" + errorCount);
} catch (IOException e) {
e.printStackTrace();
}
```
方法 10：生成常量或随机无限流 —— Stream.generate
generate 接受一个 Supplier，不断调用它来生成流元素。务必配合 limit 使用。

```java
// 生成10个随机数
Stream<Double> randomStream = Stream.generate(Math::random).limit(10);
randomStream.forEach(System.out::println);

// 生成5个相同字符串
Stream<String> echoStream = Stream.generate(() -> "Echo").limit(5);
echoStream.forEach(System.out::println);
```
总结与选择建议
场景 推荐方法
已知少量元素 Stream.of(...)
数组 Arrays.stream()
任何 Collection (List, Set, Queue)    collection.stream()
Map 的键、值或条目 map.keySet().stream() 等
按规律生成数字/序列 Stream.iterate(seed, f).limit(n)
文件逐行处理 Files.lines(path)
字符串按模式分割 Pattern.compile(regex).splitAsStream(str)
生成恒定值或随机数 Stream.generate(supplier).limit(n)
掌握这十种创建方式，你就能轻松地将各种形态的数据源——无论是内存中的集合、外部的文件，还是动态生成的序列——接入 Stream
的强大处理管道中，为后续的过滤、映射、归约等操作铺平道路。