---
title: Java 11 为字符串新增了多个便捷方法
date: 2025-10-15 11:36:33
category: 后端
tags: 新特性
---

http://qianniu.javastack.cn/18-9-26/91229065.jpg

Java 11 已经正式推出，其中对 String 类的新增方法特别引人注目。虽然之前概述过 Java 11 的八大更新，但关于字符串处理的增强部分值得单独展开，其设计巧妙且实用。

Java 11 为字符串新增了多个便捷方法，具体如下：

```text
// 检查字符串是否仅包含空白字符
"   ".isBlank(); // 结果为 true

// 移除字符串首尾的空白字符
"  Java实战  ".strip(); // 返回 "Java实战"

// 仅移除末尾空白
"  Java实战  ".stripTrailing(); // 返回 "  Java实战"

// 仅移除开头空白
"  Java实战  ".stripLeading(); // 返回 "Java实战  "

// 重复字符串指定次数
"Java".repeat(3); // 返回 "JavaJavaJava"

// 按行拆分并统计行数
"A\nB\nC".lines().count(); // 返回 3
```
在这些新方法中，repeat 和 lines 尤为实用，下面进一步探讨其用法。

repeat 方法详解
repeat 方法用于将字符串重复多次，可替代以往常用的工具类方法，例如 StringUtils.repeat。其内部实现如下：

```text
public String repeat(int count) {
if (count < 0) {
throw new IllegalArgumentException("重复次数不能为负数：" + count);
}
if (count == 1) {
return this;
}
final int len = value.length;
if (len == 0 || count == 0) {
return "";
}
if (len == 1) {
final byte[] single = new byte[count];
Arrays.fill(single, value[0]);
return new String(single, coder);
}
if (Integer.MAX_VALUE / count < len) {
throw new OutOfMemoryError("重复 " + len + " 字节的字符串 " + count +
" 次将超出字符串最大长度限制。");
}
final int limit = len * count;
final byte[] multiple = new byte[limit];
System.arraycopy(value, 0, multiple, 0, len);
int copied = len;
for (; copied < limit - copied; copied <<= 1) {
System.arraycopy(multiple, 0, multiple, copied, copied);
}
System.arraycopy(multiple, 0, multiple, copied, limit - copied);
return new String(multiple, coder);
}
```
实际使用示例如下：

```text
String str = "Java";

// 传入负数将抛出 IllegalArgumentException
System.out.println(str.repeat(-2));

// 传入 0 则返回空字符串
System.out.println(str.repeat(0));

// 重复三次
System.out.println(str.repeat(3)); // 输出 JavaJavaJava

// 若重复次数极大，会引发 OutOfMemoryError
System.out.println(str.repeat(Integer.MAX_VALUE));
```
由此可见，repeat 方法并非无限可用，当重复次数过大导致内存超限时，会抛出内存溢出异常。

lines 方法解析
```text
public Stream<String> lines() {
return isLatin1() ? StringLatin1.lines(value)
: StringUTF16.lines(value);
}
```
lines 方法将字符串按行拆分为一个 Stream<String>，能自动识别 \n 和 \r 作为换行符。

```text
// 统计行数，输出 4
System.out.println("A\nB\nC\rD".lines().count());
```
这一方法在处理多行文本时非常方便，例如批量读取文件内容后直接转换为 Stream 并逐行处理，能优雅地处理不同平台的换行符差异。

这些新增的字符串方法，虽然看似简单，却能在日常开发中显著提升代码的简洁性与表达力，是 Java 11 中不容忽视的实用改进。