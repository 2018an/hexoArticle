---
title: 快速配置 Tomcat 启用 HTTPS 安全协议
date: 2025-10-29 17:30:25
category: 后端
tags: Tomcat
---

为网站启用 HTTPS 加密协议是保障数据传输安全的重要措施。本文将介绍如何在 Tomcat 服务器中快速配置 HTTPS 支持。

步骤一：调整服务器配置
打开 Tomcat 安装目录下的 conf/server.xml 配置文件，找到被注释的 HTTPS
连接器配置，移除注释并启用该配置段。注意需要手动添加证书密钥密码（keystorePass），该密码在创建证书时设定。

xml
<!-- 启用HTTPS连接器，端口8443 -->
<Connector port="8443" protocol="org.apache.coyote.http11.Http11Protocol"
maxThreads="150"
SSLEnabled="true"
scheme="https"
secure="true"
clientAuth="false"
sslProtocol="TLS"
keystorePass="your_keystore_password" />
步骤二：生成安全证书
使用 JDK 自带的 keytool 工具生成自签名证书。按提示依次输入相关信息，生成过程如下：

bash
keytool -genkey -alias tomcat_https -keyalg RSA -keysize 2048
执行命令后，按照提示输入：

密钥库密码（此密码需与配置文件中的 keystorePass 保持一致）

证书相关信息（姓名、组织、地区等）

证书密钥密码（可直接回车使用与密钥库相同的密码）

验证 HTTPS 访问
配置完成后重启 Tomcat 服务器，即可通过 https://服务器地址:8443/项目路径 访问启用 HTTPS 的应用。

重要提示：此方法生成的为自签名证书，仅适用于开发和测试环境。生产环境建议使用权威证书机构（CA）签发的正式证书，但配置原理相同。