---
title: Java8 新特性之方法引用
date: 2025-10-15 11:36:33
category: 后端
tags: 新特性
---

方法引用（Method References）是 Java 8 中一种更简洁的 Lambda 表达式写法。当 Lambda
表达式仅仅是调用一个已有方法时，使用方法引用可以让代码更加简洁、清晰，提高代码的可读性。

方法引用的本质
方法引用不是方法调用，而是对现有方法的引用。它通过 :: 操作符将方法名与类或对象分隔开来，创建了一个函数式接口的实例。

java
// Lambda 表达式
Function<String, Integer> lambda = s -> Integer.parseInt(s);

// 方法引用（更简洁）
Function<String, Integer> methodRef = Integer::parseInt;

// 两者功能完全相同
System.out.println(lambda.apply("123")); // 123
System.out.println(methodRef.apply("123")); // 123
四种方法引用形式

1. 静态方法引用：ClassName::staticMethod
   java
   public class StaticMethodReference {
   public static void main(String[] args) {
   // 1.1 基本使用：将静态方法作为函数式接口的实现
   Function<String, Integer> parser = Integer::parseInt;
   System.out.println("解析整数: " + parser.apply("42"));

        // 1.2 多参数静态方法
        BiFunction<Integer, Integer, Integer> max = Math::max;
        System.out.println("最大值: " + max.apply(10, 20));
        
        // 1.3 自定义静态方法引用
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
        names.forEach(StaticMethodReference::printWithPrefix);
        
        // 1.4 在 Stream 中使用
        List<String> numbers = Arrays.asList("1", "2", "3", "4", "5");
        List<Integer> ints = numbers.stream()
                .map(Integer::parseInt)  // 静态方法引用
                .collect(Collectors.toList());
        System.out.println("转换后的整数: " + ints);
        
        // 1.5 复杂静态方法引用
        Function<Double, Double> sqrt = Math::sqrt;
        Function<Double, Double> log = Math::log;
        Function<Double, Double> exp = Math::exp;
        
        // 组合函数
        Function<Double, Double> complex = sqrt.andThen(log).andThen(exp);