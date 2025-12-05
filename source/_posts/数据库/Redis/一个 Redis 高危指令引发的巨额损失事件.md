---
title: 一个 Redis 高危指令引发的巨额损失事件
date: 2025-10-30 14:42:34
category: 数据库
tags: Redis
---

近期技术安全事故频发，今日又见报道：某公司 PHP 工程师因在线执行 Redis 危险指令，导致企业直接经济损失高达 400 万元。

究竟是什么 Redis 指令具备如此破坏力，能造成如此重大的损失？

事件简要情况如下：

据媒体报道，某公司技术部门连续发生两起本年度重大事故，累计造成公司资金损失 400 万元，事故原因如下：

由于 PHP 工程师直接在生产环境操作 Redis，执行了 keys * wxdb（此处省略）cf8* 这类指令，导致 Redis 服务被锁死，CPU
使用率急剧飙升，进而使得所有支付链路卡顿阻塞。十几秒后，当服务恢复时，积压的请求流量瞬间涌向关系型数据库，引发数据库雪崩效应，最终导致数据库宕机。

该公司已严肃声明，若再发生类似事故，将直接予以开除，并表示后续将逐步回收运维部门的各项操作权限。

看完这则消息，笔者深感震惊：为何如此基础的安全问题仍在发生？为何生产环境中的高危指令未被有效禁用？这类事件被公开报道，确实显得十分低级。

不论涉事公司规模大小，发生此类事故都极不应该，相关责任人员理应引咎反省！

稍有 Redis 使用经验的人员都清楚，在生产环境中绝对不能执行 keys * 及相关模糊匹配指令。虽然这类指令在数据量小的时候使用方便、功能强大，但一旦数据量过大，就会导致
Redis 阻塞和 CPU 急剧升高，严重影响服务稳定性。因此，在生产环境中强烈建议禁用或重命名此类高危指令。

还有哪些 Redis 危险指令？
除了 keys 之外，Redis 中还存在其他几个需要警惕的高危指令：

flushdb
删除当前所选数据库中的所有键。该指令执行后不会返回失败。

flushall
删除 Redis 中所有数据库的所有键，而不仅仅是当前数据库。该指令同样不会返回失败。

config
允许客户端直接修改 Redis 的运行时配置，可能引发配置错乱或安全风险。

如何禁用或重命名危险指令？
在 Redis 的默认配置文件 redis.conf 中，找到 SECURITY 安全配置区域，可以看到相关说明：

```text
################################## SECURITY ###################################

# Command renaming.

#

# It is possible to change the name of dangerous commands in a shared

# environment. For instance the CONFIG command may be renamed into something

# hard to guess so that it will still be available for internal-use tools

# but not available for general clients.
```

通过添加 rename-command 配置项，即可实现指令的安全管控。

1）彻底禁用指令
将指令重命名为空字符串即可禁用：

```text
rename-command KEYS     ""
rename-command FLUSHALL ""
rename-command FLUSHDB  ""
rename-command CONFIG   ""
```
2）重命名为不易猜测的指令
将指令重命名为随机字符串或自定义名称：

```text
rename-command KEYS     "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
rename-command FLUSHALL "YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY"
rename-command FLUSHDB  "ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ"
rename-command CONFIG   "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"
```
其中 X、Y、Z、A 可替换为自定义的新指令名称或随机字符。

通过以上配置，可有效避免因误操作或恶意执行高危指令而导致的生产事故。