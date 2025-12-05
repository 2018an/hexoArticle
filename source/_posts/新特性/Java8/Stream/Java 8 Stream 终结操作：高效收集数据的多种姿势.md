---
title: Java 8 Stream 终结操作：高效收集数据的多种姿势
date: 2025-10-15 11:36:33
category: 后端
tags: 新特性
---

在之前的学习中，我们大多是把集合转为流，进行各种中间操作。但数据处理完毕后，我们往往需要将结果重新“收回”到一个数据结构中，比如
List、Set 或 Map。Stream API 提供的 collect 方法，正是完成这一“收集”工作的关键终端操作。

一、collect 方法的两副面孔
Stream 接口中定义了两个 collect 方法：

```java
// 方法1：使用 Collector 收集器
<R, A> R collect(Collector<? super T, A, R> collector);

// 方法2：使用三个函数参数自定义收集过程
<R> R collect(Supplier<R> supplier,
BiConsumer<R, ? super T> accumulator,
BiConsumer<R, R> combiner);
```
第一种方法借助 Collector 接口的实现类来定义收集规则，这也是我们最常用的方式。第二种方法则更为底层，允许我们通过三个函数式参数直接控制收集的每一步。

二、使用 Collectors 工具类：开箱即用的收集策略
Java 8 在 java.util.stream.Collectors 类中提供了大量静态工厂方法，用于创建常用的收集器。我们无需自己实现 Collector
接口，直接调用它们即可。

2.1 归集到 List 与 Set
```java
// 收集到 List
List<T> list = stream.collect(Collectors.toList());

// 收集到 Set
Set<T> set = stream.collect(Collectors.toSet());
```
这两个方法会自动处理重复元素和顺序问题。

2.2 实战示例：从游戏数据中收集信息
假设我们有一组游戏对战数据，包含英雄、玩家和实时金币数：

```java
// 数据类
class BattleRecord {
private String hero;
private String player;
private int gold;
// 构造方法、getter、toString 省略
}
```
我们希望完成两个收集任务：

提取所有玩家的金币信息，存入 List。

提取所有不重复的英雄，存入 Set。

```java
public class StreamCollectDemo {
public static void main(String[] args) {
List<BattleRecord> records = Arrays.asList(
new BattleRecord("盖伦", "RNG-Letme", 100),
new BattleRecord("诸葛亮", "RNG-Xiaohu", 300),
new BattleRecord("露娜", "RNG-MLXG", 300),
new BattleRecord("狄仁杰", "RNG-UZI", 500),
new BattleRecord("牛头", "RNG-Ming", 500)
);

        // 收集玩家金币列表
        List<PlayerGold> goldList = records.stream()
            .map(r -> new PlayerGold(r.getPlayer(), r.getGold()))
            .collect(Collectors.toList());

        System.out.println("玩家金币列表：");
        goldList.forEach(System.out::println);

        // 收集不重复的英雄集合
        Set<String> heroSet = records.stream()
            .map(BattleRecord::getHero)
            .collect(Collectors.toSet());

        System.out.println("\n出场英雄集合：");
        heroSet.forEach(System.out::println);
    }

}
```
运行结果如下：

```text
玩家金币列表：
PlayerGold{player='RNG-Letme', gold=100}
PlayerGold{player='RNG-Xiaohu', gold=300}
PlayerGold{player='RNG-MLXG', gold=300}
PlayerGold{player='RNG-UZI', gold=500}
PlayerGold{player='RNG-Ming', gold=500}
```

出场英雄集合：
牛头
诸葛亮
露娜
狄仁杰
盖伦
提示：结合 Collectors 与 JSON 序列化库（如 FastJson、Gson），可以轻松实现集合数据到 JSON 数组的转换，这在 Web 开发中极为实用。

三、三参数 collect：自定义收集过程
当你需要更精细地控制收集行为，或者目标容器比较特殊时，可以使用三参数的 collect 方法。

supplier：提供一个结果容器（如 ArrayList::new）。

accumulator：定义如何将元素添加到容器中（如 List::add）。

combiner：在并行流中，定义如何合并两个部分结果（如 List::addAll）。

3.1 演进示例：从匿名类到方法引用
我们尝试将 BattleRecord 流收集到一个 HashSet 中。

原始写法（匿名内部类）：

```java
HashSet<BattleRecord> set = records.stream().collect(
new Supplier<HashSet<BattleRecord>>() {
public HashSet<BattleRecord> get() { return new HashSet<>(); }
},
new BiConsumer<HashSet<BattleRecord>, BattleRecord>() {
public void accept(HashSet<BattleRecord> set, BattleRecord record) {
set.add(record);
}
},
new BiConsumer<HashSet<BattleRecord>, HashSet<BattleRecord>>() {
public void accept(HashSet<BattleRecord> set1, HashSet<BattleRecord> set2) {
set1.addAll(set2);
}
}
);
```
Lambda 简化版：

```java
HashSet<BattleRecord> set = records.stream().collect(
() -> new HashSet<>(),
(s, r) -> s.add(r),
(s1, s2) -> s1.addAll(s2)
);
```
方法引用终极版：

```java
HashSet<BattleRecord> set = records.stream().collect(
HashSet::new,
HashSet::add,
HashSet::addAll
);
```
可以看到，通过方法引用，代码变得异常简洁清晰。

四、核心要点总结
优先使用 Collectors：该类提供了 toList、toSet、toMap、groupingBy、joining 等丰富方法，能满足绝大多数收集需求。

理解三参数 collect：当遇到非常规收集场景时，可以使用 supplier、accumulator、combiner 这三个参数来自定义逻辑。这在处理并发流或自定义聚合时非常有用。

保持操作特性：与 reduce 操作一样，传递给 collect 的函数必须满足无状态、不干预和关联性的要求，尤其是在并行流中。

灵活组合：收集操作常与 map、filter、sorted 等中间操作联用，构建出强大且高效的数据处理管道。

掌握收集操作，意味着你能够将 Stream 流水线的最终成果，优雅地封装回你需要的任何数据结构中，完成数据处理的闭环。