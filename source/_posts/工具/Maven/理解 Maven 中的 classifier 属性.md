---
title: 理解 Maven 中的 classifier 属性
date: 2025-10-30 14:42:34
category: 工具
tags: Maven
---

有时在声明依赖时，明明坐标正确，却无法成功下载。例如：

xml
<dependency>
<groupId>net.sf.json-lib</groupId>
<artifactId>json-lib</artifactId>
<version>2.4</version>
</dependency>
这个配置看上去没有问题，但实际上 Maven 无法找到对应的 jar 文件。查看仓库目录结构可以看到，该版本下并没有默认的
json-lib-2.4.jar，而是存在多个带有后缀的变体，例如：

json-lib-2.4-jdk13.jar

json-lib-2.4-jdk15.jar

对应的源码包、文档包等

在这些情况下，就需要使用 classifier 属性来指定所需的具体构件。classifier 用于区分同一版本下不同用途或不同环境的输出文件。

修正后的依赖配置如下：

xml
<dependency>
<groupId>net.sf.json-lib</groupId>
<artifactId>json-lib</artifactId>
<version>2.4</version>
<classifier>jdk15</classifier>
</dependency>
添加 classifier 后，Maven 就会下载正确的文件 json-lib-2.4-jdk15.jar。

注：json-lib 已停止维护，建议选用 fastjson、Jackson、Gson 等现代 JSON 处理库。