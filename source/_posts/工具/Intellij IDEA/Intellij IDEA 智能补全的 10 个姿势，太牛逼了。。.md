---
title: Intellij IDEA 智能补全的 10 个姿势，太牛逼了。。
date: 2025-10-30 14:42:34
category: 工具
tags: IDEA
---


关于智能提示，这次我再分享一篇关于自动补全方面的。

首先来看一下下面这张图，在方法里面有效位置后面输入一个.，最后面会列表所有可用的自动补全的关键字，这也就是今天要分享的内容。

![](http://img.javastack.cn/20190520171322.png)

下面再介绍几个它们的用法，其实很简单，跟上次一样，这次我同样还是录了动图，这样看得更直观，看起来更牛逼。。

## 1、快速打印输出

除了用 sout 开头快速生成，还能在后面快速生成。

![](http://img.javastack.cn/sout.gif)

## 2、快速定义局部变量

在字符串或者数字……后面输入 .var，回车，IDEA会自动推断并快速定义一个局部变量，不过它是 final 类型的。

![](http://img.javastack.cn/var.gif)

## 3、快速定义成员变量

在值后面输入.field，可以快速定义一个成员变量，如果当前方法是静态的，那生成的变量也是静态的。

![](http://img.javastack.cn/field.gif)

## 4、快速格式化字符串

在字符串后面输入.format，回车，IDEA会自动生成 String.format...语句，牛逼吧！

![](http://img.javastack.cn/format.gif)

## 5、快速判断（非）空

```
if (xx != null)
if (xx == null)
```

像上面这种判断空/非空的情况非常多吧，其实可以快速生成 if 判断语句块，非空：.notnull 或者 .nn，空：.null。

![](http://img.javastack.cn/null.gif)

## 6、快速取反判断

输入 .not 可以让布尔值快速取反，再输入 .if 可快速生成 if 判断语句块。

![](http://img.javastack.cn/notif.gif)

## 7、快速遍历集合

下面是几种 for 循环语句的快速生成演示，.for, .fori, .forr 都可以满足你的要求。

![](http://img.javastack.cn/for.gif)

## 8、快速返回值

在值后面输入.return，可以让当前值快速返回。

![](http://img.javastack.cn/return.gif)

## 9、快速生成同步锁

在对象后面输入.synchronized，可以快速生成该对象的同步锁语句块。

![](http://img.javastack.cn/synchronized.gif)

## 10、快速生成JDK8语句

下面演示的是快速生成 Lambda 以及 Optional 语句。

![](http://img.javastack.cn/jdk8.gif)

好了，今天栈长就介绍了 Intellij IDEA 如何更使用快速补全功能、涨姿势了吧。

- Intellij IDEA 最常用配置详细图解
- Intellij IDEA 非常6的10个姿势
- Intellij IDEA 所有乱码解决方案
- Intellij IDEA 阅读源码的4个绝技
- Intellij IDEA Debug调试技巧
