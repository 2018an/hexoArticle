---
title: Java 12 引入的 Switch表达式
date: 2025-10-15 11:36:33
category: 后端
tags: 新特性
---

switch 语句是 Java 中的元老级控制结构，但它的语法多年来变化不大。Java 12 引入的 Switch
表达式（预览特性），彻底改变了它的书写方式，让代码变得更简洁、更安全、更强大。

告别繁琐：箭头语法与多重标签
假设我们有一个表示任务状态的枚举：

java
public enum TaskStatus {
CREATED, ASSIGNED, IN_PROGRESS, BLOCKED, COMPLETED, CANCELLED;
}
传统写法（Java 12 之前）：

java
private static int getStatusCodeOld(TaskStatus status) {
int code;
switch (status) {
case CREATED:
code = 100;
break;
case ASSIGNED:
case IN_PROGRESS:
code = 200; // 多个状态共享同一逻辑
break;
case COMPLETED:
code = 300;
break;
case CANCELLED:
case BLOCKED:
code = 400;
break;
default:
throw new IllegalStateException("未知状态");
}
return code;
}
冗长的 break，容易遗漏的 default，以及为了共享逻辑而故意安排的 case 穿透（这可能带来隐患）。

Java 12 新写法：

java
private static int getStatusCodeNew(TaskStatus status) {
return switch (status) {
case CREATED -> 100;
case ASSIGNED, IN_PROGRESS -> 200; // 一行处理多个case，清晰！
case COMPLETED -> 300;
case CANCELLED, BLOCKED -> 400;
// 不再需要default，因为枚举已覆盖所有情况，但编译器可能仍会提示。
// 若有未覆盖的可能性，必须提供default分支。
};
}
看，变化有多大！

箭头语法 (->)： 取代了冒号，意味着这个分支会直接产生一个值，并且不会发生穿透。

多重 case 标签： 可以用逗号将多个条件合并，逻辑一目了然。

作为表达式返回： switch 现在可以直接产生值并返回，无需先赋值给中间变量。

更安全： 箭头语法天然阻断了意外的 case 穿透，减少了Bug。

需要执行多条语句？用代码块！
如果某个分支需要更复杂的计算，可以使用花括号 {} 包裹代码块，并用 yield 关键字（在后续Java版本中稳定）返回结果。

java
private static String getStatusMessage(TaskStatus status) {
return switch (status) {
case COMPLETED -> "任务已成功完成。";
case CANCELLED -> {
log.warn("任务被取消");
yield "任务已被取消。"; // 在块中使用 yield 返回结果
}
case BLOCKED -> {
String msg = "任务受阻，原因：" + fetchBlockReason();
yield msg;
}
default -> "任务正在进行中。";
};
}
重要提示
Switch 表达式在 Java 12 和 13 中作为预览特性提供。这意味着你需要在使用时通过编译器参数 --enable-preview 来启用它，并且其语法在
Java 13 中略有微调（例如，引入了 yield）。该特性在 Java 14 中正式稳定。

总结
Java 12 的 Switch 表达式是一次巨大的语法革新。它通过引入箭头语法、多重标签和表达式返回值，显著提升了代码的简洁性、可读性和安全性。虽然最初是预览版，但它指明了语言进化的方向，值得每一位
Java 开发者立即学习和尝试。拥抱这个新变化，让你写的 switch 从此告别冗长与潜在风险。

https://img/20190613135450.png
https://img/20190613135537.png