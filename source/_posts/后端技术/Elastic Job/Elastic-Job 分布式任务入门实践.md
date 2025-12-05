---
title: Elastic-Job 分布式任务入门实践
date: 2025-10-29 17:30:25
category: 后端
tags: Elastic Job
---

Elastic-Job 支持通过 JAVA API 与 Spring 配置两种方式定义任务。本文采用 JAVA API 方式演示基础任务创建。考虑到当前主流开发方式，建议使用
Spring Boot 整合而非传统 Spring 配置文件。

Elastic-Job 依赖 Zookeeper 作为分布式协调组件，目前仅支持 Zookeeper。本文假设已具备 Zookeeper 集群环境。

#### 运行环境

1、JDK 版本需为 1.7 或更高。

2、Zookeeper 版本需为 3.4.6 或更高。

3、Maven 版本需为 3.0.4 或更高。

#### 添加 Maven 依赖

```xml

<dependency>
    <groupId>com.dangdang</groupId>
    <artifactId>elastic-job-lite-core</artifactId>
    <version>2.1.5</version>
</dependency>
```

注意：该依赖可能引入多个版本的 curator-client，导致方法调用异常。建议显式添加 curator-client 依赖避免冲突。

```xml

<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-client</artifactId>
    <version>2.11.1</version>
</dependency>
```

定义任务
Elastic-Job 提供 Simple、Dataflow 及 Script 三类任务模板。

执行方法参数 shardingContext 包含任务配置、分片信息与运行时上下文。可通过 getShardingTotalCount()、getShardingItem()
等方法获取分片总数、当前节点分片编号等数据。

以下创建一个 Simple 类型任务示例：

```java
public class MyElasticJob implements SimpleJob {

    @Override
    public void execute(ShardingContext context) {
        switch (context.getShardingItem()) {
            case 0: {
                System.out.println("MyElasticJob - 0");
                break;
            }
            case 1: {
                System.out.println("MyElasticJob - 1");
                break;
            }
            case 2: {
                System.out.println("MyElasticJob - 2");
                break;
            }
            default: {
                System.out.println("MyElasticJob - default");
            }
        }
    }
}
```

注：上述代码中的 0-2 涉及分布式任务分片机制

分布式任务执行时，需将总体任务拆分为多个独立任务单元，由不同服务器分别处理特定分片。

举例说明：假设需遍历数据库表的任务，现有 2 台服务器。为提升效率，每台服务器应处理 50% 数据。此时可将任务分为 2 个分片，服务器
A 处理奇数 ID 记录，服务器 B 处理偶数 ID 记录。若分为 10 个分片，则数据处理逻辑应为：每片对应 ID%10 的余数，假设服务器 A 分配至
0-4 分片，则处理 ID 以 0-4 结尾的记录；服务器 B 分配至 5-9 分片，处理 ID 以 5-9 结尾的记录。

分片策略详解：http://elasticjob.io/docs/elastic-job-lite/02-guide/job-sharding-strategy/

任务配置
Elastic-Job 配置分为 Core、Type 和 Root 三个层级，采用类似装饰器模式进行组装。

Core 对应 JobCoreConfiguration，提供任务核心配置（如任务名、分片总数、CRON 表达式等）。

Type 对应 JobTypeConfiguration，包含 SIMPLE、DATAFLOW、SCRIPT 三种类型的专属配置（如 DATAFLOW 是否流式处理、SCRIPT 的执行命令等）。

Root 对应 JobRootConfiguration，提供 Lite 与 Cloud 两种部署模式的配置（如 Lite 是否覆盖本地配置、Cloud 的 CPU/内存资源设置等）。

在 Spring Boot 启动类中配置任务：

```java
private static CoordinatorRegistryCenter createRegistryCenter() {
    CoordinatorRegistryCenter regCenter = new ZookeeperRegistryCenter(new ZookeeperConfiguration("192.168.10.31:2181,192.168.10.32:2181,192.168.10.33:2181", "elastic-job-demo"));
    regCenter.init();
    return regCenter;
}

private static LiteJobConfiguration createJobConfiguration() {
    // 配置任务核心参数
    JobCoreConfiguration simpleCoreConfig = JobCoreConfiguration.newBuilder("demoSimpleJob", "0/15 * * * * ?", 10).build();

    // 定义 SIMPLE 类型任务配置
    SimpleJobConfiguration simpleJobConfig = new SimpleJobConfiguration(simpleCoreConfig, MyElasticJob.class.getCanonicalName());

    // 构建 Lite 模式任务根配置
    LiteJobConfiguration simpleJobRootConfig = LiteJobConfiguration.newBuilder(simpleJobConfig).build();
}

@Bean
public CommandLineRunner commandLineRunner() {
    return (String... args) -> {
        new JobScheduler(createRegistryCenter(), createJobConfiguration()).init();
    };
}
```

SimpleJobConfiguration 实现了 JobTypeConfiguration 接口。

LiteJobConfiguration 实现了 JobRootConfiguration 接口。

通过 CommandLineRunner 确保 Spring Boot 启动后初始化 Elastic-Job 任务。

更多 Spring Boot 基础配置请参考相关专题文档。

完整配置参考官方文档：http://elasticjob.io/docs/elastic-job-lite/02-guide/config-manual/

任务执行
通过 maven 命令 spring-boot:run 启动应用。

执行输出示例：

```text
MyElasticJob - 0
MyElasticJob - 1
MyElasticJob - 2
MyElasticJob - default
MyElasticJob - default
MyElasticJob - default
MyElasticJob - default
MyElasticJob - default
MyElasticJob - default
MyElasticJob - default
```

单实例运行时所有 10 个分片均由同一节点处理。将应用打包后以不同端口启动多个实例，可观察到分片分配变化：

实例 A 输出：

```text
MyElasticJob - 0
MyElasticJob - 1
MyElasticJob - 2
MyElasticJob - default
MyElasticJob - default
```

实例 B 输出：

```text
MyElasticJob - default
MyElasticJob - default
MyElasticJob - default
MyElasticJob - default
MyElasticJob - default
```

输出结果验证了分片成功。停止任一实例后，系统自动重新分片，剩余实例将处理全部分片。

分片机制极大提升了任务处理效率与灵活性，整体架构清晰易懂，推荐在分布式场景中应用。

更多实战技巧将持续分享，欢迎关注与转发支持！