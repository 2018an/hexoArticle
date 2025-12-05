---
title: String不可变性的本质与极限突破
date: 2025-10-15 11:36:33
category: 后端
tags: 进阶
---

String类被设计为不可变，其源码关键字段如下：

text
public final class String {
private final char value[];
private int hash;
}
类为final，且内部字符数组为private final，因此常规操作无法修改其内容。

常规情况下的不可变性
以下代码仅改变了引用指向，而非原字符串内容：

text
String str = "Python";
str = "Java"; // 指向新对象
str = str.substring(1); // 创建新对象 "ava"
https://img/18-9-12-688492.jpg

substring、replace等方法均返回新String对象，原对象保持不变。

通过反射突破限制
利用反射可修改final数组内的元素，从而“改变”字符串：

text
String str = "Hello Python";
Field field = String.class.getDeclaredField("value");
field.setAccessible(true);
char[] value = (char[]) field.get(str);
value[6] = 'J';
value[7] = 'a';
value[8] = 'v';
value[9] = 'a';
System.out.println(str); // 输出 "Hello Java"
此操作违反了String的设计契约，可能引发安全问题，实际开发中应严格避免。

真正的不可变依赖于封装与final修饰，但反射机制赋予了“破坏”这种封装的途径，这提醒我们在安全敏感场景需额外防护。