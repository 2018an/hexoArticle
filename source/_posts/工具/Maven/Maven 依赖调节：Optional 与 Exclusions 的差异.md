---
title: Maven 依赖调节：Optional 与 Exclusions 的差异
date: 2025-10-30 14:42:34
category: 工具
tags: Maven
---

在 Maven 依赖管理中，<optional> 和 <exclusions> 都可用于控制依赖是否被传递，但它们的控制方向恰好相反。

Optional：在提供方设置，标记某个依赖为“可选”，阻止其向上传递。

Exclusions：在使用方设置，主动排除传递过来的某个依赖。

示例说明
假设项目结构如下：

Project-X 依赖 Project-A
Project-A 依赖 Project-B

场景一：使用 Optional
若 A 引用 B 时设置了 <optional>true</optional>，则 B 仅在 A 中可用，不会传递给 X。

xml
<!-- Project-A 的 pom.xml -->
<dependency>
    <groupId>sample.ProjectB</groupId>
    <artifactId>Project-B</artifactId>
    <version>1.0</version>
    <optional>true</optional>
</dependency>
此时 X 若需要 B，必须显式声明对 B 的依赖。

场景二：使用 Exclusions
若 A 引用 B 时未设置 optional，B 会默认传递给 X。如果 X 不需要 B，则可在依赖 A 时将其排除：

xml
<!-- Project-X 的 pom.xml -->
<dependency>
    <groupId>sample.ProjectA</groupId>
    <artifactId>Project-A</artifactId>
    <version>1.0</version>
    <exclusions>
        <exclusion>
            <groupId>sample.ProjectB</groupId>
            <artifactId>Project-B</artifactId>
        </exclusion>
    </exclusions>
</dependency>
使用建议
当某个依赖仅用于当前模块内部，不希望影响其他模块时，使用 Optional。

当引入的依赖传递了不需要的子依赖时，使用 Exclusions 进行排除。

两者结合使用，可以实现更精细的依赖传递控制，避免冗余和冲突。

更多详细说明可参考官方文档：
http://maven.apache.org/guides/introduction/introduction-to-optional-and-excludes-dependencies.html