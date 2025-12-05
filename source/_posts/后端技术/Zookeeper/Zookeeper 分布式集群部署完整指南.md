---
title: Zookeeper 分布式集群部署完整指南
date: 2025-10-29 17:30:25
category: 后端
tags: Zookeeper
---

Zookeeper 作为分布式协调服务，在生产环境中通常以集群方式部署以确保高可用性。本文将详细介绍如何搭建一个稳定的 Zookeeper
集群环境。

#### 获取安装文件

> 官方网站：http://zookeeper.apache.org/

从官网下载最新稳定版本（本文以 `zookeeper-3.4.11` 为例）。

#### 基础安装步骤

**1、上传安装文件**
将下载的压缩包（如：zookeeper-3.4.11.tar.gz）上传至目标服务器。

**2、解压文件**

```bash
$ tar zxvf zookeeper-3.4.11.tar.gz
```

**3、规范安装路径**

```bash
$ mv zookeeper-3.4.11 /usr/local/zookeeper
```

#### 集群环境搭建

根据分布式系统原理，Zookeeper 集群需要至少 3 个节点（2n+1 原则）才能形成有效的多数决策机制。以下演示三节点集群配置方法，多节点配置原理相同。

**1、创建数据存储目录**

```bash
$ cd /usr/local/zookeeper
$ mkdir data
```

**2、准备配置文件**

```bash
$ cd conf
$ cp zoo_sample.cfg zoo.cfg
```

**3、编辑核心配置**

```bash
$ vi zoo.cfg
```

修改以下关键配置项：

```properties
# 指定数据持久化目录
dataDir=/usr/local/zookeeper/data
# 定义集群节点列表
# 格式：server.序号=IP地址:数据同步端口:选举通信端口
server.1=192.168.10.31:2888:3888
server.2=192.168.10.32:2888:3888
server.3=192.168.10.33:2888:3888
```

**4、配置节点标识**

```bash
$ cd ../data
$ echo "1" > myid
```

每个节点的 `myid` 文件内容必须与配置文件中 `server.x` 的序号严格对应。

**5、配置防火墙规则**
开放集群通信所需端口：

```bash
# 客户端连接端口
$ sudo firewall-cmd --permanent --add-port=2181/tcp

# 节点间数据同步端口
$ sudo firewall-cmd --permanent --add-port=2888/tcp

# 节点间选举通信端口
$ sudo firewall-cmd --permanent --add-port=3888/tcp

$ sudo firewall-cmd --reload
```

**6、同步配置至其他节点**
将配置好的目录复制到集群其他机器：

```bash
$ scp -r /usr/local/zookeeper user@192.168.10.32:/usr/local/
```

在其他节点重复步骤4-5，注意修改 `myid` 文件中的数字。

**7、启动集群服务**
在所有节点执行启动命令：

```bash
$ /usr/local/zookeeper/bin/zkServer.sh start
```

**8、验证集群状态**
查看各节点角色状态：

```bash
$ /usr/local/zookeeper/bin/zkServer.sh status
```

正常输出应显示 `leader`（主节点）或 `follower`（从节点）。

#### 客户端连接测试

连接到指定集群节点：

```bash
$ ./zkCli.sh -server 192.168.10.31:2181
```

若连接本机节点，可省略 `-server` 参数。

#### 重要注意事项

1. **端口配置**：确保所有节点的端口配置一致且未被占用
2. **时间同步**：集群所有节点需保持时间同步（建议配置NTP服务）
3. **资源分配**：根据数据量合理分配磁盘空间和内存资源
4. **伪集群部署**：单机部署多个实例时，需为每个实例配置独立的数据目录、日志文件和通信端口

通过以上步骤，即可完成一个高可用的 Zookeeper 集群部署。集群搭建后，建议通过创建测试节点、读写数据等方式进一步验证集群功能完整性。