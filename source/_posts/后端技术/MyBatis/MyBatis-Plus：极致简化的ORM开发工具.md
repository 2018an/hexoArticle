---
title: MyBatis-Plus：极致简化的ORM开发工具
date: 2025-10-29 17:30:25
category: 后端
tags: MyBatis
---

今天向使用Mybatis的开发者推荐一款高效工具：MyBatis-Plus（简称MP）。该工具在Mybatis基础上提供增强功能，完全兼容原生特性，旨在提升开发效率、简化编码工作。

其设计理念是成为Mybatis最理想的辅助工具，如同经典游戏中的黄金搭档，协同工作，效能倍增。

![](img/20190529110805.png)

官方资源：

> 官网：https://mybatis.plus/

> 代码仓库：https://github.com/baomidou/mybatis-plus

目前该项目已在GitHub获得超过5,000颗星标。

![](img/20190529111707.png)

## 核心特性

- 非侵入设计：完全兼容原生Mybatis，引入后对现有项目无任何影响
- 极小开销：自动注入基础CRUD操作，性能损耗微乎其微，支持面向对象编程
- 强化CRUD功能：内置通用Mapper与Service，少量配置即可实现单表主要操作，配备强大条件构造器
- Lambda支持：通过Lambda表达式便捷构建查询条件，避免字段拼写错误
- 多数据库兼容：支持MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer等主流数据库
- 智能主键生成：提供4种主键策略（包含分布式唯一ID生成序列），自由配置解决主键问题
- XML热加载：Mapper对应XML支持热加载，简单CRUD操作可无需XML配置
- ActiveRecord模式：实体类继承Model即可实现完整CRUD操作
- 全局方法注入：支持全局通用方法注入（一次编写，随处使用）
- 关键词转义：自动处理数据库保留关键词，支持自定义关键词列表
- 代码生成器：通过代码或Maven插件快速生成各层代码，支持模板引擎，提供丰富自定义选项
- 分页插件：基于Mybatis物理分页，配置后分页查询与普通列表查询同样简便
- 性能分析插件：输出SQL语句及执行耗时，开发测试阶段快速定位性能瓶颈
- 全局拦截插件：全表删除、更新操作智能分析与阻断，支持自定义拦截规则防止误操作
- SQL注入防护：内置SQL注入剥离器，有效防范注入攻击

## 架构设计

![](img/20190529110859.png)

## 快速入门

#### 1、添加项目依赖

```xml

<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.1.1</version>
</dependency>
```

2、继承基础接口

```java
public interface UserMapper extends BaseMapper<User> {

}
```

3、数据查询示例

```java
List<User> userList = userMapper.selectList(
        new QueryWrapper<User>()
                .lambda()
                .ge(User::getAge, 18)
);
```

MyBatis-Plus将自动生成以下SQL：

```sql
SELECT *
FROM user
WHERE age >= 18
```