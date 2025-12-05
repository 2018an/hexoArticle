---
title: 在 Windows 上安装与配置 MongoDB
date: 2025-10-30 14:42:34
category: 数据库
tags: MongoDB
---

获取安装包
前往 MongoDB 官方网站下载适用于 Windows 的安装包：

https://www.mongodb.com/download-center

为方便测试，此处选择 Windows 64 位企业版（评估试用版）。虽然官方要求操作系统为 Windows Server 2008 R2 或更高版本，但在
Windows 7 环境下通常也可正常运行。

如需了解企业版的高级特性，请参考：MongoDB Enterprise Advanced

配置 MongoDB 服务
以下以配置为 Windows 服务为例，实现开机自启动。

1. 配置环境变量
   将 MongoDB 的 bin 目录路径（例如 C:\MongoDB\Server\3.4\bin）添加到系统的 PATH 环境变量中。

2. 创建数据与日志目录
   在合适位置创建以下目录，用于存储数据库文件和日志：

```text
C:\MongoDB\data\db
C:\MongoDB\data\log
```

3. 创建配置文件
   在 MongoDB 安装目录下（例如 C:\MongoDB\Server\3.4\）创建配置文件 mongod.cfg，内容如下：

yaml
systemLog:
destination: file
path: C:\MongoDB\data\log\mongod.log
storage:
dbPath: C:\MongoDB\data\db

4. 安装 Windows 服务
   以管理员身份打开命令提示符，执行以下命令，通过指定配置文件将 MongoDB 安装为系统服务：

```text
mongod --config C:\MongoDB\Server\3.4\mongod.cfg --install
```
服务管理命令
启动 MongoDB 服务：

```text
net start mongodb
```
停止 MongoDB 服务：

```text
net stop mongodb
```
连接 MongoDB
安装并启动服务后，可以通过以下方式连接：

命令行客户端：在命令提示符中输入 mongo 命令。

图形化客户端工具：推荐使用 Robo 3T（原 Robomongo），它提供免费社区版与付费专业版。

下载地址：https://robomongo.org/