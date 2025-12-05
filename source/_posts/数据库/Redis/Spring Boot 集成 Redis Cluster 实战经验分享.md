---
title: Spring Boot 集成 Redis Cluster 实战经验分享
date: 2025-10-30 14:42:34
category: 数据库
tags: Redis
---

添加 Spring Boot 配置
在 application.yml 中添加以下 Redis Cluster 配置：

yaml
spring.redis:
database: 0
password: 123456
timeout: 10000
pool:
max-active: 8
max-idle: 8
max-wait: -1
min-idle: 0
cluster:
nodes:

- 192.168.1.8:9001
- 192.168.1.8:9002
- 192.168.1.8:9003
  注意：配置中只需列出主（Master）节点地址，无需配置从（Slave）节点。Spring Boot 会自动完成其余配置。

配置完成后，即可像使用单节点 Redis 一样操作集群，框架会自动根据 key 进行分片。

遇到的典型问题与解决
在集成过程中，可能会遇到如下连接异常：

```text
Caused by: redis.clients.jedis.exceptions.JedisConnectionException: Could not get a resource from the pool
Caused by: java.net.ConnectException: Connection refused: connect
```
跟踪异常栈发现，客户端尝试连接的地址是 127.0.0.1，而非配置中指定的 192.168.1.8。

问题根源：
当 Spring Boot 客户端连接集群时，会向配置的节点请求集群拓扑信息。而集群节点返回的地址信息，默认是其自身绑定的地址。如果节点配置文件
redis.conf 中未明确设置 bind 参数，或绑定的是 127.0.0.1，则客户端会拿到这个回环地址，导致无法连接。

解决方案：
修改每个 Redis 集群节点（包括主节点和从节点）的 redis.conf 配置文件，添加或修改 bind 参数，指定为客户端可访问的 IP 地址（如服务器内网
IP）：

```text
bind 192.168.1.8
```
修改后，重启所有 Redis 集群节点，Spring Boot 应用即可正常连接并进行读写操作。