---
title: 负载均衡常用算法梳理与Nginx策略详解
date: 2025-10-30 14:42:34
category: 后端
tags: 算法
---

负载均衡常用算法
1、轮询调度法
请求依次分配给后端各服务器，实现简单，但不考虑服务器实际负载情况。

2、随机分配法
通过随机数生成器选择一台后端服务器处理请求。长期来看，请求量会趋于均匀分布，效果接近轮询。

3、源地址哈希法
根据客户端IP计算哈希值，再对服务器数量取模，保证同一IP的请求始终落在同一台服务器上，适用于需要会话保持的场景。

4、加权轮询法
根据服务器性能与负载分配不同权重，性能高的服务器获得更多请求，实现负载的动态合理分配。

5、加权随机法
在加权的基础上引入随机选择机制，避免顺序分配可能带来的负载不均衡。

6、最小连接数法
动态监控各服务器当前连接数，将新请求分配给连接数最少的服务器，提升整体资源利用率。

Nginx负载均衡策略
1、轮询（默认）
按时间顺序逐一分配请求，支持自动剔除故障节点。

2、权重分配（weight）
通过weight参数设置服务器优先级，权重越高，接收的请求比例越大。

text
upstream backend {
server 192.168.0.14 weight=10;
server 192.168.0.15 weight=10;
}
3、IP哈希（ip_hash）
根据客户端IP的哈希结果固定分配服务器，适用于需要保持会话一致性的场景。

text
upstream backend {
ip_hash;
server 192.168.0.14:88;
server 192.168.0.15:80;
}
4、响应时间优先（fair）
按服务器响应时间动态分配请求，响应越快的服务器优先获得流量。

text
upstream backend {
server server1;
server server2;
fair;
}
5、URL哈希（url_hash）
根据请求URL的哈希值分配服务器，适用于后端为缓存服务的场景。

text
upstream backend {
server squid1:3128;
server squid2:3128;
hash $request_uri;
hash_method crc32;
}
附加配置说明

down：标记服务器暂时不参与负载

weight：权重值，默认1

max_fails：允许失败次数

fail_timeout：失败后暂停时间

backup：备用服务器，在主服务器不可用时启用