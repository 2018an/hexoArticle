---
title: Windows 系统下 Minikube 本地 Kubernetes 环境部署指南
date: 2025-11-7 18:30:25
category: 后端
tags: 综合技术
---

Minikube 是一款用于在本地快速搭建单节点 Kubernetes 集群的工具，非常适合开发、测试和学习。以下是在 Windows 10/11 系统上安装和配置
Minikube 的完整步骤。

第一阶段：环境准备与前置条件

1. 开启 CPU 虚拟化支持
   这是运行虚拟机的前提。重启电脑，进入 BIOS/UEFI 设置（启动时按 F2、F10、Del 等键），找到 Virtualization Technology (Intel
   VT-x) 或 SVM Mode (AMD)，将其设置为 Enabled。保存后退出。

2. 启用 Windows 相关虚拟化功能

在 Windows 搜索栏输入 “启用或关闭 Windows 功能” 并打开。

在弹出的窗口中，找到并勾选 【Windows Hypervisor 平台】。

（可选）如果你使用的是 Windows 专业版/企业版/教育版，也可以勾选 【Hyper-V】。注意，Hyper-V 与某些虚拟机软件（如 VirtualBox）不兼容。

点击“确定”，等待安装完成并按照提示重启电脑。

3. 安装虚拟机或容器驱动（二选一）
   Minikube 需要一个底层驱动来创建运行 Kubernetes 的虚拟机。

推荐方案：安装 Docker Desktop

访问 Docker 官网下载 Docker Desktop for Windows。

运行安装程序，通常使用默认设置即可。

安装完成后启动 Docker Desktop，并根据指引完成初始化。成功后在系统托盘会显示 Docker 图标。Minikube 会自动使用其内置的 docker
驱动，最为简便。

备选方案：安装 VirtualBox

访问 VirtualBox 官网，下载 Windows 版本的安装程序。

运行安装程序，保持默认选项。安装完成后，需在 Minikube 启动时通过 --driver=virtualbox 指定使用此驱动。

第二阶段：安装 Minikube 与 kubectl

1. 使用 Windows 包管理器 Winget 安装
   这是目前 Windows 上最便捷的安装方式。

在开始菜单搜索 “PowerShell”，右键点击并选择 “以管理员身份运行”。

在打开的管理员 PowerShell 窗口中，执行以下命令：

powershell
winget install minikube
Winget 会自动下载并安装 Minikube，同时会一并安装 Kubernetes 命令行工具 kubectl。等待命令执行完成，出现成功提示。

2. 验证安装

关闭刚才的管理员 PowerShell 窗口。

重新打开一个普通的 PowerShell 或命令提示符窗口（此步骤很重要，用于刷新系统环境变量）。

输入以下命令检查安装是否成功：

powershell
minikube version
该命令应输出 Minikube 的版本信息。

powershell
kubectl version --client
该命令应输出 kubectl 客户端的版本信息。

第三阶段：启动你的第一个集群
在普通 PowerShell 窗口中，运行以下命令启动 Minikube 集群：

powershell
minikube start
如果安装了 Docker Desktop，Minikube 默认会使用 docker 驱动。

如果使用 VirtualBox，则需要使用：minikube start --driver=virtualbox

首次启动会下载所需的镜像，请耐心等待，直至看到类似 “Done! kubectl is now configured to use “minikube” cluster” 的提示。

第四阶段：基本操作与验证
查看集群状态：

powershell
minikube status
打开 Kubernetes 仪表盘（可选）：

powershell
minikube dashboard
此命令会自动在浏览器中打开 Kubernetes 的 Web 管理界面。

停止与删除集群（当不再需要时）：

powershell
minikube stop # 暂停集群
minikube delete # 彻底删除集群及其所有数据
至此，你已在 Windows 上成功搭建了一个本地 Kubernetes 学习与开发环境。