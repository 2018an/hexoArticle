---
title: Spring Boot 应用无缝集成 Apache Kafka 实战指南
date: 2025-10-29 17:30:25
category: 后端
tags: Kafka
---

搭建 Kafka 开发环境（单机伪集群）
Kafka 依赖 ZooKeeper 进行元数据管理与集群协调。以下演示在单台 Linux 服务器上，部署一个包含三个 Broker 节点的伪集群。

1. 获取发行版

下载地址：http://kafka.apache.org/downloads

2. 准备运行目录

bash
tar -zxvf kafka_2.11-1.0.0.tgz
mv kafka_2.11-1.0.0 kafka-broker1
cp -r kafka-broker1 kafka-broker2
cp -r kafka-broker1 kafka-broker3

3. 配置并启动 ZooKeeper 集群
   分别编辑三个节点下的 config/zookeeper.properties，关键配置如下（以节点1为例）：

properties
dataDir=/tmp/zookeeper/node1 # 数据目录，各节点不同
clientPort=2181 # 客户端连接端口，节点2设为2182，节点3设为2183
server.1=localhost:2888:3888
server.2=localhost:4888:5888
server.3=localhost:6888:7888
在每个节点的 dataDir 目录下创建 myid 文件，内容分别为 1, 2, 3。

启动命令（后台运行）：

bash
cd kafka-broker1
bin/zookeeper-server-start.sh config/zookeeper.properties &

# 同理启动 broker2, broker3 的 ZooKeeper

注意：确保 /tmp/zookeeper/nodeX 目录存在且有写入权限。

4. 配置并启动 Kafka Broker 集群
   分别编辑 config/server.properties（以节点1为例）：

properties
broker.id=1 # 集群内唯一ID，节点2设为2，节点3设为3
listeners=PLAINTEXT://:9091 # 监听地址与端口，节点2用9092，节点3用9093
log.dirs=/tmp/kafka-logs-1 # 日志目录，各节点不同
zookeeper.connect=localhost:2181,localhost:2182,localhost:2183 # ZK集群地址
启动命令：

bash
cd kafka-broker1
bin/kafka-server-start.sh config/server.properties &

# 同理启动 broker2, broker3

5. 基础功能验证

创建主题：

bash
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic test-topic
生产消息（在 broker1 执行）：

bash
bin/kafka-console-producer.sh --broker-list localhost:9091 --topic test-topic
消费消息（在 broker2 执行）：

bash
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test-topic --from-beginning
Spring Boot 应用集成 Kafka

1. 引入 Spring for Apache Kafka 依赖
   在 pom.xml 中添加：

xml
<dependency>
<groupId>org.springframework.kafka</groupId>
<artifactId>spring-kafka</artifactId>
<version>2.8.0</version> <!-- 请使用当前稳定版本 -->
</dependency>

2. 配置连接与序列化参数
   在 application.yml 中配置：

yaml
spring:
kafka:
bootstrap-servers: localhost:9091,localhost:9092,localhost:9093
producer:
key-serializer: org.apache.kafka.common.serialization.StringSerializer
value-serializer: org.apache.kafka.common.serialization.StringSerializer

# 可选配置：重试次数、批处理大小等

consumer:
group-id: my-springboot-group # 消费者组ID
auto-offset-reset: earliest # 无偏移量时从开始读取
key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
value-deserializer: org.apache.kafka.common.serialization.StringDeserializer

3. 实现消息生产者
   利用 Spring 自动配置的 KafkaTemplate 发送消息：

java
@RestController
@RequestMapping("/msg")
public class MessageController {

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    @PostMapping("/send")
    public String sendMessage(@RequestParam String topic,
                              @RequestParam String key,
                              @RequestParam String message) {
        kafkaTemplate.send(topic, key, message);
        // send() 方法返回一个 ListenableFuture，可用于异步处理发送结果
        return "消息已发送至主题：" + topic;
    }

}

4. 实现消息消费者
   使用 @KafkaListener 注解方便地声明消费者方法：

java
@Component
@Slf4j
public class KafkaMessageConsumer {

    @KafkaListener(topics = "test-topic", groupId = "my-springboot-group")
    public void listen(String message) {
        log.info("接收到消息：{}", message);
        // 在此处进行业务逻辑处理
    }

    // 可以监听多个主题，或通过 SpEL 表达式动态指定主题
    @KafkaListener(topics = {"topic-a", "topic-b"})
    public void listenMultipleTopics(ConsumerRecord<String, String> record) {
        log.info("主题[{}], 分区[{}], 收到键值对：{} -> {}",
                record.topic(), record.partition(), record.key(), record.value());
    }

}

5. 高级特性与最佳实践

事务支持：通过配置 KafkaTransactionManager 支持生产端事务。

消息确认模式：配置 ack 模式（0，1，all）以平衡性能与可靠性。

消费组管理：利用消费者组的自动分区重平衡特性，实现高可用与水平扩展。

错误处理：通过配置 ErrorHandler 或使用 @KafkaListener 的 errorHandler 属性处理消费异常。

参考资料

Spring Boot Kafka 支持：https://docs.spring.io/spring-boot/docs/current/reference/html/messaging.html#messaging.kafka

Spring for Apache Kafka 官方文档：https://docs.spring.io/spring-kafka/reference/html/