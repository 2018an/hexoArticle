---
title: 深入理解Java类初始化顺序的三个关键示例
date: 2025-10-15 11:36:33
category: 后端
tags: 进阶
---

类初始化顺序是Java基础中容易混淆的知识点，涉及静态成分、实例成分及继承关系。下面通过三个示例系统梳理其执行顺序。

示例一：单一类的初始化顺序
text
public class ClassInitOrderTest {
public static String staticField = "static field";
static {
System.out.println(staticField);
System.out.println("static block");
}
private String field = "member field";
{
System.out.println(field);
System.out.println("non-static block");
}
public ClassInitOrderTest() {
System.out.println("constructor");
}
public static void main(String[] args) {
new ClassInitOrderTest();
}
}
输出顺序为：

text
static field
static block
member field
non-static block
constructor
结论：静态变量 → 静态代码块 → 成员变量 → 非静态代码块 → 构造器

示例二：继承体系中的初始化顺序
text
class Parent {
private static String parentStaticField = "parent static field";
static { System.out.println(parentStaticField); }
private String parentField = "parent member field";
{ System.out.println(parentField); }
public Parent() { System.out.println("parent constructor"); }
}
public class Child extends Parent {
private static String childStaticField = "child static field";
static { System.out.println(childStaticField); }
private String childField = "child member field";
{ System.out.println(childField); }
public Child() { System.out.println("child constructor"); }
public static void main(String[] args) { new Child(); }
}
输出顺序为：

text
parent static field
parent static block
child static field
child static block
parent member field
parent non-static block
parent constructor
child member field
child non-static block
child constructor
结论：父类静态 → 子类静态 → 父类实例 → 父类构造 → 子类实例 → 子类构造

示例三：同一类中静态成分的顺序性
text
public class TestOrder {
private static A a = new A();
static { System.out.println("static block"); }
private static B b = new B();
public static void main(String[] args) { new TestOrder(); }
}
class A { public A() { System.out.println("static field A"); } }
class B { public B() { System.out.println("static field B"); } }
输出顺序为：

text
static field A
static block
static field B
结论：同一类中静态变量与静态代码块的初始化顺序取决于它们在源码中的书写顺序，而非变量必然先于代码块。