---
title: 全面解决 IntelliJ IDEA 中的字符编码乱码问题
date: 2025-10-30 14:42:34
category: 工具
tags: IDEA
---

在使用 IntelliJ IDEA 时，不同场景下出现的乱码问题确实令人困扰。本文将系统梳理几种常见的乱码情形，并提供已验证的解决方案，帮助你彻底摆脱编码烦恼。

常见乱码场景与对应修复方案

1. 项目源代码文件内容显示乱码
   解决路径： File > Settings (Windows/Linux) 或 IntelliJ IDEA > Preferences (macOS) > Editor > File Encodings
   操作： 将 “Global Encoding”、”Project Encoding” 以及 “Default encoding for properties files” 统一设置为 UTF-8。
   https://img/20190807113633.png

2. 运行 Main 方法时控制台输出乱码
   解决路径： Settings/Preferences > Build, Execution, Deployment > Compiler > Java Compiler
   操作： 在 “Additional command line parameters” 选项中填入：-encoding utf-8。
   https://img/20190807113844.png

3. 使用 Tomcat 运行 Web 项目时控制台乱码
   这是一个多步骤的解决过程，请依次尝试：

步骤 A (配置当前运行配置)：
操作： 打开 “Run/Debug Configurations”，选择你的 Tomcat 配置，在 “Server” 标签页的 “VM options” 中输入：-Dfile.encoding=UTF-8。
https://img/20190807114037.png

步骤 B (修改 IDE 虚拟机选项)：
操作： 关闭 IDEA，找到安装目录下 bin 文件夹中的 idea64.exe.vmoptions (或 idea.exe.vmoptions)
文件，在文件末尾添加一行：-Dfile.encoding=UTF-8，然后重启 IDEA。
https://img/20190807134723.png

步骤 C (修改内部自定义选项)：
操作： 如果乱码依旧，在 IDEA 运行时，通过菜单栏 Help > Edit Custom VM Options… 打开配置文件，同样在末尾添加
-Dfile.encoding=UTF-8 并重启。
https://img/20190807114251.png
https://img/20190807114334.png