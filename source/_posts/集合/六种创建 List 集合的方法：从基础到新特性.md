---
title: 六种创建 List 集合的方法：从基础到新特性
date: 2025-10-15 11:36:33
category: 后端
tags: 集合
---

在 Java 开发中，List 是最常用的集合接口之一。初始化一个 List 有多种方法，其中一些方法存在易被忽略的“陷阱”。本文将为你系统介绍六种常见的初始化方式。

方法一：常规新增法
这是最基础、最直观的方式。

```java
List<String> list = new ArrayList<>(); // JDK7+ 可省略泛型类型
list.add("A");
list.add("B");
list.add("C");
System.out.println(list); // 输出：[A, B, C]
```
特点：简单灵活，适用于任何场景。

方法二：使用 Arrays.asList(T... a)
Arrays 工具类提供了快速从数组创建列表的方法。

```java
List<String> list = Arrays.asList("Java", "Python", "C++");
System.out.println(list); // 输出：[Java, Python, C++]
⚠️ 重要陷阱：该方法返回的 List 是 Arrays 的一个内部类，大小固定。不支持 add()、remove() 等结构性修改操作，否则会抛出
UnsupportedOperationException。
```
可变方案：如果需要可变的列表，可以将其作为参数传入一个新的 ArrayList 构造器。

```java
List<String> mutableList = new ArrayList<>(Arrays.asList("1", "2", "3"));
mutableList.add("4"); // 现在可以了
```
方法三：使用 Collections 工具类
Collections 类提供了几个特殊的静态工厂方法。

Collections.nCopies(int n, T o)：创建一个包含 n 个相同对象 o 的不可变列表。

```java
List<String> copies = Collections.nCopies(3, "item");
System.out.println(copies); // 输出：[item, item, item]
Collections.singletonList(T o)：创建一个只包含一个指定对象的不可变列表。比 Arrays.asList(o) 更节约内存，语义更清晰。
```


```java
List<String> single = Collections.singletonList("alone");
Collections.emptyList()：返回一个不可变的空列表。常用于方法返回值，避免返回 null。
```

```java
List<String> empty = Collections.emptyList();
```
注意：以上方法返回的列表都是不可变的。如需可变，同样需要用 new ArrayList<>(...) 包装。

方法四：匿名内部类 + 实例初始化块
一种利用 Java 语法特性的“炫技”写法。

```java
List<String> list = new ArrayList<String>() {{
add("Tom");
add("Jerry");
}};
System.out.println(list); // 输出：[Tom, Jerry]
```
原理：外层花括号创建了一个 ArrayList 的匿名子类，内层花括号是一个实例初始化块，在构造时执行。这种方法虽然简洁，但会创建额外的类，且可读性对部分开发者不友好，生产代码中慎用。

方法五：Java 8 Stream API
利用 Stream 的流畅式编程风格。

```java
List<String> list = Stream.of("Apple", "Banana", "Orange")
.collect(Collectors.toList());
System.out.println(list);
```
特点：函数式风格，易于进行链式操作（如过滤、映射等）。生成的 ArrayList 是可变的。

方法六：Java 9+ 的 List.of() 工厂方法
Java 9 在 List 接口中引入了静态工厂方法 of。

```java
List<String> list = List.of("A", "B", "C");
System.out.println(list); // 输出：[A, B, C]
```
⚠️ 重要特性：该方法返回的是一个高度优化、不可变的列表。不支持任何修改操作，且对 null 元素零容忍（传入 null 会抛出
NullPointerException）。它是创建小型常量列表的现代推荐方式。

总结与选择建议
日常可变列表：首选方法一（常规新增） 或方法二（包装 Arrays.asList）。

小型不可变常量列表：Java 9+ 环境首选方法六（List.of()）；低版本可使用方法二（Arrays.asList）
或方法三（Collections.singletonList/nCopies）。

函数式处理：考虑方法五（Stream API）。

避免使用：方法四（匿名内部类） 在大多数生产场景下并非好选择，方法二（直接使用 Arrays.asList 的结果进行修改） 是常见错误根源。

Map 和 Set 也有类似的初始化方法，原理相通。