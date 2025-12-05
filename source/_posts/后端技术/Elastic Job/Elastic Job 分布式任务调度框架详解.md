---
title: Elastic Job 分布式任务调度框架详解
date: 2025-10-29 17:30:25
category: 后端
tags: Elastic Job
---

#### 概述

Elastic-Job 是一款专为分布式环境设计的任务调度系统，由 Elastic-Job-Lite 与 Elastic-Job-Cloud 两个独立模块构成。

Elastic-Job-Lite 被设计为轻量级、去中心化的架构，通过 jar 包方式提供服务，实现分布式任务的高效协调与管理。

#### 核心特性

**1、分片执行机制**

- 将总任务分解为多个子任务并行执行
- 支持根据服务器数量动态扩展或收缩任务处理能力
- 自动检测任务节点的加入与退出，实现无缝协调

**2、多样化任务模式**

- 支持基于定时器的任务调度
- 支持数据驱动的任务类型（规划中）
- 兼容常驻型与瞬时型任务
- 提供多语言任务执行支持

**3、云平台集成**

- 深度适配 Mesos、Kubernetes 等主流调度平台
- 任务运行不依赖特定 IP、磁盘等有状态资源
- 基于 Netflix Fenzo 实现智能资源分配与调度

**4、高可用保障**

- 定时自检与故障自动恢复机制
- 分布式环境下任务分片唯一性保障
- 支持任务失败自动转移与遗漏任务重新执行

**5、任务聚合管理**

- 同类任务汇聚至同一执行器统一处理
- 降低系统资源消耗与初始化成本
- 动态分配额外资源至新增任务

**6、便捷运维**

- 提供功能完善的运维管理界面
- 支持任务执行历史记录与追踪
- 注册中心数据一键导出，便于备份与问题排查

#### 系统架构示意图

**Elastic-Job-Lite 架构**

![image](img/18-2-24-59132374.jpg)

**Elastic-Job-Cloud 架构**

![image](http://ovfotjrsi.bkt.clouddn.com/docs/img/architecture/elastic_job_cloud.png)

#### 相关资源

> 官方网站：http://elasticjob.io/index_zh.html\
> 码云仓库：https://gitee.com/elasticjob\
> GitHub 地址：https://github.com/elasticjob/elastic-job\
> 应用企业列表：http://elasticjob.io/docs/elastic-job-lite/00-overview/company