---
title: 功能强大的Java数据库连接池
date: 2025-10-29 17:30:25
category: 后端
tags: 连接池
---

Druid是阿里巴巴开源的一款高性能Java数据库连接池。它不仅提供了基础的连接池功能，更内置了丰富的监控和扩展能力，可以说是一款为实时监控而设计的数据库连接池。

项目地址：https://github.com/alibaba/druid/

引入依赖
在Maven项目中添加以下依赖即可：

```xml
<dependency>
<groupId>com.alibaba</groupId>
<artifactId>druid</artifactId>
<version>1.1.2</version>
</dependency>
```
常用配置参考
以下是一份较为完整的Spring Bean配置示例：

```xml
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
init-method="init" destroy-method="close">
<!-- 数据库连接基本信息 -->
<property name="url" value="${jdbc_url}" />
<property name="username" value="${jdbc_user}" />
<property name="password" value="${jdbc_password}" />

      <!-- 连接池容量配置 -->
      <property name="initialSize" value="1" />
      <property name="minIdle" value="1" /> 
      <property name="maxActive" value="20" />
   
      <!-- 获取连接的最大等待时间（毫秒） -->
      <property name="maxWait" value="60000" />
   
      <!-- 检测并关闭空闲连接的间隔时间（毫秒） -->
      <property name="timeBetweenEvictionRunsMillis" value="60000" />
   
      <!-- 连接在池中最小可被回收的空闲时间（毫秒） -->
      <property name="minEvictableIdleTimeMillis" value="300000" />
    
      <!-- 连接有效性检测SQL -->
      <property name="validationQuery" value="SELECT 'x'" />
      <property name="testWhileIdle" value="true" />
      <property name="testOnBorrow" value="false" />
      <property name="testOnReturn" value="false" />
   
      <!-- 是否缓存PreparedStatement -->
      <property name="poolPreparedStatements" value="true" />
      <property name="maxPoolPreparedStatementPerConnectionSize" value="20" />
   
      <!-- 配置监控统计用的过滤器 -->
      <property name="filters" value="stat" /> 

</bean>
```
通常，根据应用负载调整initialSize、minIdle、maxActive这三个参数即可。对于Oracle数据库，建议将poolPreparedStatements设为true以提升性能；而对于MySQL或分库分表较多的场景，则可设为false。

启用Web监控
在web.xml中配置以下Servlet和Filter，即可启用Druid内置的Web监控台：

```xml
<!-- Druid监控视图Servlet -->
<servlet>  
    <servlet-name>DruidStatView</servlet-name>  
    <servlet-class>com.alibaba.druid.support.http.StatViewServlet</servlet-class>
</servlet>  
<servlet-mapping>  
    <servlet-name>DruidStatView</servlet-name>  
    <url-pattern>/druid/*</url-pattern>  
</servlet-mapping>  

<!-- Druid监控统计过滤器 -->
<filter>
	<filter-name>DruidWebStatFilter</filter-name>
	<filter-class>com.alibaba.druid.support.http.WebStatFilter</filter-class>
	<init-param>
		<param-name>exclusions</param-name>
		<param-value>*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*</param-value>
	</init-param>
</filter>
<filter-mapping>
	<filter-name>DruidWebStatFilter</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>
```
配置完成后，通过访问 http://localhost:8080/你的项目路径/druid/ 即可查看详细的连接池监控信息。