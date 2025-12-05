---
title: Java 中的父类向子类强制转型原则
date: 2025-10-14 14:42:34
category: 后端
tags: 基础
---


在 Java 类型系统中，子类向父类的转换（向上转型）是安全的，因为子类继承了父类的所有特性。然而，父类向子类的强制转换（向下转型）则可能导致
ClassCastException。

示例分析
java
class Animal {}
class Dog extends Animal {}

public class Test {
public static void main(String[] args) {
// 情况一：直接创建父类实例，无法转为子类
Animal animal = new Animal();
Dog dog = (Dog) animal; // 抛出 ClassCastException

        // 情况二：父类引用指向子类实例，可以安全转换
        Animal animal2 = new Dog();
        Dog dog2 = (Dog) animal2; // 转换成功
    }

}
核心原则
只有父类引用实际指向的是该子类（或其子类）的实例时，才能成功转换为该子类。
换句话说，向下转型前应确保对象的真实类型与目标类型匹配。

安全转型建议
使用 instanceof 进行类型检查：

java
if (animal instanceof Dog) {
Dog dog = (Dog) animal;
}
尽量通过设计避免频繁向下转型，善用多态。

