---
title: 从零构建 Redis Cluster 官方集群：详尽步骤与问题解决
date: 2025-10-30 14:42:34
category: 数据库
tags: Redis
---

本文将完整演示如何从零开始搭建 Redis Cluster 官方集群，记录搭建过程中的关键步骤与常见问题的解决方案。

准备 Ruby 运行环境
由于 Redis 官方提供的集群管理工具 redis-trib.rb 是用 Ruby 编写的，因此需要先安装 Ruby 2.2.2 及以上版本，并且需要指定
OpenSSL 支持。

安装 OpenSSL

```text
$ wget https://www.openssl.org/source/openssl-1.0.2m.tar.gz
$ tar -zxvf openssl-1.0.2m.tar.gz
$ cd openssl-1.0.2m
$ ./config --prefix=/usr/local/openssl
$ ./config -t
$ make
$ make install
$ openssl version
```
安装 Ruby

```text
$ yum remove ruby # 移除旧版本
$ wget https://cache.ruby-lang.org/pub/ruby/2.4/ruby-2.4.2.tar.gz
$ tar -zxvf ruby-2.4.2.tar.gz
$ cd ruby-2.4.2
$ ./configure --with-openssl-dir=/usr/local/openssl
$ make
$ make install
$ sudo ln -s /usr/local/bin/ruby /usr/bin/ruby
```
安装 RubyGems

```text
$ wget https://rubygems.org/rubygems/rubygems-2.3.0.tgz
$ tar -zxvf rubygems-2.3.0.tgz
$ cd rubygems-2.3.0
$ ruby setup.rb
```
安装并配置 Zlib
编辑 Ruby 源码中的 Zlib 编译配置：

```text
$ vi /ruby-2.4.2/ext/zlib/Makefile
```
将 zlib.o: $(top_srcdir)/include/ruby.h 修改为 zlib.o: ../../include/ruby.h，然后执行：

```text
$ yum install zlib*
$ cd /ruby-2.4.2/ext/zlib
$ ruby extconf.rb
$ make
$ make install
```
安装 Redis Ruby 库
```text
$ gem install redis
```
若出现 Unable to require openssl 错误，需安装 openssl-devel 并重新编译 Ruby。

搭建 Redis Cluster 集群
创建集群目录结构
进入一个新目录，创建六个以端口号命名的子目录，用于存放不同节点的配置与数据。

```text
$ mkdir redis-cluster
$ cd redis-cluster
$ mkdir 9001 9002 9003 9004 9005 9006
```
准备节点配置文件
在每个端口目录（9001~9006）下创建 redis.conf 文件，内容示例如下（端口需对应修改）：

```text
port 9001
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
cluster-enabled：启用集群模式。

cluster-config-file：指定集群节点配置文件路径，由 Redis 自动维护。

cluster-node-timeout：节点超时时间。
```

复制 Redis 服务程序
将编译好的 redis-server 可执行文件复制到 redis-cluster 目录下。

启动所有集群节点
分别进入每个端口目录，启动对应的 Redis 实例：

```text
$ cd 9001
$ ../redis-server ./redis.conf
```
对其他五个目录重复此操作。

创建集群
使用 redis-trib.rb 工具，将六个独立的实例组建成一个三主三从的集群：

```text
$ ./redis-trib.rb create --replicas 1 127.0.0.1:9001 127.0.0.1:9002 127.0.0.1:9003 127.0.0.1:9004 127.0.0.1:9005
127.0.0.1:9006
```
命令执行后，工具会给出一个分配方案（包括哪三个节点是主节点，哪三个是从节点，以及槽位分配），输入 yes 确认即可。成功后会输出 All
16384 slots covered.，表示所有哈希槽均已分配，集群运行正常。

查看集群节点状态
连接任意一个节点，查看集群节点信息：

```text
$ redis-cli -c -p 9001
127.0.0.1:9001> cluster nodes
```
连接并使用集群
使用 -c 参数以集群模式连接客户端，这样当操作的 key 不在当前节点时，客户端会自动重定向到正确的节点。

```text
$ ./redis-cli -c -h 192.168.1.8 -p 9002 -a yourpassword
```
解决主从复制不同步问题
若搭建主从集群后发现从节点无法同步主节点数据，很可能是因为主节点设置了密码，而从节点配置中未指定主节点的认证密码。

解决方案：
停止所有从节点，在各自的 redis.conf 配置文件中添加以下配置（密码需与主节点设置的 requirepass 一致）：

```text
masterauth yourpassword
```
添加后重启从节点，主从复制即可恢复正常。