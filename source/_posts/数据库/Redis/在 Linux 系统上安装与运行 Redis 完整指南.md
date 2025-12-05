---
title: 在 Linux 系统上安装与运行 Redis 完整指南
date: 2025-10-30 14:42:34
category: 数据库
tags: Redis
---

下载 Redis 安装包
前往 Redis 官方网站下载最新的 Linux 版本安装包。请注意，Redis 官方不提供 Windows 版本。

官网下载地址：https://redis.io/

下载完成后，将安装包上传至 Linux 服务器。

安装 Redis
解压安装包

```text
> tar -zxvf redis-4.0.2.tar.gz
```
> 进入解压目录

```text
> cd redis-4.0.2
```
> 编译 Redis

```text
> make
```
> 如遇编译错误，可参考以下解决方案：

错误提示：make: cc：命令未找到 或 make: *** [adlist.o] 错误 127
解决方案：安装 gcc 编译器

```text
> yum install gcc
```
> 错误提示：collect2: ld returned 1 exit status 等链接错误
> 解决方案：编辑 src/.make-settings 文件，修改 OPT=-O2 -march=x86-64

执行编译测试

```text
> make test
```
> 若测试报错提示缺少 tcl，可按以下步骤安装：

```text
> wget http://downloads.sourceforge.net/tcl/tcl8.6.7-src.tar.gz
> tar -zxvf tcl8.6.7-src.tar.gz
> cd tcl8.6.7/unix/
> ./configure
> make
> make install
```
> 安装 Redis
> 返回 Redis 源码目录执行安装：

```text
> make install
```
> 启动 Redis 服务
> 进入 Redis 的 src 目录，通过指定配置文件来启动服务：

```text
> ./redis-server ../redis.conf
```
> 若启动成功，将看到类似如下的输出信息：

```text
_._                                                  
_.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 4.0.2 (00000000/0) 64 bit
.-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 6651
  `-._    `-._  `-./  _.-'    _.-'                                   
|`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
|`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
`-._    `-.__.-'    _.-'                                       
`-._        _.-'                                           
    `-.__.-'                                               
```
启动过程中出现的若干警告（如 maxclients 受限、overcommit_memory 设置等）可根据提示进行系统级优化，但在测试环境中通常可暂时忽略。

连接 Redis
启动服务后，可通过以下方式连接：

使用命令行客户端
在 src 目录下执行：

```text
> ./redis-cli
```
> 连接成功后提示符将变为：127.0.0.1:6379>

使用图形化客户端工具
推荐使用 Redis Desktop Manager，它提供免费社区版和付费专业版。

下载地址：https://redisdesktop.com/download

至此，Redis 已在 Linux 系统上成功安装并运行。