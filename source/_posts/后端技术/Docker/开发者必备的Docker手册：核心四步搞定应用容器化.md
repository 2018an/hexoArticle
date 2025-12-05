---
title: 开发者必备的Docker手册：核心四步搞定应用容器化
date: 2025-10-29 17:30:25
category: 后端
tags: docker
---

### 一、Docker技术概述

- Docker是一款开源的轻量级容器化引擎技术
- 基于Go语言开发，遵循Apache2.0开源协议
- 开发者可以将应用程序及其所有依赖项打包到标准化的轻量级容器中，实现跨平台部署
- 容器采用沙箱隔离机制，彼此之间完全独立，互不干扰
- 与传统虚拟机技术（如VMware、VirtualBox）相比，Docker直接运行在宿主操作系统内核上，性能损耗极低，启动速度更快

**通俗易懂的解释：**

> Docker允许将软件及其运行环境打包成一个标准化的镜像文件，其他用户可以直接使用这个镜像来运行应用。
> 运行时的镜像实例称为容器，容器启动速度极快，类似于Windows系统中的Ghost系统镜像，开箱即用。

### 二、Docker架构核心组件

- Docker镜像：创建容器的模板文件，包含应用程序运行所需的所有内容
- Docker容器：镜像的运行实例，代表一个独立运行的应用程序环境
- Docker客户端：用户通过命令行工具或API与Docker守护进程交互的接口
- Docker主机：运行Docker守护进程和容器的物理或虚拟服务器
- Docker仓库：集中存储和分发Docker镜像的场所，类似代码仓库。Docker Hub作为官方仓库，提供了海量的镜像资源

### 三、Docker环境部署与管理

#### 1. 系统环境检查

> Docker要求CentOS系统的内核版本不低于3.10

执行以下命令检查当前系统版本：
uname -r

text

如版本不符合要求，需要先升级系统

#### 2. 系统更新（可选）

yum update

text

#### 3. Docker安装

yum install docker

text

#### 4. 启动Docker服务

systemctl start docker

text

#### 5. 设置开机自启

systemtctl enable docker

text

#### 6. 停止Docker服务

systemtctl stop docker

text

### 四、Docker实战操作指南

#### 4.1 镜像管理操作

> Docker官方镜像仓库Docker Hub汇集了丰富的镜像资源，[访问官网](https://hub.docker.com/explore/)

##### 4.1.1 搜索镜像

除了在[Docker Hub](https://hub.docker.com/explore/)网站搜索外，还可以使用命令行搜索，以MySQL为例：
docker search mysql

text

搜索结果示例：

![](https://upload-images.jianshu.io/upload_images/1900599-b0939b50cc4738ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 4.1.2 下载镜像

下载命令格式：`docker pull 镜像名:版本标签`，`tag`参数可选，默认为`latest`最新版
docker pull mysql

text

##### 4.1.3 查看镜像列表

获取本地已下载的镜像列表：
docker images

text

![](https://upload-images.jianshu.io/upload_images/1900599-c339bb96f54cbabc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

各列含义说明：

`REPOSITORY`：镜像仓库名称
`TAG`：镜像版本标签，`latest`表示最新版本
`IMAGE_ID`：镜像唯一标识符
`CREATED`：镜像创建时间
`SIZE`：镜像文件大小

##### 4.1.4 删除镜像

删除指定镜像：
docker rmi image-id

text

清理所有镜像：
docker rmi $(docker images -q)

text

#### 4.2 容器生命周期管理

> 类比软件使用流程：下载安装包→安装软件→运行程序
> 以Tomcat为例演示完整流程

##### 4.2.1 搜索镜像

docker search tomcat

text

##### 4.2.2 获取镜像

docker pull tomcat

text

##### 4.2.3 启动容器

基础容器启动命令：
docker run --name container-name -d image-name

text

参数说明：
`--name`：为容器指定标识名称
`-d`：后台运行模式，命令执行后立即返回控制台
`image-name`：要运行的镜像名称

##### 4.2.4 查看运行状态

查看当前正在运行的容器：
docker ps

text

![](https://upload-images.jianshu.io/upload_images/1900599-f620ae32f4356669.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

状态信息说明：

`CONTAINER ID`：容器唯一标识ID
`IMAGE`：容器基于的镜像
`COMMAND`：容器启动时执行的命令
`CREATED`：容器创建时间
`STATUS`：当前运行状态
`PORTS`：容器监听的端口信息
`NAMES`：容器名称

##### 4.2.5 停止容器

停止运行中的容器：
docker stop container-name/container-id

text

##### 4.2.6 查看所有容器

查看包括已停止的所有容器：
docker ps -a

text

##### 4.2.7 启动已停止容器

重新启动容器：
docker start container-name/container-id

text

##### 4.2.8 删除容器

删除单个容器：
docker rm container-id

text

批量删除所有容器：
docker rm $(docker ps -a -q )

text

##### 4.2.9 端口映射配置

默认情况下，容器内部的服务无法从外部网络直接访问。需要通过端口映射机制将容器端口暴露给外部网络。

Docker使用`-p`参数实现端口映射：
docker run --name tomcat1 -d tomcat
docker run --name tomcat2 -d -p 8888:8080 tomcat

text

上述命令将主机端口8888映射到容器内部端口8080。

执行后通过`docker ps`查看：

![](https://upload-images.jianshu.io/upload_images/1900599-86662f6e11bef5e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从`PORTS`列可以看出，`tomcat2`配置了端口映射，而`tomcat1`没有。

访问测试：
`http://服务器IP:8080/` // tomcat1，无法访问
`http://服务器IP:8888/` // tomcat2，可正常访问

端口映射语法格式：
ip:主机端口:容器端口 # 指定IP和端口映射
ip::容器端口 # 指定IP，主机端口自动分配
主机端口:容器端口 # 所有IP的主机端口映射到容器端口

text

##### 4.2.10 查看容器日志

查看容器运行日志：
docker logs container-id/container-name

text

##### 4.2.11 查看端口映射

查看容器的端口映射配置：
docker port container-id

text

示例输出：
[root@docker ~]#docker port 46114af6b44e
8080/tcp -> 0.0.0.0:8888
[root@docker ~]#docker port cea668ee4db0

text
空输出表示未配置端口映射。

##### 4.2.12 容器交互操作

运行中的容器是一个完整的Linux环境，可以像普通系统一样登录操作。

进入容器：
docker exec -it container-id/container-name bash

text

退出容器：
exit

text

##### 4.2.13 扩展命令参考

> 官方完整命令文档参考：
> https://docs.docker.com/engine/reference/commandline/docker/
