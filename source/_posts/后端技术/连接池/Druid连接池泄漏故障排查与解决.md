---
title: Druid连接池泄漏故障排查与解决
date: 2025-10-29 17:30:25
category: 后端
tags: 连接池
---

案例一：最近我们负责的一个应用频繁出现响应迟缓的现象，必须通过重启服务才能暂时恢复正常，由此引发了一系列用户投诉。经过深入追踪，发现问题根源在于Druid数据库连接池发生了连接泄漏。

错误日志如下：

```text
ERROR - com.alibaba.druid.pool.GetConnectionTimeoutException: wait millis 60000, active 50, maxActive 50, creating 0
at com.alibaba.druid.pool.DruidDataSource.getConnectionInternal(DruidDataSource.java:1512)
at com.alibaba.druid.pool.DruidDataSource.getConnectionDirect(DruidDataSource.java:1255)
at com.alibaba.druid.filter.FilterChainImpl.dataSource_connect(FilterChainImpl.java:5007)
at com.alibaba.druid.filter.stat.StatFilter.dataSource_getConnection(StatFilter.java:680)
at com.alibaba.druid.filter.FilterChainImpl.dataSource_connect(FilterChainImpl.java:5003)
at com.alibaba.druid.pool.DruidDataSource.getConnection(DruidDataSource.java:1233)
at com.alibaba.druid.pool.DruidDataSource.getConnection(DruidDataSource.java:1225)
at com.alibaba.druid.pool.DruidDataSource.getConnection(DruidDataSource.java:90)
```
从日志可以看出，连接池中的所有连接均已占用，且等待60秒后仍无法获得新连接，最终抛出超时异常。

问题的本质在于应用程序中某些代码段在获取数据库连接后未能正确释放。由于代码规模较大，全面审查所有数据库操作逻辑较为困难。我们通过启用Druid的三项相关配置，最终锁定了泄漏点并予以解决。

新增配置如下：

```xml
<!-- 启用连接强制回收机制 -->
<property name="removeAbandoned" value="true" />

<!-- 连接被判定为泄露的阈值时间，单位毫秒 -->
<property name="removeAbandonedTimeoutMillis" value="600000"/>

<!-- 回收连接时是否输出相关日志 -->
<property name="logAbandoned" value="true" />
```
这些设置专用于应对连接泄漏问题。当removeAbandoned开启后，若连接被占用时间超过removeAbandonedTimeoutMillis所设阈值，连接池将强制回收该连接。

Druid数据源在初始化时会启动一个后台线程，定期扫描并回收此类超时连接。相关逻辑可参考源码中的com.alibaba.druid.pool.DruidDataSource#createAndStartDestroyThread方法。

当logAbandoned设置为true时，在回收连接的同时会记录该连接被获取时的调用堆栈信息。这份日志能够清晰指出哪些代码段打开了连接却未关闭，极大便利了问题定位。

```text
abandon connection, owner thread: https-jsse-nio-4443-exec-9, connected at : 1573521883837, open stackTrace
at java.lang.Thread.getStackTrace(Thread.java:1589)
at com.alibaba.druid.pool.DruidDataSource.getConnectionDirect(DruidDataSource.java:1305)
at com.alibaba.druid.filter.FilterChainImpl.dataSource_connect(FilterChainImpl.java:4619)
at com.alibaba.druid.filter.stat.StatFilter.dataSource_getConnection(StatFilter.java:680)
at com.alibaba.druid.filter.FilterChainImpl.dataSource_connect(FilterChainImpl.java:4615)
at com.alibaba.druid.pool.DruidDataSource.getConnection(DruidDataSource.java:1225)
at com.alibaba.druid.pool.DruidDataSource.getConnection(DruidDataSource.java:1217)
at com.alibaba.druid.pool.DruidDataSource.getConnection(DruidDataSource.java:90)
at org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource.getConnection(AbstractRoutingDataSource.java:
162)
...
```
尽管此配置能有效协助排查连接泄漏，但在生产环境中需谨慎使用。若业务中存在执行时间较长的数据库事务，可能会被此机制误判为泄漏而强制回收，从而引发新的问题。


案例二：处理Druid连接池与Oracle Clob类型的兼容性问题
遇到的问题
在通过Druid连接池操作Oracle数据库，并试图将查询结果中的Clob字段转换为原生Oracle Clob类型时，程序抛出了以下异常：

```text
java.lang.ClassCastException: com.alibaba.druid.proxy.jdbc.ClobProxyImpl cannot be cast to oracle.sql.CLOB
```
异常分析
ClobProxyImpl
无法直接转换为Oracle原生的CLOB类型。其根本原因在于，Druid对原始的Clob对象进行了封装，引入了自己的代理类com.alibaba.druid.proxy.jdbc.ClobProxyImpl。当代码中尝试将其强制转换为oracle.sql.CLOB时，便导致了类型转换失败。

解决方案
目前的解决思路是：先获取到Druid封装后的代理对象，再通过代理对象取得其中包装的原生Oracle Clob内容。

以下是一个工具方法的示例：

```java
public class ClobUtil {

	public static CLOB parseOracleClob(Clob clob) {
		// 假设从结果集中得到的是Druid包装后的SerializableClob
		SerializableClob sclob = (SerializableClob) clob;
		Clob wrappedClob = sclob.getWrappedClob();

		// 针对Druid的特殊处理
		if (wrappedClob instanceof ClobProxy) {
			ClobProxy clobProxy = (ClobProxy) wrappedClob;
			// 获取被代理的原始Clob对象
			wrappedClob = clobProxy.getRawClob();
		}

		// 此时wrappedClob应为原生的Oracle CLOB
		return (CLOB) wrappedClob;
	}

}
```
