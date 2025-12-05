---
title: 三种获取Java类名的方法对比与适用场景
date: 2025-10-15 11:36:33
category: 后端
tags: 进阶
---

Java中获取类名的主要方式有三种，分别返回不同格式的类名信息：

getName()：返回JVM内部使用的类名表示形式。

getCanonicalName()：返回更易理解的规范类名。

getSimpleName()：返回不包含包名的简称。

通过以下示例可清晰辨别三者差异：

text
public class TestClass {
public static void main(String[] args) {
// 外部普通类
System.out.println("方法名 类名");
System.out.println("getName            " + TestClass.class.getName());
System.out.println("getCanonicalName   " + TestClass.class.getCanonicalName());
System.out.println("getSimpleName      " + TestClass.class.getSimpleName());
System.out.println();

		// 内部类
		System.out.println("getName            " + TestInnerClass.class.getName());
		System.out.println("getCanonicalName   " + TestInnerClass.class.getCanonicalName());
		System.out.println("getSimpleName      " + TestInnerClass.class.getSimpleName());
		System.out.println();

		// 数组类
		TestInnerClass[] array = new TestInnerClass[3];
		System.out.println("getName            " + array.getClass().getName());
		System.out.println("getCanonicalName   " + array.getClass().getCanonicalName());
		System.out.println("getSimpleName      " + array.getClass().getSimpleName());
	}
	static class TestInnerClass {}

}
输出结果：

text
方法名 类名
getName com.test.TestClass
getCanonicalName com.test.TestClass
getSimpleName TestClass

getName com.test.TestClass$TestInnerClass
getCanonicalName com.test.TestClass.TestInnerClass
getSimpleName TestInnerClass

getName            [Lcom.test.TestClass$TestInnerClass;
getCanonicalName com.test.TestClass.TestInnerClass[]
getSimpleName TestInnerClass[]
其中数组类的getName()返回值中的 [L 和 ; 是JNI字段描述符的约定，[ 表示数组，L 表示类描述符开始。

结论
对于普通类，getName()与getCanonicalName()结果相同。

对于内部类，getName()使用$分隔，而getCanonicalName()使用.分隔。

对于数组类，getSimpleName()会保留[]后缀，而getCanonicalName()会以更可读的形式展示维度信息。