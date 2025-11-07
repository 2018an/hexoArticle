---
title: Minikube安装教程
date: 2025-11-7 18:30:25
category: 后端
tags: 综合技术
---
**阶段一：系统准备（先决条件）**

**启用虚拟化 (BIOS/UEFI 设置)**

重启电脑，进入 BIOS/UEFI 设置界面（通常在启动时按 `**F2**`, `**F10**`, `**Del**` 等键）。

找到关于 **Virtualization Technology (VT-x)**、**Intel Virtualization Technology** 或 **SVM Mode** (AMD CPU) 的选项，将其设置为 **Enabled**。

保存并退出。这是运行虚拟机的基础，必须开启。

**启用 Windows Hypervisor 平台 (可选，但推荐)**

在 Windows 搜索框中输入“启用或关闭 Windows 功能”，并打开它。

在弹出窗口中，找到并勾选 **【Windows Hypervisor 平台】**。

如果打算使用 Hyper-V，也请勾选 **【Hyper-V】**（请注意，Hyper-V 仅在 Windows 10/11 专业版、企业版或教育版中可用）。

点击“确定”，等待安装完成，并根据提示**重启计算机**。

**安装容器/虚拟机驱动（二选一）**  
Minikube 需要一个驱动来创建虚拟机。**推荐使用 Docker，因为它最简单且性能良好。**

**选项 A (推荐): 安装 Docker Desktop**

1.  访问 [Docker Desktop for Windows 官网](https://docs.docker.com/desktop/install/windows-install/ "Docker Desktop for Windows 官网")。
2.  下载并运行安装程序。
3.  安装完成后，启动 Docker Desktop。它会指导你完成初始设置。
4.  安装成功后，系统托盘会出现一个鲸鱼图标。

**选项 B: 安装 VirtualBox**

1.  访问 [VirtualBox 官网](https://www.virtualbox.org/wiki/Downloads "VirtualBox 官网")。
2.  下载 **Windows hosts** 的安装程序。
3.  运行安装程序，所有选项保持默认即可。

**阶段二：安装 Minikube 和 Kubectl**

**使用 Winget 安装 Minikube**

在开始菜单中搜索 **“PowerShell”** 或 **“命令提示符”**。

**右键点击**它，选择 **“以管理员身份运行”**。

在打开的窗口中，输入以下命令并回车：

```
`powershellwinget install minikube`

AI写代码




```

Winget 会自动处理下载和安装过程，包括安装 Kubernetes 的命令行工具 `**kubectl**`。等待出现“已成功安装”的提示。

**验证安装**

关闭刚才的管理员窗口，**重新打开一个普通的 PowerShell 或命令提示符窗口**（无需管理员权限）。这一步是为了刷新系统环境变量。

输入以下命令检查 Minikube 版本：

```
`powershellminikube version`

AI写代码




```

输入以下命令检查 `**kubectl**` 是否也已安装：

```
`powershellkubectl version --client`

AI写代码




```

如果两个命令都返回了版本号，说明安装成功。