---
title: Intellij IDEA 智能补全的 10 个姿势，太牛逼了。。
date: 2025-10-30 14:42:34
category: 工具
tags: IDEA
---

IntelliJ IDEA 的智能补全功能远超基本提示。在代码中输入 . 后，IDEA
会提供一系列后缀补全选项，能快速生成围绕当前表达式的一系列常用代码结构。下面通过动态示例展示十种提升编码效率的补全姿势。

1. 快速生成输出语句
   在表达式后输入 .sout，可快速将其包裹在 System.out.println() 中。
   https://img/sout.gif

2. 快速声明局部变量
   在表达式后输入 .var，IDEA 会自动推断类型并生成一个 final 类型的局部变量声明。
   https://img/var.gif

3. 快速声明成员变量
   在表达式后输入 .field，IDEA 会将其转换为当前类的成员变量。若当前上下文为静态方法，则生成静态变量。
   https://img/field.gif

4. 快速格式化字符串
   在字符串字面量后输入 .format，可快速生成 String.format(...) 语句。
   https://img/format.gif

5. 快速生成空值判断
   在变量后输入 .null 或 .nn (.notnull)，可快速生成 if (xx == null) 或 if (xx != null) 判断代码块。
   https://img/null.gif

6. 快速进行布尔取反与判断
   在布尔表达式后输入 .not 可快速取反，继续输入 .if 则可直接生成包含该条件的 if 语句块。
   https://img/notif.gif

7. 快速生成循环迭代
   在集合或数组后输入 .for、.fori (正向索引循环) 或 .forr (反向索引循环)，可快速生成对应的 for 循环代码块。
   https://img/for.gif

8. 快速生成返回语句
   在表达式后输入 .return，可快速生成 return 该表达式的语句。
   https://img/return.gif

9. 快速生成同步锁
   在任何对象后输入 .synchronized，可快速生成以该对象为锁的 synchronized 代码块。
   https://img/synchronized.gif

10. 快速生成 JDK8+ 代码
    在集合后输入 .stream 可快速转换为 Stream，配合其他补全可快速生成 Lambda 表达式或 Optional 包装代码。
    https://img/jdk8.gif