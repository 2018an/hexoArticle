---
title: Elastic Job 动态调节功能详解
date: 2025-10-29 17:30:25
category: 后端
tags: Elastic Job
---

lastic Job 内置了易于操作的运维控制台，方便用户实时监控任务状态、动态调整任务参数、执行任务操作及查询任务信息。

#### 设计思路

运维控制台与 elastic-job-lite 核心模块无直接耦合，通过读取任务注册中心的数据展示任务状态，或更新注册中心配置来实现全局参数调整。

需注意，控制台仅能控制任务本身的运行状态，无法直接启停任务进程，因为控制台与任务执行节点是完全解耦的。

#### 主要功能

- 身份验证与访问控制

- 注册中心及事件追踪数据源配置

- 任务参数快速修改

- 从任务与服务器维度查看运行状态

- 任务启停、禁用、移除等生命周期管理

- 执行事件记录查询

#### 功能限制

- 新增任务

任务在首次执行时会自动注册。由于 Elastic-Job-Lite 以 `jar` 包形式运行，不具备任务分发能力。如需通过运维平台完整发布任务，请选用
`Elastic-Job-Cloud` 版本。

#### 控制台部署指南

**1、获取最新稳定版源码包**

> 下载地址：https://github.com/elasticjob/elastic-job-lite

此处我们选取最新的 `2.1.5` 版本发布包。

**2、编译源码包**

解压至任意目录，执行 `mvn install` 完成编译。
cd d:/elastic-job-lite-2.1.5
mvn install

text

![](img/18-3-19-70978286.jpg)

**3、启动运维平台**

在编译输出目录 `d:\elastic-job-lite-2.1.5\elastic-job-lite\elastic-job-lite-console\target` 中找到打包文件
`elastic-job-lite-console-2.1.5.tar.gz`，解压后运行 `bin` 目录下的 `start.bat`（Windows）或 `start.sh`（Linux）。

![](img/18-3-19-67592187.jpg)

默认服务端口为 `8899`，可通过启动脚本的 `-p` 参数指定其他端口。

**4、访问控制台**

系统预设管理员账户（root/root）与访客账户（guest/guest）。管理员具备全部权限，访客仅可查看。账户信息可通过
`conf\auth.properties` 文件调整。
root.username=root
root.password=root
guest.username=guest
guest.password=guest

text

浏览器访问 `http://localhost:8899/`，输入账户信息即可登录。

![](img/18-3-19-91428457.jpg)

**5、连接注册中心**

控制台启动后，需先添加目标注册中心并建立连接。

![](img/18-3-19-89674953.jpg)

**6、任务管理操作**

支持任务配置更新、详情查看、暂停运行、立即终止、手动触发等操作。注意：任务终止后需重启应用才能恢复，控制台无法直接重新启动。

![](img/18-3-19-94019933.jpg)

以上就是 Elastic-Job 运维控制台的搭建与使用全流程说明。

