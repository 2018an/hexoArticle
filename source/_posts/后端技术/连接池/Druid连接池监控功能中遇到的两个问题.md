---
title: Druid连接池监控功能中遇到的两个问题
date: 2025-10-29 17:30:25
category: 后端
tags: 连接池
---


https://img/18-2-4-38258738.jpg

阿里巴巴的Druid连接池以其全面的监控特性而广受欢迎。然而在实际部署和使用过程中，其监控模块也存在一些值得注意的问题。下面分享两个近期我们遇到的典型情况。

问题一：持续输出“session ip change too many”错误日志
错误信息对应的核心源码片段
位于 com.alibaba.druid.support.http.stat.WebSessionStat#addRemoteAddress 方法中：

```java
public void addRemoteAddress(String ip) {
if (remoteAddresses == null) {
this.remoteAddresses = ip;
return;
}

    if (remoteAddresses.contains(ip)) {
        return;
    }

    if (remoteAddresses.length() > 256) {
        LOG.error("session ip change too many");
        return;
    }

    remoteAddresses += ';' + ip;

}
```
Druid获取客户端IP的逻辑
相关代码位于 com.alibaba.druid.util.DruidWebUtils：

```java
public static String getRemoteAddr(HttpServletRequest request) {
String ip = request.getHeader("x-forwarded-for");
if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
ip = request.getHeader("Proxy-Client-IP");
}
if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
ip = request.getHeader("WL-Proxy-Client-IP");
}
if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
ip = request.getRemoteAddr();
}

    return ip;

}
```
问题分析
这是Druid会话监控功能的一部分，它会记录同一Session ID对应的所有访问IP。当拼接后的IP字符串长度超过256个字符时，便会记录此错误日志（实际功能不受影响）。

观察发现，单个会话的请求次数并不高，但记录的IP字符串却异常长。原因在于，当请求经过多层代理时，x-forwarded-for等头部可能包含多个IP（例如：192.168.1.2,192.168.1.3,192.168.1.4）。这样，几次请求就可能使remoteAddresses字符串长度超过256，从而触发错误日志。

解决方式

如果不需要会话监控功能，可以直接关闭它。在WebStatFilter的配置中添加：

```xml
<init-param>
<param-name>sessionStatEnable</param-name>
<param-value>false</param-value>
</init-param>
```
修改Druid源码，对多段IP进行截取（例如只取第一段），并考虑增大IP记录长度的限制。

目前，即使在较新的Druid版本中，此问题依然存在。

https://img/18-1-29-92452744.jpg

在GitHub的Druid官方issue列表中也能找到同类问题，但似乎尚未被修复。因此，我们暂时选择了关闭会话监控功能。

问题二：访问监控页面时发生ConcurrentModificationException
错误堆栈如下：

```text
java.util.ConcurrentModificationException
at java.util.LinkedHashMap$LinkedHashIterator.nextEntry(LinkedHashMap.java:394)
at java.util.LinkedHashMap$ValueIterator.next(LinkedHashMap.java:409)
at java.util.Collections$UnmodifiableCollection$1.next(Collections.java:1067)
at com.alibaba.druid.support.http.stat.WebAppStat.getSessionStatDataList(WebAppStat.java:504)
at com.alibaba.druid.support.http.stat.WebAppStatUtils.getSessionStatDataList(WebAppStatUtils.java:64)
at com.alibaba.druid.support.http.stat.WebAppStatManager.getSessionStatData(WebAppStatManager.java:100)
at com.alibaba.druid.stat.DruidStatService.getWebSessionStatDataList(DruidStatService.java:205)
at com.alibaba.druid.stat.DruidStatService.service(DruidStatService.java:161)
at com.alibaba.druid.support.http.StatViewServlet.process(StatViewServlet.java:162)
at com.alibaba.druid.support.http.ResourceServlet.service(ResourceServlet.java:253)
```
原因追踪
查看源码后发现，问题同样出在会话监控相关的统计代码中。

https://img/18-1-29-83615861.jpg

在循环内部对集合进行修改，而其他线程可能同时也在操作该集合，导致了ConcurrentModificationException异常。

因此，最终的解决方案同样是：关闭会话级别的监控功能。