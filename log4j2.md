[TOC]

# 一. 依赖

官网：https://logging.apache.org/log4j/2.x/index.html

中文文档：https://www.docs4dev.com/docs/zh/log4j2/2.x/all/manual-index.html

maven 构建项目可添加相关依赖：

```xml
<!-- log4j2 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>${log4j2-version}</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>${log4j2-version}</version>
</dependency>

```

# 二. 基本设置

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!-- status : 指定log4j本身的打印日志的级别.ALL< Trace < DEBUG < INFO < WARN < ERROR
    < FATAL < OFF。 monitorInterval : 用于指定log4j自动重新配置的监测间隔时间，单位是s,最小是5s. -->
<Configuration status="WARN" monitorInterval="30">
    <Properties>
        <!-- 配置日志文件输出目录 ${sys:user.home} -->
        <Property name="LOG_HOME">/root/workspace/lucenedemo/logs</Property>
        <property name="ERROR_LOG_FILE_NAME">/root/workspace/lucenedemo/logs/error</property>
        <property name="WARN_LOG_FILE_NAME">/root/workspace/lucenedemo/logs/warn</property>

        <property name="PATTERN">%d{yyyy-MM-dd HH:mm:ss.SSS} [%t-%L] %-5level %logger{36} - %msg%n</property>
    </Properties>

    <Appenders>
        <!--这个输出控制台的配置 -->
        <Console name="Console" target="SYSTEM_OUT">
            <!-- 控制台只输出level及以上级别的信息(onMatch),其他的直接拒绝(onMismatch) -->
            <ThresholdFilter level="trace" onMatch="ACCEPT"
                             onMismatch="DENY" />
            <!-- 输出日志的格式 -->
            <!--
                %d{yyyy-MM-dd HH:mm:ss, SSS} : 日志生产时间
                %p : 日志输出格式
                %c : logger的名称
                %m : 日志内容，即 logger.info("message")
                %n : 换行符
                %C : Java类名
                %L : 日志输出所在行数
                %M : 日志输出所在方法名
                hostName : 本地机器名
                hostAddress : 本地ip地址 -->
            <PatternLayout
                    pattern="${PATTERN}" />
        </Console>

        <!--文件会打印出所有信息，这个log每次运行程序会自动清空，由append属性决定，这个也挺有用的，适合临时测试用 -->
        <!--append为TRUE表示消息增加到指定文件中，false表示消息覆盖指定的文件内容，默认值是true -->
        <File name="log" fileName="logs/test.log" append="false">
            <PatternLayout
                    pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </File>
        <!-- 这个会打印出所有的info及以下级别的信息，每次大小超过size，
        则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档 -->
        <RollingFile name="RollingFileInfo" fileName="${LOG_HOME}/info.log"
                     filePattern="${LOG_HOME}/$${date:yyyy-MM}/info-%d{yyyy-MM-dd}-%i.log">
            <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch） -->
            <ThresholdFilter level="info" onMatch="ACCEPT"
                             onMismatch="DENY" />
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n" />
            <Policies>
                <!-- 基于时间的滚动策略，interval属性用来指定多久滚动一次，默认是1 hour。 modulate=true用来调整时间：比如现在是早上3am，interval是4，那么第一次滚动是在4am，接着是8am，12am...而不是7am. -->
                <!-- 关键点在于 filePattern后的日期格式，以及TimeBasedTriggeringPolicy的interval，
                日期格式精确到哪一位，interval也精确到哪一个单位 -->
                <!-- log4j2的按天分日志文件 : info-%d{yyyy-MM-dd}-%i.log-->
                <TimeBasedTriggeringPolicy interval="1" modulate="true" />
                <!-- SizeBasedTriggeringPolicy:Policies子节点， 基于指定文件大小的滚动策略，size属性用来定义每个日志文件的大小. -->
                <!-- <SizeBasedTriggeringPolicy size="2 kB" />  -->
            </Policies>
        </RollingFile>

        <RollingFile name="RollingFileWarn" fileName="${WARN_LOG_FILE_NAME}/warn.log"
                     filePattern="${WARN_LOG_FILE_NAME}/$${date:yyyy-MM}/warn-%d{yyyy-MM-dd}-%i.log">
            <ThresholdFilter level="warn" onMatch="ACCEPT"
                             onMismatch="DENY" />
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n" />
            <Policies>
                <TimeBasedTriggeringPolicy />
                <SizeBasedTriggeringPolicy size="2 kB" />
            </Policies>
            <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件，这里设置了20 -->
            <DefaultRolloverStrategy max="20" />
        </RollingFile>

        <RollingFile name="RollingFileError" fileName="${ERROR_LOG_FILE_NAME}/error.log"
                     filePattern="${ERROR_LOG_FILE_NAME}/$${date:yyyy-MM}/error-%d{yyyy-MM-dd-HH-mm}-%i.log">
            <ThresholdFilter level="error" onMatch="ACCEPT"
                             onMismatch="DENY" />
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n" />
            <Policies>
                <!-- log4j2的按分钟 分日志文件 : warn-%d{yyyy-MM-dd-HH-mm}-%i.log-->
                <TimeBasedTriggeringPolicy interval="1" modulate="true" />
                <!-- <SizeBasedTriggeringPolicy size="10 MB" /> -->
            </Policies>
        </RollingFile>

    </Appenders>

    <!--然后定义logger，只有定义了logger并引入的appender，appender才会生效-->
    <Loggers>
        <!--过滤掉spring和mybatis的一些无用的DEBUG信息-->
        <logger name="org.springframework" level="INFO"/>
        <logger name="org.mybatis" level="INFO"/>

        <!-- 第三方日志系统 -->
        <logger name="org.springframework.core" level="info" />
        <logger name="org.springframework.beans" level="info" />
        <logger name="org.springframework.context" level="info" />
        <logger name="org.springframework.web" level="info" />
        <logger name="org.jboss.netty" level="warn" />
        <logger name="org.apache.http" level="warn" />


        <!-- 配置日志的根节点 -->
        <root level="all">
            <appender-ref ref="Console"/>
            <appender-ref ref="RollingFileInfo"/>
            <appender-ref ref="RollingFileWarn"/>
            <appender-ref ref="RollingFileError"/>
        </root>

    </Loggers>

</Configuration>

```

# 三. Loggers 配置详解

Loggers 节点，常见的有两种：Root 和 Logger

`Root 节点用来指定项目的根日志，如果没有单独指定 `Logger`，那么就会默认使用该 `Root` 日志输出

## 1. Root

每个配置都必须有一个根记录器 Root，如果没有配置，则将使用默认根 LoggerConfig，其级别为 Error 且附加了 Console appender。根记录器和其他记录器之间的主要区别是：

1. 根记录器没有 name 属性
2. 根记录器不支持 additivity 属性，因为他没有父级

> - 日志输出级别，共有 8 个级别，按照从低到高为：All < Trace < Debug < Info < Warn < Error < Fetal < OFF
> - Appender-Ref: Root 的子节点，用来指定该入职输出到那个 Appender

## 2. Logger

Logger 节点用来单独指定日志的形式，比如要为指定包下的 class 指定不同的日志级别等

使用 Logger 元素必须有一个 name 属性，Root Logger 不用 name 元属性

每个 Logger 可以使用 Trace/Debug/Info/Warn/Error/All/OFF 之一配置级别，如果未指定级别，则默认为 Error。可以为 additivity 属性分配至 true 或 false。如果省略该属性，则将使用默认值 true

Logger 还可以配置一个或多个 Appender-Ref 属性，引用的每个 appender 将与指定的 Logger 关联，如果在 Logger 上配置了多个 appender，则在处理日志记录事件时会调用多个 appender

- name：用来指定该 Logger 所适用的类或者类所在的包全路径，继承自 Root 节点，一般是项目包名或者框架的包名，比如：com.jourwon, org.springframework
- level：日志输出级别
- Appender-Ref：Logger 的子节点，用来指定该日志输出到哪个 Appender，如果没有指定，就会默认继承自 Root。如果指定了，那么会在指定的这个 Appender 和 Root 的 Appender 中都会输出，此时我们可以设置 Logger 的 additivity = "false" 只在自定义的 Appender 中进行输出

# 四. 代码中使用

```java
package com.log;
 
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.junit.Test;
 
public class Log4j2Test {
    // 定义日志记录器对象
    public static final Logger LOGGER = LogManager.getLogger(Log4j2Test.class);
    // 快速入门
    @Test
    public void testQuick() throws Exception {
        // 日志消息输出
        LOGGER.fatal("fatal");
        LOGGER.error("error");
        LOGGER.warn("warn");
        LOGGER.info("info");
        LOGGER.debug("debug");
        LOGGER.trace("trace");
    }
}
```

