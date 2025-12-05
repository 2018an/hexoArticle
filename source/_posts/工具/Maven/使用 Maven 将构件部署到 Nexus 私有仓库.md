---
title: 使用 Maven 将构件部署到 Nexus 私有仓库
date: 2025-10-30 14:42:34
category: 工具
tags: Maven
---


在 Nexus 3 中，通过界面直接上传构件的功能已被移除，需借助 Maven 命令或插件完成部署操作。

部署第三方 JAR 包
如果某个 JAR 包仅存在于本地，而公共仓库中没有，可以将其发布到私库供团队使用。

执行以下命令：

bash
mvn deploy:deploy-file \
-DgroupId=com.example \
-DartifactId=test \
-Dversion=0.0.1 \
-Dpackaging=jar \
-Dfile=/path/to/your/test-0.0.1.jar \
-Durl=http://nexus.example.com:8081/repository/3rd-party/ \
-DrepositoryId=nexus-repo
注意：-Dfile 指向的路径不应与本地仓库路径相同，否则可能导致错误。

部署自有项目
企业内部项目通常需要发布到私有仓库，以便其他模块引用。除了先打包再上传，也可直接通过 Maven 插件在构建过程中完成部署。

在项目的 pom.xml 中添加发布配置：

xml
<distributionManagement>
<repository>
<id>nexus-repo</id>
<name>Releases</name>
<url>http://nexus.example.com:8081/repository/maven-releases/</url>
</repository>
<snapshotRepository>
<id>nexus-repo</id>
<name>Snapshots</name>
<url>http://nexus.example.com:8081/repository/maven-snapshots/</url>
</snapshotRepository>
</distributionManagement>
然后在 IDE 或命令行中执行 mvn deploy 即可完成发布。

认证信息配置
上述命令或配置中使用的 repositoryId 需与 Maven settings.xml 中定义的服务器凭证对应：

xml
<servers>
<server>
<id>nexus-repo</id>
<username>admin</username>
<password>admin123</password>
</server>
</servers>
通过合理配置，可轻松实现构件的统一管理与团队共享。

在 Maven 项目中，远程仓库是获取依赖的重要来源。以下为几个常用的公开仓库地址，开发者可根据网络状况与需求选择合适的镜像站点，以加快构建速度。

text
https://repo1.maven.org/maven2/
这是 Maven 官方的中央仓库，全球开发者默认使用的依赖来源。

text
http://maven.jahia.org/maven2/
Jahia 提供的公共 Maven 仓库，常用于特定框架或组件的分发。

text
http://maven.aliyun.com/nexus/content/groups/public/
阿里云提供的国内镜像仓库，访问速度较快，适合在中国大陆使用。