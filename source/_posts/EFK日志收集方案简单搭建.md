---
title: EFK日志收集方案简单搭建
date: 2022/06/12 22:18:00
categories:
- 日志收集
tags:
- efl
---

### 示例应用

示例应用`app-samp-foo`，spring-boot项目，主要配置如下：

#### 日志配置

bootstrap.yml

```yaml
spring:
  application:
    name: app-samp-foo
logging:
  file:
    max-history: 7
  config: classpath:logback-spring.xml
  home: /data/app/logs/${spring.application.name}
```

<!--more-->

logback-spring.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!--<include resource="org/springframework/boot/logging/logback/base.xml"/>-->

    <!-- 避免生成spring.log -->
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    <include resource="org/springframework/boot/logging/logback/console-appender.xml"/>

    <jmxConfigurator/>
    <springProperty scope="context" name="springAppName" source="spring.application.name"/>
    <springProperty scope="context" name="logPath" source="logging.home"/>
    <springProperty scope="context" name="maxHistory" source="logging.file.max-history"/>

    <appender name="infoAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
        <file>${logPath}/${springAppName}.info.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${logPath}/%d{yyyy-MM-dd}-${springAppName}-%i.info.log</fileNamePattern>
            <maxHistory>${maxHistory:-7}</maxHistory>
            <maxFileSize>50MB</maxFileSize>
            <totalSizeCap>10GB</totalSizeCap>
        </rollingPolicy>
        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
            <layout class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.TraceIdPatternLogbackLayout">
                <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度 %msg：日志消息，%n是换行符-->
                <!-- 异常堆栈信息会被自动追踪打印 ref: https://logback.qos.ch/manual/layouts.html -->
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS,Asia/Shanghai} [%tid] [%thread] %-5level %logger{50} - %msg%n</pattern>
            </layout>
        </encoder>
    </appender>

    <appender name="errAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
        <file>${logPath}/${springAppName}.err.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${logPath}/%d{yyyy-MM-dd}-${springAppName}-%i.err.log</fileNamePattern>
            <maxHistory>${maxHistory:-7}</maxHistory>
            <maxFileSize>50MB</maxFileSize>
            <totalSizeCap>10GB</totalSizeCap>
        </rollingPolicy>
        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
            <layout class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.TraceIdPatternLogbackLayout">
                <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度 %msg：日志消息，%n是换行符-->
                <!-- 异常堆栈信息会被自动追踪打印 ref: https://logback.qos.ch/manual/layouts.html -->
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS,Asia/Shanghai} [%tid] [%thread] %-5level %logger{50} - %msg%n</pattern>
            </layout>
        </encoder>
    </appender>

    <!-- add converter for %tid -->
    <conversionRule conversionWord="tid"
                    converterClass="org.apache.skywalking.apm.toolkit.log.logback.v1.x.LogbackPatternConverter"/>

    <appender name="infoAppenderJson" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
        <file>${logPath}/${springAppName}.info.json</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${logPath}/%d{yyyy-MM-dd}-${springAppName}-%i.info.json</fileNamePattern>
            <maxHistory>${maxHistory:-7}</maxHistory>
            <maxFileSize>50MB</maxFileSize>
            <totalSizeCap>10GB</totalSizeCap>
        </rollingPolicy>
        <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
            <providers class="net.logstash.logback.composite.loggingevent.LoggingEventJsonProviders">
                <pattern>
                    <pattern>
                        {
                        "date":"%date{yyyy-MM-dd HH:mm:ss.SSS,Asia/Shanghai}",
                        "level":"%level",
                        "tid": "%tid",
                        "appName":"${springAppName}",
                        "thread":"%thread",
                        "class": "%logger{1.}:%L",
                        "message": "%message",
                        "stackTrace":"%xThrowable{full}"
                        }
                    </pattern>
                </pattern>
            </providers>
        </encoder>
    </appender>

    <appender name="errAppenderJson" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
        <file>${logPath}/${springAppName}.err.json</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${logPath}/%d{yyyy-MM-dd}-${springAppName}-%i.err.json</fileNamePattern>
            <maxHistory>${maxHistory:-7}</maxHistory>
            <maxFileSize>50MB</maxFileSize>
            <totalSizeCap>10GB</totalSizeCap>
        </rollingPolicy>
        <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
            <providers class="net.logstash.logback.composite.loggingevent.LoggingEventJsonProviders">
                <pattern>
                    <pattern>
                        {
                        "date":"%date{yyyy-MM-dd HH:mm:ss.SSS,Asia/Shanghai}",
                        "level":"%level",
                        "tid": "%tid",
                        "appName":"${springAppName}",
                        "thread":"%thread",
                        "class": "%logger{1.}:%L",
                        "message": "%message",
                        "stackTrace":"%xThrowable{full}"
                        }
                    </pattern>
                </pattern>
            </providers>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="infoAppender"/>
        <appender-ref ref="errAppender"/>
        <appender-ref ref="infoAppenderJson"/>
        <appender-ref ref="errAppenderJson"/>
    </root>
</configuration>

```

注意以上日志配置，加入了skywalking tid输出和日志增加了json格式，需要再pom中加入以下依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.skywalking</groupId>
        <artifactId>apm-toolkit-logback-1.x</artifactId>
        <version>8.9.0</version>
    </dependency>
    <dependency>
        <groupId>net.logstash.logback</groupId>
        <artifactId>logstash-logback-encoder</artifactId>
        <version>5.3</version>
    </dependency>
</dependencies>
```

#### Filebeat配置

filebeat.yml

```yaml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - E:\data\app\logs\*\*.info.json
  json: #解析json日志配置，用法可参考同目录下filebeat.reference.yml文件
    message_key: "message" #java输出json格式日志中代表日志内容的字段key
    keys_under_root: true
    overwrite_keys: true
    add_error_key: true

#==================== Elasticsearch template setting ==========================

setup.template.settings:
  index.number_of_shards: 3
  
#================================ Processors ==================================

processors:
  - drop_fields:
      fields: ["log", "input","prospector"] # 移除非必要字段

#================================ Outputs =====================================

#-------------------------- Elasticsearch output ------------------------------
output.elasticsearch:
  hosts: ["localhost:9200"]
  index: "app-log-%{+yyyy.MM.dd}" # 自定义索引名称必须同时修改setup.template.name & setup.template.pattern

setup:
  template:
    name: "app-log"
    pattern: "app-log-*"
    enabled: false #关掉默认的模板配置
    overwrite: true #开启新设置的模板
    
#----------------------------- Console output ---------------------------------
# 控制台输出配置示例,output只能同时打开一项
#output.console:
#  enabled: true
#  codec.json:
#    pretty: false
#    escape_html: true
```

启动es和filebeat

```shell script
## 启动es
PS D:\dev_app\elasticsearch-6.8.23\bin> pwd

Path
----
D:\dev_app\elasticsearch-6.8.23\bin

PS D:\dev_app\elasticsearch-6.8.23\bin> .\elasticsearch.bat

## 测试配置是否ok
$ ./filebeat.exe test config
Config OK

## 启动
$ ./filebeat.exe -e -c ./filebeat.yml
```
