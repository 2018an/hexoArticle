---
title: Deepseek 本地部署各个版本超级详细教学，网页版、软件版
date: 2025-11-04 14:42:34
category: 程序人生
tags: 程序人生
---

#### 前言

最近，人工智能行业出现了一股新的浪潮，DeepSeek作为一款备受关注的开源语言模型，凭借其出色的表现和广泛的应用前景，迅速在全球范围内吸引了大量的关注。
从技术圈到商业领域，DeepSeek的热度持续上升，甚至出现了“爆满”的趋势。这不仅展现了其强大的技术优势，也表明市场和用户对它的高度期待。

本篇文章将为大家讲解如何在本地环境中安装并运行DeepSeek模型，同时结合DeepSeek、Ollama、OpenWebUI、Chatbox AI与Cherry Studio等工具，实现高效便捷的模型交互体验。
无论你是技术爱好者、开发者还是企业用户，本文将提供简明的部署步骤，帮助你迅速掌握并充分发挥DeepSeek的优势。

{% asset_img 1.png 图片描述 %}

#### Ollama 介绍

Ollama 是一个开源平台，专为大型语言模型（LLM）设计，旨在简化用户在本地运行、管理和与这些模型进行互动的过程。其核心特色包括：

**本地部署与离线使用**：支持在本地计算机上运行，无需依赖云服务，确保数据隐私。

**跨平台支持**：兼容 Windows、macOS 和 Linux 系统。

**丰富的模型库**：提供多种预训练模型（如 Llama、DeepSeek 等），并支持用户上传自己的模型。

**易于集成和使用**：提供[命令行工具](https://cloud.tencent.com/product/cli?from_column=20065&from=20065)（CLI）、Python SDK 和 RESTful API，方便与其他项目和服务集成。

**性能优化**：支持多 GPU 并行推理加速，能够有效管理内存和计算资源。

**社区驱动**：开源项目，拥有活跃的社区支持。

#### Ollama 安装

打开 Ollama 官方下载页面：[https://ollama.com/download](/developer/tools/blog-entry?target=https%3A%2F%2Follama.com%2Fdownload&objectId=2498296&objectType=1&contentType=undefined)。

在页面中找到 **Windows** 版本的下载链接，点击下载安装程序（通常是一个 `.exe` 文件）。

{% asset_img 2.png 图片描述 %}

下载完成后，双击下载的 `.exe` 文件启动安装程序。

按照安装向导的提示进行操作，包括选择安装路径、接受许可协议等。

点击“下一步”完成安装。

{% asset_img 3.png 图片描述 %}

##### 如何卸载

打开控制面板 ——> 卸载程序 ——>找到删除右键

{% asset_img 4.png 图片描述 %}

##### 注意（如果安装D盘看）：

将Ollama安装到D盘可以释放C盘空间，避免系统盘拥堵，同时提升性能、增强[数据安全](https://cloud.tencent.com/product/dsgc?from_column=20065&from=20065)性，并优化磁盘管理。

下载之后如果点击直接安装默认会安装在`C`盘。 **在下载文件所在文件夹中，**右键在终端打开。

{% asset_img 5.png 图片描述 %}
输入以下命令，指定安装路径（例如安装到 `D:\Ollama`）：

*   `/DIR` 参数用于指定安装路径。
*   确保路径格式正确，路径中不要包含空格或特殊字符。

代码语言：javascript

代码运行次数：0

运行

AI代码解释

复制

```
.\OllamaSetup.exe /DIR=D:\Deekseep\Ollama
```

{% asset_img 6.png 图片描述 %}
{% asset_img 7.png 图片描述 %}
{% asset_img 8.png 图片描述 %}
##### 环境变量

此电脑 ——> 属性 ——>高级系统设置 ——> 环境变量

Path ——> 编辑 ——> 新建 ——> 路径

{% asset_img 9.png 图片描述 %}
{% asset_img 10.png 图片描述 %}
{% asset_img 11.png 图片描述 %}
#### **Ollama 常用命令**

启动 Ollama

代码语言：javascript

代码运行次数：0

运行

AI代码解释

复制

```
ollama serve
```

*   **功能**：启动 Ollama 服务。

创建模型

代码语言：javascript

代码运行次数：0

运行

AI代码解释

复制

```
ollama create <模型名>
```

*   **功能**：从 Modelfile 创建一个自定义模型。

显示模型信息

代码语言：javascript

代码运行次数：0

运行

AI代码解释

复制

```
ollama show <模型名>
```

*   **功能**：显示指定模型的详细信息。

运行模型

代码语言：javascript

代码运行次数：0

运行

AI代码解释

复制

```
ollama run <模型名>
```

*   **功能**：下载（如果尚未下载）并运行指定模型。

停止运行中的模型

代码语言：javascript

代码运行次数：0

运行

AI代码解释

复制

```
ollama stop <模型名>
```

*   **功能**：停止正在运行的模型。

从模型库下载模型

代码语言：javascript

代码运行次数：0

运行

AI代码解释

复制

```
ollama pull <模型名>
```

*   **功能**：从 Ollama 模型库下载指定模型。

将模型推送到模型库

代码语言：javascript

代码运行次数：0

运行

AI代码解释

复制

```
ollama push <模型名>
```

*   **功能**：将本地模型推送到模型库（需要权限）。

列出所有模型

代码语言：javascript

代码运行次数：0

运行

AI代码解释

复制

```
ollama list
```

*   **功能**：显示本地已下载的所有模型。

列出正在运行的模型

代码语言：javascript

代码运行次数：0

运行

AI代码解释

复制

```
ollama ps
```

*   **功能**：显示当前正在运行的模型。

复制模型

代码语言：javascript

代码运行次数：0

运行

AI代码解释

复制

```
ollama cp <源模型名> <目标模型名>
```

*   **功能**：复制一个模型到新的名称。

删除模型

代码语言：javascript

代码运行次数：0

运行

AI代码解释

复制

```
ollama rm <模型名>
```

*   **功能**：删除本地指定的模型。

查看帮助信息

代码语言：javascript

代码运行次数：0

运行

AI代码解释

复制

```
ollama help
```

*   **功能**：显示帮助信息，提供关于所有命令的详细说明。

查看版本信息

代码语言：javascript

代码运行次数：0

运行

AI代码解释

复制

```
ollama --version
```

*   **功能**：显示当前 Ollama 的版本信息。

#### deepseek-r1模型下载

地址：[https://ollama.com/search](/developer/tools/blog-entry?target=https%3A%2F%2Follama.com%2Fsearch&objectId=2498296&objectType=1&contentType=undefined)

{% asset_img 12.png 图片描述 %}
{% asset_img 13.png 图片描述 %}
{% asset_img 14.png 图片描述 %}
#### deepseek-r1模型比较

|版本|适用场景|资源需求|性能表现|
| --- | --- | --- | --- |
|1.5B|轻量级任务（短文本生成、基础问答）|资源消耗低，适合低配设备|生成质量和复杂性有限|
|7B/8B|中等复杂度任务（文案生成、表格处理）|需要中等硬件配置，如 RTX 3060 12G|性能与资源消耗平衡，适合中等任务|
|14B|复杂任务（长文本生成、数据分析）|需要较高硬件配置，如 A100|处理复杂任务能力强，适合专业场景|
|32B/70B|超大规模任务（语言建模、大规模训练）|需要高显存（建议≥24G）或优化后的量化版本|性能逼近更大模型，适合复杂推理|

#### 构建可视化工具

##### open-webui（网页版，局域网可访问）

Open-WebUI 推荐使用 **Python 3.11**，更高版本可能存在兼容性问题。

如果未安装 Python，请从 [Python 官网](/developer/tools/blog-entry?target=https%3A%2F%2Fwww.python.org%2F&objectId=2498296&objectType=1&contentType=undefined)下载并安装 Python 3.11。

在终端或命令提示符中运行以下命令：

代码语言：javascript

代码运行次数：0

运行

AI代码解释

复制

```
pip install open-webui
```

安装完成后，可以通过以下命令启动 Open-WebUI：

代码语言：javascript

代码运行次数：0

运行

AI代码解释

复制

```
open-webui serve
```

启动后，Open-WebUI 默认运行在 `http://localhost:8080`，可以通过浏览器访问该地址。

{% asset_img 15.png 图片描述 %}
{% asset_img 16.png 图片描述 %}
可以在设置更改，相关内容

{% asset_img 17.png 图片描述 %}
##### Chatbox AI（软件版）

**访问Chatbox AI官网**：前往[Chatbox AI官网](/developer/tools/blog-entry?target=https%3A%2F%2Fchatboxai.app%2Fzh&objectId=2498296&objectType=1&contentType=undefined)。

**下载安装包**：根据你的操作系统（Windows、macOS、Linux等）选择对应的安装包。

**安装程序**：双击下载的安装文件，按照提示完成安装。

{% asset_img 18.png 图片描述 %}
**选择语言和主题**：安装完成后，启动Chatbox AI客户端，可在设置菜单中选择语言和主题。

**配置****API**：如果需要使用特定的AI模型，可在“API配置”中输入相应的API密钥或选择预设的模型版本。

**连接本地模型（如Ollama）**：

确保已安装并运行Ollama。

在Chatbox AI中进入设置，选择“Ollama”作为模型提供方。

设置API地址为`http://localhost:11434`，并选择已下载的模型（如DeepSeek）

{% asset_img 19.png 图片描述 %}
{% asset_img 20.png 图片描述 %}
##### Cherry Studio（软件版）

访问 Cherry Studio 官方网站 [https://cherry-ai.com/download](/developer/tools/blog-entry?target=https%3A%2F%2Fcherry-ai.com%2Fdownload&objectId=2498296&objectType=1&contentType=undefined)，根据您的操作系统（Windows、macOS 或 Linux）选择对应版本的安装包

下载完成后，双击安装包运行。

按照安装向导提示选择安装路径和语言（默认为中文）。

完成安装后，启动客户端并同意用户协议即可进入主界面

**配置本地模型（如 DeepSeek）**：

安装本地模型（如通过 Ollama 安装 DeepSeek 模型）。

在 Cherry Studio 中点击左下角的“设置”按钮，进入“模型服务”页面。

找到对应的本地模型服务（如 Ollama），选择管理并配置本地模型。

**配置云服务（如硅基流动）**：

注册硅基流动账号并创建 API 密钥。

在 Cherry Studio 的设置中，找到对应的云服务（如硅基流动），将 API 密钥粘贴到指定位置。

添加并检查模型（如 DeepSeek-R1 或 DeepSeek-V3），完成配置

{% asset_img 21.png 图片描述 %}
{% asset_img 22.png 图片描述 %}
{% asset_img 23.png 图片描述 %}
#### 如何使用

鼠标右键，打开终端

输入 ollama serve 开启 ollama

（可选）网页版open-webui

输入 open-webui serve 不要关掉终端，等一会打开网页，http://localhost:8080

（可选）软件版Chatbox AI，打开软件，运行

（可选）软件版Cherry Studio，打开软件，运行