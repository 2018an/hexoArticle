---
title: Java Jar 包操作指南
date: 2025-10-14 14:42:34
category: 后端
---


JAR（Java Archive）是一种跨平台的压缩文件格式，常用于打包 Java 类、资源文件和元数据。

常用 jar 命令
参数 说明
c 创建 JAR 包
t 列出 JAR 内容
x 解压 JAR 包
u 更新文件到 JAR
f 指定 JAR 文件名
v 输出详细信息
0 不压缩（仅存储）
M 不生成 MANIFEST.MF
C 切换目录执行命令
实用示例
bash

# 创建不压缩的 JAR（适用于快速更新）

jar cvfM0 app.jar BOOT-INF/ META-INF/ org/

# 查看 JAR 内容

jar tf app.jar

# 解压 JAR

jar xvf app.jar

# 向 JAR 中添加/更新文件

jar uf app.jar BOOT-INF/classes/application.yml
使用场景
快速修改线上 JAR 包中的配置文件。

查看第三方库的内容结构。

构建可分发的应用程序包。