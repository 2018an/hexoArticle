---
title: Redis 核心操作指令详解，非常详细！
date: 2025-10-30 14:42:34
category: 数据库
tags: Redis
---

本文系统性地整理和演示了 Redis 在日常使用中的高频命令，涵盖服务管理、键操作、字符串、集合、列表和哈希等多种数据类型。

更全面的 Redis 资源可在后台回复关键词获取。

服务管理相关命令

1. 启动 Redis 服务
```text

> redis-server [--port 6379]
```
> 若启动参数较多，推荐通过配置文件启动：

```text
> redis-server [配置文件路径]
```
> 6379 是 Redis 的标准默认端口。

2. 连接至 Redis 服务
```text
> ./redis-cli [-h 主机地址 -p 端口]
```

3. 停止 Redis 服务
```text
> redis-cli shutdown

> kill Redis进程PID
```
> 两种方式效果一致。

4. 发送指令的两种方式
   带参数直接执行：

```text
> redis-cli shutdown
```
> 进入交互模式后执行：

```text
> ./redis-cli
> 127.0.0.1:6379> shutdown
```

5. 测试连接状态
```text
   127.0.0.1:6379> ping
```
   PONG
   键（Key）操作命令
   获取所有键名
   语法：keys pattern

```text
127.0.0.1:6379> keys *
```

* 为通配符，会返回所有键名。该操作时间复杂度为 O(n)，数据量大时慎用。

获取键总数
语法：dbsize

```text
127.0.0.1:6379> dbsize
```
此命令直接读取内部计数器，时间复杂度 O(1)。

检查键是否存在
语法：exists key [key ...]

```text
127.0.0.1:6379> exists key1 key2
```
支持同时检查多个键，返回存在的数量。

删除键
语法：del key [key ...]

```text
127.0.0.1:6379> del key1 key2
```
返回成功删除的键数量。

查询键的数据类型
语法：type key

```text
127.0.0.1:6379> type mykey
```
string
移动键至其他数据库
语法：move key db

```text
127.0.0.1:6379> move mykey 2
127.0.0.1:6379> select 2
```
查询键的剩余生存时间
秒级：ttl key
毫秒级：pttl key

```text
127.0.0.1:6379> ttl mykey
```
返回 -1 表示永不过期。

设置键的过期时间
秒级：expire key seconds
毫秒级：pexpire key milliseconds

```text
127.0.0.1:6379> expire mykey 60
```
移除键的过期时间
语法：persist key

```text
127.0.0.1:6379> persist mykey
```
重命名键
语法：rename key newkey

```text
127.0.0.1:6379> rename oldkey newkey
```
字符串（String）操作命令
字符串是 Redis 最基础的数据类型，单个值最大可存储 512MB。

设置键值对
语法：set key value [EX seconds] [PX milliseconds] [NX|XX]

```text
127.0.0.1:6379> set username "zhangsan"
```
NX 表示键不存在时才设置，XX 表示键存在时才更新。

获取键值
语法：get key

```text
127.0.0.1:6379> get username
```
值递增/递减
若值为数字，可使用 incr 递增，decr 递减：

```text
127.0.0.1:6379> incr counter
```
递增指定数值使用 incrby，浮点数用 incrbyfloat。

批量设置与获取
批量设置：mset key value [key value ...]
批量获取：mget key [key ...]

```text
127.0.0.1:6379> mset k1 v1 k2 v2
127.0.0.1:6379> mget k1 k2
```
获取字符串长度
语法：strlen key

```text
127.0.0.1:6379> strlen username
```
向字符串尾部追加内容
语法：append key value

```text
127.0.0.1:6379> append username "_append"
```
获取字符串子串
语法：getrange key start end

```text
127.0.0.1:6379> getrange username 0 4
```
集合（Set）操作命令
集合是无序且元素唯一的容器。

添加元素
语法：sadd key member [member ...]

text
127.0.0.1:6379> sadd languages java python go
获取所有元素
语法：smembers key

text
127.0.0.1:6379> smembers languages
随机获取元素
语法：srandmember key [count]

text
127.0.0.1:6379> srandmember languages 2
检查元素是否存在
语法：sismember key member

text
127.0.0.1:6379> sismember languages java
获取集合元素数量
语法：scard key

text
127.0.0.1:6379> scard languages
删除指定元素
语法：srem key member [member ...]

text
127.0.0.1:6379> srem languages go
随机弹出元素
语法：spop key [count]

text
127.0.0.1:6379> spop languages 1
有序集合（Sorted Set）操作命令
有序集合在集合基础上为每个元素关联一个分数（score），用于排序。

添加元素
语法：zadd key [NX|XX] [CH] [INCR] score member [score member ...]

text
127.0.0.1:6379> zadd rank 95 "Alice" 87 "Bob"
获取元素分数
语法：zscore key member

text
127.0.0.1:6379> zscore rank "Alice"
按排名范围获取
语法：zrange key start stop [WITHSCORES]

text
127.0.0.1:6379> zrange rank 0 -1 WITHSCORES
按分数范围获取
语法：zrangebyscore key min max [WITHSCORES] [LIMIT offset count]

text
127.0.0.1:6379> zrangebyscore rank 80 100
增加元素分数
语法：zincrby key increment member

text
127.0.0.1:6379> zincrby rank 5 "Alice"
获取集合元素总数
语法：zcard key

text
127.0.0.1:6379> zcard rank
统计分数区间内元素数量
语法：zcount key min max

text
127.0.0.1:6379> zcount rank 80 100
删除元素
语法：zrem key member [member ...]

text
127.0.0.1:6379> zrem rank "Bob"
获取元素排名
语法：zrank key member

text
127.0.0.1:6379> zrank rank "Alice"
列表（List）操作命令
列表是基于双向链表实现的有序元素集合，元素可重复，支持两端操作。

从左侧插入元素
语法：lpush key value [value ...]

text
127.0.0.1:6379> lpush mylist a b c
从右侧插入元素
语法：rpush key value [value ...]

text
127.0.0.1:6379> rpush mylist x y z
通过索引设置值
语法：lset key index value

text
127.0.0.1:6379> lset mylist 0 "newvalue"
从左侧弹出元素
语法：lpop key

text
127.0.0.1:6379> lpop mylist
从右侧弹出元素
语法：rpop key

text
127.0.0.1:6379> rpop mylist
获取列表长度
语法：llen key

text
127.0.0.1:6379> llen mylist
获取指定范围内的元素
语法：lrange key start stop

text
127.0.0.1:6379> lrange mylist 0 -1
通过索引获取元素
语法：lindex key index

text
127.0.0.1:6379> lindex mylist 1
删除指定值的元素
语法：lrem key count value

text
127.0.0.1:6379> lrem mylist 2 "value"
修剪列表
语法：ltrim key start stop

text
127.0.0.1:6379> ltrim mylist 1 3
哈希（Hash）操作命令
哈希类型用于存储字段-值映射，适合表示对象。

设置单个字段
语法：hset key field value

text
127.0.0.1:6379> hset user:1 name "LiLei"
批量设置字段
语法：hmset key field value [field value ...]

text
127.0.0.1:6379> hmset user:1 age 20 city "Beijing"
仅当字段不存在时设置
语法：hsetnx key field value

text
127.0.0.1:6379> hsetnx user:1 email "test@example.com"
获取单个字段值
语法：hget key field

text
127.0.0.1:6379> hget user:1 name
批量获取字段值
语法：hmget key field [field ...]

text
127.0.0.1:6379> hmget user:1 name age
获取所有字段和值
语法：hgetall key

text
127.0.0.1:6379> hgetall user:1
获取所有字段名
语法：hkeys key

text
127.0.0.1:6379> hkeys user:1
获取所有字段值
语法：hvals key

text
127.0.0.1:6379> hvals user:1
检查字段是否存在
语法：hexists key field

text
127.0.0.1:6379> hexists user:1 name
获取字段数量
语法：hlen key

text
127.0.0.1:6379> hlen user:1
递增字段值
语法：hincrby key field increment

text
127.0.0.1:6379> hincrby user:1 age 1
删除字段
语法：hdel key field [field ...]

text
127.0.0.1:6379> hdel user:1 city
以上为 Redis 常用核心命令的整理与演示，掌握这些指令可应对绝大多数开发场景。