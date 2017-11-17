title: springboot下logback日志配置
author: 赵旭东
tags:
  - Spring Boot
  - logback
categories:
  - 技术
date: 2017-11-17 17:29:00
---
在resources文件夹下建立logback.xml文件。

<!--more-->


文件内容如下：
```
<?xml version="1.0" encoding="UTF-8" ?>
<configuration scan="true" scanPeriod="60 seconds" debug="true">
    <!--日志存放目录，根据实际情况修改-->
    <property name="log.folder" value="/opt/logs/ocr"/>
    <!--日志名字，根据实际情况修改-->
    <property name="log.name" value="ocr-service"/>
    <property name="ENCODER_PATTERN" value="[%d{yyyy-MM-dd HH:mm:ss} %5p %class:%L] %m%n"/>

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder" charset="UTF-8">
            <Pattern>${ENCODER_PATTERN}</Pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>info</level>
        </filter>
    </appender>

    <appender name="errorLogger"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.folder}/error.%d{yyyy-MM-dd}.log
            </fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder charset="UTF-8">
            <pattern>${ENCODER_PATTERN}</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>error</level>
        </filter>
    </appender>


    <appender name="infoLogger"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.folder}/info.%d{yyyy-MM-dd}.log
            </fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder charset="UTF-8">
            <pattern>${ENCODER_PATTERN}</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>info</level>
        </filter>
    </appender>

    <appender name="warnLogger"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.folder}/warn.%d{yyyy-MM-dd}.log
            </fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder charset="UTF-8">
            <pattern>${ENCODER_PATTERN}</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>warn</level>
        </filter>
    </appender>

    <root level = "debug">
        <appender-ref ref="errorLogger"/>
        <appender-ref ref="infoLogger"/>
        <appender-ref ref="warnLogger"/>
        <appender-ref ref="STDOUT"/>
        <!--<appender-ref ref="FILE"/>-->
    </root>
</configuration>

```