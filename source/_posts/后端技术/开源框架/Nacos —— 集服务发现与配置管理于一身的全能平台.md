---
title: 阿里启动新项目：Nacos，比 Eureka 更强！
date: 2025-10-29 17:30:25
category: 后端
tags: 开源
---

![](img/18-11-21-1234410.jpg)

Nacos 是什么？
Nacos（Naming and Configuration Service）是阿里巴巴开源的下一代云原生应用基础设施。它定位为一个动态服务发现、配置管理和服务管理平台，旨在帮助开发者更轻松地构建和管理现代化的微服务架构。

Nacos 提供了一系列简单易用的特性，核心目标是帮助用户快速落地动态服务发现、服务健康监测、动态配置管理和服务元数据管理，是构建“以服务为中心”的应用架构（如微服务、云原生）的关键组件。

官方网站：https://nacos.io
开源地址：https://github.com/alibaba/nacos

全景图展示了其在微服务生态中的核心地位：
![](img/18-11-21-54787075.jpg)

核心概念解析
![](img/18-11-21-95099270.jpg)

服务 (Service)
一个或一组可供客户端远程调用的软件功能单元。Nacos 广泛支持多种服务生态，包括 Kubernetes Service、gRPC/Dubbo RPC Service 以及
Spring Cloud RESTful Service。

服务注册中心 (Service Registry)
存储服务、服务实例及其元数据的核心数据库。服务实例启动时自动注册，关闭时自动注销。消费者通过查询注册中心来获取可用的服务实例列表。

服务元数据 (Service Metadata)
用于精细化描述服务的数据，如服务端点、版本号、权重、标签、路由规则及安全策略等。

服务提供者 (Service Provider)
服务的提供方，负责将自身服务注册到 Nacos 服务器。

服务消费者 (Service Consumer)
服务的调用方，从 Nacos 服务器查询并调用所需服务。

配置 (Configuration)
从应用代码中分离出来的、用于适配不同运行环境的参数和变量。动态配置是调整系统运行时行为的重要手段。

配置管理 (Configuration Management)
对配置的编辑、存储、分发、版本管理、变更审计等一系列活动的统称。

命名服务 (Naming Service)
管理分布式系统中“名称”到“元数据”映射关系的服务。例如，将服务名解析为端点列表（服务发现），是命名服务的核心场景之一。

配置服务 (Configuration Service)
在应用运行时，提供动态配置管理能力的服务提供者。

Nacos 与 Spring Cloud 生态的对比
相较于 Spring Cloud Netflix 系列组件，Nacos 提供了一个更为统一和强大的解决方案。

一个简单的等式可以概括：Nacos ≈ Spring Cloud Eureka + Spring Cloud Config

Nacos 能够无缝集成 Spring 全家桶，并且在很多场景下可以替代以下两个核心组件：

替代 Eureka：通过 spring-cloud-starter-alibaba-nacos-discovery 实现更强大的服务注册与发现。

替代 Config：通过 spring-cloud-starter-alibaba-nacos-config 实现配置的集中管理和动态刷新。

Nacos 在数据一致性模型（支持AP和CP）、配置管理的实时性、以及运维的便捷性上，都提供了更优的选择。

参考资料
Nacos 官方文档