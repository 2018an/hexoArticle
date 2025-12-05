---
title: 本地部署Deepseek各个版本超级详细教学，网页版、软件版
date: 2025-11-04 14:42:34
category: 程序人生
tags: 程序人生
---

最近，AI领域风起云涌，DeepSeek作为备受瞩目的开源大模型，以其卓越的性能和开放的生态吸引了全球开发者的目光。想要在个人电脑上拥有一个私有、安全且功能强大的AI助手吗？本文将为你提供一份从零开始的本地部署全指南。

{% asset_img 1.png 图片描述 %}

第一步：认识你的引擎——Ollama
Ollama是你本地运行大型语言模型的得力管家。它支持Windows、macOS和Linux系统，让你无需联网、不依赖云端服务就能轻松管理和与多种模型（包括Llama、DeepSeek等）进行交互。它提供了便捷的命令行工具和API，并且对GPU资源进行了优化，是个人部署AI的绝佳起点。

第二步：安装与配置Ollama
下载安装包：访问Ollama官网，找到对应你操作系统的安装程序（如Windows的.exe文件）并下载。
{% asset_img 2.png 图片描述 %}

自定义安装路径：如果不想占用宝贵的C盘空间，可以在下载文件夹中打开终端，使用/DIR参数指定安装目录，例如：

bash
.\OllamaSetup.exe /DIR=D:\MyAI\Ollama
{% asset_img 5.png 图片描述 %}
{% asset_img 6.png 图片描述 %}
{% asset_img 7.png 图片描述 %}
{% asset_img 8.png 图片描述 %}

设置环境变量（可选但推荐）：将Ollama的安装路径添加到系统环境变量PATH中，以便在任何位置都能使用ollama命令。
{% asset_img 9.png 图片描述 %}
{% asset_img 10.png 图片描述 %}
{% asset_img 11.png 图片描述 %}

第三步：获取DeepSeek模型
Ollama安装好后，就可以拉取你需要的模型了。DeepSeek提供了不同规模的版本，以适应不同的硬件和能力需求。

访问模型库：在Ollama官网或使用命令搜索deepseek模型。
{% asset_img 12.png 图片描述 %}
{% asset_img 13.png 图片描述 %}
{% asset_img 14.png 图片描述 %}

选择合适版本：根据你的显卡显存选择合适的模型。例如，对于拥有8GB以上显存的用户，deepseek-r1:7b是一个在性能和资源消耗间取得良好平衡的选择。

拉取模型：打开命令行，执行以下命令（请确保Ollama服务已启动）：

bash
ollama pull deepseek-r1:7b
第四步：为模型选择一个交互界面（强烈推荐）
在命令行里对话不够直观，以下三款图形化工具能极大提升你的使用体验。

方案A：Open WebUI（网页版）

安装：确保已安装Python 3.11，然后运行 pip install open-webui。

启动：运行 open-webui serve，在浏览器中访问 http://localhost:8080 即可。

特点：功能全面，可通过浏览器访问，甚至支持局域网内其他设备访问。
{% asset_img 15.png 图片描述 %}
{% asset_img 16.png 图片描述 %}
{% asset_img 17.png 图片描述 %}

方案B：Chatbox AI（桌面软件）

安装：前往官网下载对应操作系统的客户端并安装。

配置：在软件设置中，选择Ollama作为服务提供商，API地址填写 http://localhost:11434，然后选择已下载的DeepSeek模型。

特点：界面清爽，专注于对话，使用简单。
{% asset_img 18.png 图片描述 %}
{% asset_img 19.png 图片描述 %}
{% asset_img 20.png 图片描述 %}

方案C：Cherry Studio（桌面软件）

安装：从其官方网站下载并安装客户端。

配置：同样在设置中连接本地Ollama服务，管理并添加DeepSeek模型。它还支持配置云端API，功能更强大。

特点：界面现代美观，功能集成度高。
{% asset_img 21.png 图片描述 %}
{% asset_img 22.png 图片描述 %}
{% asset_img 23.png 图片描述 %}

第五步：启动与使用
启动核心服务：在任何终端中运行 ollama serve，并保持该窗口运行。

启动交互界面：根据你的选择，运行 open-webui serve 或直接打开Chatbox AI / Cherry Studio软件。

开始对话：在打开的网页或软件中，选择你下载的DeepSeek模型，即可开始与你的私人AI助手进行代码编写、问题解答、创意写作等各类互动。

至此，一个完全运行在你本地、数据私密、响应迅速的DeepSeek AI环境就搭建完成了。尽情探索AI的创造力吧！