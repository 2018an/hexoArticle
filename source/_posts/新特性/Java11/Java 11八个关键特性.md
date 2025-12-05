---
title: Java 11八个关键特性
date: 2025-10-15 11:36:33
category: 后端
tags: 新特性
---

http://qianniu.javastack.cn/18-9-26/91229065.jpg

2018年9月25日，Oracle 正式推出了 Java 11。这是自 Java 8 之后的首个长期支持（LTS）版本，其支持周期将延续至2026年。

目前大多数企业仍在使用 Java 8，而 Java 9 和 10 由于非长期支持且变动较大，在生产环境中较为少见。Java 11
作为长期支持版本，包含了之前版本的所有功能，并带来了许多重要更新，下面将介绍其中八个关键特性。

1. 局部变量类型推断（var 关键字）
   Java 10 引入的 var 关键字在 Java 11 中得以延续，允许在局部变量声明时省略显式类型，由编译器自动推断。

```text
var message = "Hello Java 11";
System.out.println(message);
```
这等价于：

```text
String message = "Hello Java 11";
```
var 让代码更简洁，尤其适用于复杂类型或泛型实例的声明。但需注意，它仅适用于局部变量，不可用于成员变量、方法参数或返回类型。

2. 字符串处理增强
   Java 11 为 String 类新增了多个实用方法：

```text
// 判断字符串是否空白
"   ".isBlank(); // true

// 去除首尾空白字符
"  Example  ".strip(); // "Example"

// 去除尾部空白
"  Example  ".stripTrailing(); // "  Example"

// 去除首部空白
"  Example  ".stripLeading(); // "Example  "

// 重复字符串
"Ab".repeat(3); // "AbAbAb"

// 转换为行流并计数
"A\nB\nC".lines().count(); // 3
```

3. 集合工厂方法强化
   从 Java 9 开始，集合框架引入了 of 和 copyOf 静态方法，用于创建不可变集合。

```text
var list = List.of("Java", "Python", "Go");
var copy = List.copyOf(list);
System.out.println(list == copy); // true，因为源已为不可变集合
var list = new ArrayList<String>();
var copy = List.copyOf(list);
System.out.println(list == copy); // false，因为源为可变集合，copyOf 会创建新实例
```
注意：通过这些方法创建的集合均为不可变，尝试修改会抛出 UnsupportedOperationException。

4. Stream API 扩展
   Java 9 对 Stream 新增了多个方法：

```text
// 允许单个元素为 null
Stream.ofNullable(null).count(); // 0

// takeWhile：遇到第一个不符合条件的元素即停止
Stream.of(1, 2, 3, 2, 1)
.takeWhile(n -> n < 3)
.collect(Collectors.toList()); // [1, 2]

// dropWhile：丢弃符合条件的初始元素，直到遇到第一个不符合条件的元素
Stream.of(1, 2, 3, 2, 1)
.dropWhile(n -> n < 3)
.collect(Collectors.toList()); // [3, 2, 1]
```

5. Optional 类功能增强
   Optional 新增了 orElseThrow、stream 和 or 等方法，使其更灵活。

```text
Optional.of("JavaStack").orElseThrow(); // 返回 "JavaStack"，若为空则抛出异常
Optional.of("JavaStack").stream().count(); // 1，转换为 Stream
Optional.ofNullable(null)
.or(() -> Optional.of("Default"))
.get(); // 返回 "Default"
```

6. InputStream 新增 transferTo 方法
   InputStream 新增 transferTo 方法，便于将数据直接传输到 OutputStream。

```text
var loader = ClassLoader.getSystemClassLoader();
var input = loader.getResourceAsStream("data.txt");
var tempFile = File.createTempFile("temp", ".txt");
try (var output = new FileOutputStream(tempFile)) {
input.transferTo(output);
}
```

7. 标准化的 HTTP Client API
   Java 11 将孵化已久的 HTTP Client API 正式纳入标准库，支持同步和异步请求。

```text
var request = HttpRequest.newBuilder()
.uri(URI.create("https://example.com"))
.build();
var client = HttpClient.newHttpClient();

// 同步请求
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println(response.body());

// 异步请求
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
.thenApply(HttpResponse::body)
.thenAccept(System.out::println);
```

8. 单命令编译运行源代码
   Java 11 允许直接使用 java 命令运行单个源代码文件，无需先显式编译。

```text
// 传统方式
javac Main.java
java Main

// Java 11 新方式
java Main.java
```
其他值得关注的新特性
响应式编程 Flow API

模块化系统（Java Platform Module System）

应用程序类数据共享

动态类文件常量

JShell（交互式编程环境）

飞行记录器（Flight Recorder）

Unicode 10 支持

垃圾回收器改进：G1 并行 Full GC、ZGC、Epsilon GC

弃用 Nashorn JavaScript 引擎

结语
虽然目前许多项目仍在使用 Java 8，但 Java 11
作为长期支持版本，无疑是升级的重要选择。其引入的新特性不仅提升了开发效率，也增强了语言的表现力。及时了解并逐步应用这些新功能，将帮助我们在未来的开发中保持技术竞争力。