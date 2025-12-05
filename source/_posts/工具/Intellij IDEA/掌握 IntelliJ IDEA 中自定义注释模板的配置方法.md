---
title: 掌握 IntelliJ IDEA 中自定义注释模板的配置方法
date: 2025-10-30 14:42:34
category: 工具
tags: IDEA
---

从 Eclipse 转向 IntelliJ IDEA 的过渡期可能会遇到一些适应性问题，其中注释模板的配置方式差异尤为显著。经过多次尝试与磨合，如今已能完全适应
IDEA 的开发环境。尽管 IDEA 在众多方面表现出色，但其注释模板的自定义流程相较于 Eclipse 的直观设置，确实显得更为复杂，甚至需要借助
Groovy 脚本来实现高级功能。

本文将聚焦于解决 IDEA 中自定义注释模板这一常见挑战，详细介绍两种核心配置方式。

文件与代码模板 (File and Code Templates)
此功能用于预设新建各类文件时的初始代码结构，包括自动生成的文件头注释。

以下以配置 Java 类文件头注释为例进行演示：

依次进入设置菜单：Editor > File and Code Templates > Files
https://img/18-10-23-68361689.jpg

在文件列表中选择 Class，可以看到模板中已包含一行引用代码：

text
#parse("File Header.java")
这行代码引用了另一个片段模板。

这个被引用的片段定义在：Editor > File and Code Templates > Includes
https://img/18-10-23-41018185.jpg

编辑 File Header 片段即可自定义所有 Java 类的通用注释模板。模板支持使用预定义变量，如 ${DATE}、${TIME}、${USER} 等，其语法基于
Apache Velocity 模板语言。
https://img/18-10-23-9061457.jpg

配置完成后，每次新建 Java 类都会自动填充预设的注释头。

实时模板 (Live Templates)
实时模板用于通过简短的缩写，在代码任意位置动态生成预设的代码片段，非常适合手动添加类或方法注释。

进入实时模板设置：Editor > Live Templates
https://img/18-10-23-56817313.jpg

点击右上角 + 号，选择 Live Template 创建一个新模板。需要配置以下关键项：

Abbreviation（缩写）： 触发模板的快捷词，例如 cc 代表类注释。

Description（描述）： 模板的简要说明。

Template text（模板文本）： 注释的具体内容，$ 包围的为变量（如 $NAME$）。
https://img/18-10-23-85720861.jpg

点击 Edit variables 按钮可为变量设置表达式或默认值。

最后，点击 Define 链接，指定该模板的应用范围（例如：Java）。
https://img/18-10-23-35553716.jpg

配置方法注释模板的注意点：

方法注释的配置流程类似，但 params 变量需要配置为特定的 Groovy 表达式，以自动获取方法参数名：

text
groovyScript("def result=''; def params=\"${_1}\".replaceAll('[\\\\[|\\\\]|\\\\s]', '').split(',').toList(); for(i = 0;
i < params.size(); i++) {result+=' * @param ' + params[i] + ((i < params.size() - 1) ? '\\n' : '')}; return result",
methodParameters())
https://img/18-10-23-59877084.jpg
https://img/18-10-23-24832650.jpg

一个关键技巧是：使用方法注释缩写（如 mc）时，光标需放置在方法体内，而非方法上方，否则 @param 可能无法正确解析出参数名。