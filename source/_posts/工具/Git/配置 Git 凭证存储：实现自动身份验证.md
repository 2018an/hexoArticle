---
title: 配置 Git 凭证存储：实现自动身份验证
date: 2025-10-30 14:42:34
category: 工具
tags: Git
---

每次执行 Git 操作时都需输入用户名和密码，无疑会拖慢开发节奏。幸运的是，Git 提供了灵活的凭证缓存与存储机制，可有效解决此问题。

为 HTTPS 协议配置凭证存储
设置永久存储
运行以下命令，可将您的凭证（用户名和密码）永久保存在磁盘上：

bash
git config --global credential.helper store
配置成功后，Git 会将凭证以明文形式存储在用户主目录下的 .git-credentials 文件中。此方法一劳永逸，但需注意本地文件的安全性。

设置临时缓存
若追求更高的安全性，可选择仅将凭证缓存在内存中一段时间：

启用默认缓存（通常15分钟）：

bash
git config --global credential.helper cache
自定义缓存时长（例如设置1小时超时）：

bash
git config credential.helper 'cache --timeout=3600'
为 SSH 协议管理密钥口令
使用 SSH 密钥对进行认证时，若密钥本身设置了密码短语（passphrase），每次使用仍需输入。可通过 ssh-agent 代理来管理密钥。

启动代理并添加密钥
bash
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_rsa
执行 ssh-add 后会提示输入一次密钥密码，之后在当前终端会话内使用该密钥将不再需要。此方法的局限在于，其生效范围仅限于当前会话，关闭终端或重启系统后需重新执行上述命令。