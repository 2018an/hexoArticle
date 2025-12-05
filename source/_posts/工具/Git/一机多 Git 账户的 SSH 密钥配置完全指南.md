---
title: 一机多 Git 账户的 SSH 密钥配置完全指南
date: 2025-10-30 14:42:34
category: 工具
tags: Git
---

在开发工作中，开发者常需同时维护多个 Git 托管账户（如公司的 GitLab、个人的 GitHub 等）。当均采用 SSH
协议连接时，默认的单密钥配置会导致冲突。通过以下步骤，可在一台机器上为不同账户配置独立的密钥。

1. 生成多对独立的 SSH 密钥
   使用 -f 参数为每个服务指定唯一的密钥文件名，避免相互覆盖。

为 GitHub 生成：

bash
ssh-keygen -t rsa -b 4096 -C "your-email@domain.com" -f ~/.ssh/github_id_rsa
为 GitLab 生成：

bash
ssh-keygen -t rsa -b 4096 -C "your-email@company.com" -f ~/.ssh/gitlab_id_rsa
其他仓库的配置方法与此类似。

2. 在托管平台添加对应公钥
   分别打开生成的 .pub 公钥文件（如 github_id_rsa.pub），将其内容完整复制，并粘贴到对应账户设置中的 “SSH Keys” 区域。

3. 创建 SSH 客户端配置文件
   在 ~/.ssh/ 目录下创建（或编辑）名为 config 的文本文件，为不同主机配置对应的密钥路径。

text

# 配置 GitHub

Host github.com
HostName github.com
IdentityFile ~/.ssh/github_id_rsa

# 配置 GitLab

Host gitlab.com
HostName gitlab.com
IdentityFile ~/.ssh/gitlab_id_rsa

# 更多配置可依此格式添加

4. 测试连接是否成功
   配置完成后，分别测试到各平台的连接：

bash
ssh -T git@github.com
ssh -T git@gitlab.com
当看到“认证成功”等相关提示信息时，即表示配置正确，现在您可以顺畅地在不同身份的仓库之间切换操作了。