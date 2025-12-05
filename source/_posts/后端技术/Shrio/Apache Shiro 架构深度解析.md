---
title: Apache Shiro 架构深度解析
date: 2025-10-29 17:30:25
category: 后端
tags: Shiro
---

![image](https://upload.wikimedia.org/wikipedia/commons/0/06/Apache_Shiro_Logo.png)

## Shiro框架概述

Apache Shiro是一个功能强大且灵活的开源安全框架，它优雅地处理身份认证、授权、企业会话管理和加密等安全需求。

Apache Shiro的首要设计目标是简单易用和理解。安全领域有时会显得复杂甚至令人困惑，但它本不应如此。框架应当尽可能隐藏复杂性，提供清晰直观的API，简化开发人员实现应用程序安全的工作。

> 官方网站：http://shiro.apache.org

## Shiro核心功能

使用Apache Shiro可以实现以下功能：

- 验证用户身份真实性

- 对用户执行访问控制，例如：

1. 判断用户是否被分配了特定安全角色；

2. 判断用户是否被允许执行某项操作；

- 在任何环境下使用Session API，即使没有Web或EJB容器；

- 在认证、访问控制或会话生命周期中对事件作出响应；

- 聚合一个或多个用户安全数据源，形成统一的复合用户视图；

- 启用单点登录(SSO)功能；

- 为未登录用户提供"记住我"服务；

- 以及更多——全部通过紧密结合、易于使用的API实现。

Shiro致力于在所有应用环境中实现这些目标——从最简单的命令行应用程序到最复杂的企业级应用，不强制依赖任何第三方框架、容器或应用服务器。当然，该项目会尽可能融入这些环境，但它能够在任何环境下立即投入使用。

## 框架特性

Apache Shiro是一个具备丰富功能的综合性应用程序安全框架。

![image](http://dl2.iteye.com/upload/attachment/0093/9788/d59f6d02-1f45-3285-8983-4ea5f18111d5.png)

##### Shiro开发团队将其称为"应用程序安全的四大基石"——身份认证、授权、会话管理和加密：

- 身份认证：有时简称为"登录"，这是验证用户声称身份真实性的过程；

- 授权：访问控制的过程，即确定"谁"能够访问"什么"资源；

- 会话管理：管理用户特定的会话，即使在非Web或EJB应用程序中；

- 加密：通过使用加密算法保护数据安全，同时保持易用性；

##### 还提供了额外功能来支持和强化不同环境下的安全关注点，特别是：

- Web支持：Shiro的Web支持API能够轻松保护Web应用程序；

- 缓存：缓存是Apache Shiro中的一等公民，确保安全操作快速高效；

- 并发：Apache Shiro利用其并发特性支持多线程应用程序；

- 测试：提供测试支持，帮助编写单元测试和集成测试，确保安全按预期工作；

- "身份模拟"：允许用户（在获得许可时）假设另一用户身份的功能，在管理脚本中很有用；

- "记住我"：在会话中记住用户身份，用户只需在必要时登录；

## Shiro架构设计

Apache Shiro的设计目标是通过直观易用的方式简化应用程序安全。Shiro的核心设计反映了大多数人对应用程序安全的思考方式——在某人（或某物）与应用程序交互的背景下进行。

应用软件通常基于用户上下文设计。也就是说，您通常会根据用户将要（或应该）如何与软件交互来设计用户界面或服务API。例如，您可能会说："
如果与我的应用程序交互的用户已登录，我将显示一个可点击的按钮来查看他们的账户信息。如果未登录，我将显示登录按钮。"

这种简单陈述表明应用程序很大程度上是为满足用户需求和请求而编写的。即使"用户"是另一个软件系统而非人类，您仍然需要编写代码来响应基于当前与软件交互的人或物的行为。

Shiro在其设计中体现了这些概念。通过匹配软件开发人员已经熟悉的直观模式，Apache Shiro在几乎所有应用程序中保持了直观性和易用性。

在最高概念层次上，Shiro架构包含三个核心概念：Subject、SecurityManager和Realms。

下图展示了这些组件如何交互的高级概述，我们将在下面详细讨论每个概念：

![image](http://dl2.iteye.com/upload/attachment/0093/9790/5e0e9b41-0cca-367f-8c87-a8398910e7a6.png)

- **Subject**

如我们教程中提到的，Subject实质上是当前执行用户的特定安全"视图"。鉴于"User"
一词通常指人，而Subject可以代表人，也可以代表第三方服务、守护进程账户、定时任务或任何类似实体——基本上是当前与软件交互的任何事物。

所有Subject实例都绑定到（且必须绑定）一个SecurityManager。当您与Subject交互时，这些交互会转换为与SecurityManager的特定subject交互。

- **SecurityManager**

SecurityManager是Shiro架构的核心，作为一个"保护伞"
对象，协调内部安全组件共同构成对象图。然而，一旦SecurityManager及其内置对象图为应用程序配置完成，它基本上独立运行，应用程序开发人员大部分时间都使用Subject
API。

稍后将更详细讨论SecurityManager，但重要的是认识到，当您与Subject交互时，实质上是幕后的SecurityManager处理所有繁重的Subject安全操作。这反映在上述基本流程图中。

- **Realms**

Realms充当Shiro与应用程序安全数据之间的"桥梁"或"连接器"
。当实际上需要与安全相关数据（如用于执行身份验证和授权的用户账户）交互时，Shiro从一个或多个为应用程序配置的Realm中查找这些数据。

从这个意义上说，Realm本质上是特定于安全的DAO：它封装了数据源的连接细节，使Shiro所需的相关数据可用。配置Shiro时，必须指定至少一个用于身份验证和/或授权的Realm。SecurityManager可以配置多个Realms，但至少需要一个。

Shiro提供了现成的Realms来连接多种安全数据源，如LDAP、关系数据库(JDBC)
、文本配置源（如INI和属性文件）等。如果默认Realm不符合需求，您可以插入自己的Realm实现来代表自定义数据源。

与其他内置组件一样，Shiro SecurityManager控制Realms如何用于获取安全和身份数据以代表Subject实例。

下图展示了Shiro的核心架构概念，随后是每个概念的简要总结：

![image](http://dl2.iteye.com/upload/attachment/0093/9792/9b959a65-799d-396e-b5f5-b4fcfe88f53c.png)

- **Subject** (org.apache.shiro.subject.Subject)

当前与软件交互的实体（用户、第三方服务、定时任务等）的特定安全"视图"；

- **SecurityManager** (org.apache.shiro.mgt.SecurityManager)

如上所述，SecurityManager是Shiro架构的核心。它基本上是一个"保护伞"
对象，协调其管理的组件确保它们协同工作。它还管理每个应用程序用户的Shiro视图，因此知道如何为每个用户执行安全操作；

- **Authenticator** (org.apache.shiro.authc.Authenticator)

Authenticator是负责执行用户身份验证（登录）尝试的组件。当用户尝试登录时，Authenticator执行此逻辑。Authenticator知道如何与一个或多个Realm协调来存储相关用户/账户信息。从这些Realm获取的数据用于验证用户身份，确保用户确实是他们声称的身份；

- **Authentication Strategy** (org.apache.shiro.authc.pam.AuthenticationStrategy)

如果配置了多个Realm，AuthenticationStrategy将协调这些Realm，确定身份验证尝试成功或失败的条件；

- **Authorizer** (org.apache.shiro.authz.Authorizer)

Authorizer负责确定应用程序中的用户访问控制。它是最终判定用户是否被允许执行某项操作的机制。与Authenticator类似，Authorizer也知道如何协调多个后端数据源来访问角色和权限信息。Authorizer使用此信息准确确定用户是否被允许执行给定操作；

- **SessionManager** (org.apache.shiro.session.SessionManager)

SessionManager知道如何创建和管理用户Session生命周期，为所有环境下的用户提供健壮的Session体验。这是安全框架领域的独特特性——Shiro能够在任何环境下本地管理用户Session，即使没有可用的Web/Servlet或EJB容器，它也会使用其内置的企业级会话管理提供相同的编程体验。SessionDAO允许任何数据源用于持久化会话；

- **SessionDAO** (org.apache.shiro.session.mgt.eis.SessionDAO)

SessionDAO代表SessionManager执行Session持久化操作。这允许将会话管理基础架构插入任何数据存储；

- **CacheManager** (org.apahce.shiro.cache.CacheManager)

CacheManager创建并管理其他Shiro组件使用的Cache实例生命周期。由于Shiro能够访问许多后端数据源进行身份验证、授权和会话管理，缓存在框架中一直是一等架构功能，用于在使用这些数据源时提高性能。任何现代开源和/或企业缓存产品都可以插入Shiro，提供快速高效的用户体验；

- **Cryptography** (org.apache.shiro.crypto.*)

密码学是企业安全框架的自然补充。Shiro的crypto包包含易于使用和理解的加密Ciphers、Hasher以及不同的编码器实现。此包中的所有类都经过精心设计，易于使用和理解。任何使用Java本地加密支持的人都知道它可能是一个难以驾驭的挑战。Shiro的加密API简化了复杂的Java机制，使加密对普通开发人员也变得易于使用；

- **Realms** (org.apache.shiro.realm.Realm)

如上所述，Realms在Shiro和应用程序安全数据之间充当"桥梁"或"连接器"
。当实际上需要与安全相关数据交互时，Shiro从一个或多个为应用程序配置的Realm中查找这些数据。您可以根据需要配置多个Realm，Shiro将为身份验证和授权进行必要的协调。

**SecurityManager详解**

由于Shiro的API鼓励以Subject为中心的编程方式，大多数应用程序开发人员很少（如果有的话）直接与SecurityManager交互。尽管如此，了解SecurityManager的工作原理仍然很重要，特别是在为应用程序配置SecurityManager时。

**设计理念**

如前所述，应用程序的SecurityManager执行安全操作并管理所有应用程序用户的状态。在Shiro的默认SecurityManager实现中，这包括：

- 身份验证

- 授权

- 会话管理

- 缓存管理

- Realm协调

- 事件传播

- "记住我"服务

- Subject创建

- 注销

以及更多。

但要尝试在单个组件中管理这么多功能是很困难的。而且，如果所有内容都集中在一个实现类中，要使这些功能灵活且可定制将会非常困难。

为了简化配置并实现灵活配置/可插拔性，Shiro的实现采用了高度模块化设计——由于这种模块化，SecurityManager实现实际上并不做太多事情。相反，SecurityManager实现主要作为一个轻量级"
容器"组件，将所有行为委托给嵌套/包装的组件。这种"包装器"设计体现在上面的详细架构图中。

虽然组件实际执行逻辑，但SecurityManager实现知道何时以及如何协调组件以完成正确的行为。SecurityManager实现和组件都符合JavaBean规范，允许您通过标准JavaBean的访问器/修改器方法轻松自定义可插拔组件。这意味着Shiro的架构组件性能够将自定义行为转化为非常容易的配置文件。

> **简易配置**
>
> 由于符合JavaBean规范，通过任何支持JavaBean风格配置的机制可以轻松使用自定义组件配置SecurityManager，如Spring、Guice、JBoss等。