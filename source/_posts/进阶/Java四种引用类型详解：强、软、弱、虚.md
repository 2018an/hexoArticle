---
title: Java四种引用类型详解：强、软、弱、虚
date: 2025-10-15 11:36:33
category: 后端
tags: 进阶
---

https://img/17-12-27-5366993.jpg

从JDK 1.2开始，Java将对象引用分为四个级别，以便更灵活地控制对象生命周期。

1. 强引用（StrongReference）
   最常见的引用类型，只要强引用存在，垃圾收集器就不会回收该对象。即使内存不足导致OOM，也不会回收强引用对象。

text
User user = new User("Java技术");

2. 软引用（SoftReference）
   通过SoftReference类实现。在内存充足时，软引用对象不会被回收；当内存不足时，这些对象会被纳入回收范围。适用于实现内存敏感的缓存。

3. 弱引用（WeakReference）
   通过WeakReference类实现。无论内存是否充足，一旦发生GC，弱引用对象都会被回收。常用于WeakHashMap等场景，当键或值无其他引用时自动清理。

4. 虚引用（PhantomReference）
   通过PhantomReference类实现。虚引用不影响对象生命周期，也无法通过它访问对象。其唯一作用是接收对象被回收的系统通知，需与ReferenceQueue联合使用。

引用类型 回收时机 典型用途
强引用 永不回收 普通对象引用
软引用 内存不足时回收 缓存
弱引用 GC时立即回收 缓存、WeakHashMap
虚引用 回收前后通知 资源清理跟踪
了解这些引用类型有助于设计更高效、内存更敏感的应用。