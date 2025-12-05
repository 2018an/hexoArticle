---
title: Logback日志文件为何不能按日滚动切割
date: 2025-10-29 17:30:25
category: 后端
tags: 日志
---

问题现场：日志堆积的烦恼
一个基于Spring Boot +
Logback的新管理系统上线后，运维同学发现日志文件无限增长，并未像预期那样每天生成一个新的日志文件。所有日志都堆积在单个文件中，只有每次重启应用时，才会创建一个新的日志文件。这给日志管理和历史查询带来了巨大困难。

根因分析：冲突的滚动策略
检查Logback的配置文件 logback-spring.xml，发现了问题所在：

xml
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
<level>INFO</level>
</filter>

    <!-- 基于时间的滚动策略 -->
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <FileNamePattern>${LOG_PATH}/info.log.%d{yyyy-MM-dd}.log</FileNamePattern>
        <MaxHistory>30</MaxHistory>
    </rollingPolicy>

    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
        <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
    </encoder>

    <!-- 问题根源：额外的基于大小的触发策略 -->
    <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
        <MaxFileSize>10MB</MaxFileSize>
    </triggeringPolicy>

</appender>
配置意图：开发者希望日志既能按天归档（TimeBasedRollingPolicy），又能在单个文件超过10MB时立即切割（SizeBasedTriggeringPolicy）。

实际冲突：RollingFileAppender 在与 TimeBasedRollingPolicy 配合时，不支持再单独配置一个
triggeringPolicy。TimeBasedRollingPolicy 本身已经是一个完整的、基于时间触发的策略，附加的 SizeBasedTriggeringPolicy
会被忽略或导致行为异常，最终结果就是基于时间的滚动失效，日志无法按天切割。

解决方案：二选一
方案一：纯按时间滚动（去掉大小策略）
如果只需要按天切割，只需保留 TimeBasedRollingPolicy，移除 triggeringPolicy 部分。

xml
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
<level>INFO</level>
</filter>

    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <!-- 每天生成一个文件，保留30天 -->
        <FileNamePattern>${LOG_PATH}/info.log.%d{yyyy-MM-dd}.log</FileNamePattern>
        <MaxHistory>30</MaxHistory>
    </rollingPolicy>

    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
        <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
    </encoder>
    <!-- 已移除 triggeringPolicy -->

</appender>
方案二：时间与大小组合滚动（推荐）
如果需要“按天归档，且一天内如果日志量过大则按大小拆分”，Logback提供了专用的组合策略 SizeAndTimeBasedRollingPolicy。

xml
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
<level>INFO</level>
</filter>

    <!-- 使用 时间+大小 组合策略 -->
    <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
        <!-- 注意：文件名模式中必须包含 %i 作为索引占位符 -->
        <FileNamePattern>${LOG_PATH}/info.log.%d{yyyy-MM-dd}.%i.log</FileNamePattern>
        <MaxHistory>30</MaxHistory>
        <!-- 单个文件最大大小 -->
        <maxFileSize>20MB</maxFileSize>
    </rollingPolicy>
	
    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
        <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
    </encoder>

</appender>
关键点：使用 SizeAndTimeBasedRollingPolicy 时，<FileNamePattern> 中的 %i（索引序号）是必需的，用于区分同一天内因大小超标而生成的多个文件。

经验总结
策略不兼容：TimeBasedRollingPolicy 与独立的 SizeBasedTriggeringPolicy 不能混用。

使用专用组合策略：需要同时按时间和大小滚动时，务必使用 SizeAndTimeBasedRollingPolicy。

配置后验证：修改配置后，务必观察日志文件的生成情况，确认滚动逻辑按预期工作。

框架通用性：此问题虽以Logback为例，但其原理具有通用性。在使用Log4j等其他框架时，同样需要仔细检查其滚动策略的配置语法，避免类似的策略冲突。