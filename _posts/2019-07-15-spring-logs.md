---
title: spring-boot 中的 log 配置
key: 2019-07-15
tags: spring-boot log log4j logback
---

在项目推进中，如果说第一件事是搭Spring框架的话，那么第二件事情就是在Sring基础上搭建日志框架

<!--more-->



## 常用日志框架

- java.util.logging：是JDK在1.4版本中引入的Java原生日志框架
- Log4j：Apache的一个开源项目，可以控制日志信息输送的目的地是控制台、文件、GUI组件等，可以控制每一条日志的输出格式，这些可以通过一个配置文件来灵活地进行配置，而不需要修改应用的代码。虽然已经停止维护了，但目前绝大部分企业都是用的log4j。
- LogBack：是Log4j的一个改良版本
- Log4j2：Log4j2已经不仅仅是Log4j的一个升级版本了，它从头到尾都被重写了



spring boot 使用 Commons Logging 作为内部的日志系统，并且给 Java Util Logging，Log4J2 以及Logback都提供了默认的配置。如果使用了 spring boot的Starters，那么默认会使用 Logback 用于记录日志。



### 控制台输出



```bash
$ java -jar myapp.jar --debug
```




## logback





控制台的默认配置



```bash
logging.pattern.console=%clr(%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}
```



其中%clr为配置不同的颜色输出，支持的颜色有以下几种：

- `blue`
- `cyan`
- `faint`
- `green`
- `magenta`
- `red`
- `yellow`



**输出顺序分析：**

1、日期和时间：精确到毫秒，并按照时间进行简单的排序，格式为：

```
%clr(%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}}){faint}
```


2、日志级别：ERROR,WARN,INFO,DEBUG,TRACE

```
%clr(${LOG_LEVEL_PATTERN:-%5p})
```


3、进程ID号

```
%clr(${PID:- })
```


4、日志内容：用 "---" 分隔符分开

```
%clr(---){faint}
```



5、线程名字：括在方括号中　

```
%clr([%15.15t]){faint}
```



6、日志的名字：通常对应的是类名

```
%clr(%-40.40logger{39}){cyan}
```

注意：Logback没有FATAL级别（映射到ERROR）





不同级别的日志信息显示为不同的颜色:

| Level   | Color  |
| ------- | ------ |
| `FATAL` | Red    |
| `ERROR` | Red    |
| `WARN`  | Yellow |
| `INFO`  | Green  |
| `DEBUG` | Green  |
| `TRACE` | Green  |

Alternatively, you can specify the color or style that should be used by providing it as an option to the conversion. For example, to make the text yellow, use the following setting:

```
%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){yellow}
```



####  输出文件



By default, Spring Boot logs only to the console and does not write log files. If you want to write log files in addition to the console output, you need to set a `logging.file` or `logging.path` property (for example, in your `application.properties`).

The following table shows how the `logging.*` properties can be used together:



**Table 26.1. Logging properties**

| `logging.file` | `logging.path`     | Example    | Description                                                  |
| -------------- | ------------------ | ---------- | ------------------------------------------------------------ |
| *(none)*       | *(none)*           |            | Console only logging.                                        |
| Specific file  | *(none)*           | `my.log`   | Writes to the specified log file. Names can be an exact location or relative to the current directory. |
| *(none)*       | Specific directory | `/var/log` | Writes `spring.log` to the specified directory. Names can be an exact location or relative to the current directory. |

Log files rotate when they reach 10 MB and, as with console output, `ERROR`-level,
`WARN`-level, and `INFO`-level messages are logged by default. Size limits can be changed
using the `logging.file.max-size` property. Previously rotated files are archived
indefinitely unless the `logging.file.max-history` property has been set.









###  日志配置

All the supported logging systems can have the logger levels set in the Spring `Environment` (for example, in `application.properties`) by using `logging.level.<logger-name>=<level>` where `level` is one of TRACE, DEBUG, INFO, WARN, ERROR, FATAL, or OFF. The `root` logger can be configured by using `logging.level.root`.

The following example shows potential logging settings in `application.properties`:

```bash
logging.level.root=WARN
logging.level.org.springframework.web=DEBUG
logging.level.o
```



#### Log Groups

It’s often useful to be able to group related loggers together so that they can all be configured at the same time. For example, you might commonly change the logging levels for *all* Tomcat related loggers, but you can’t easily remember top level packages.

To help with this, Spring Boot allows you to define logging groups in your Spring `Environment`. For example, here’s how you could define a “tomcat” group by adding it to your `application.properties`:

```
logging.group.tomcat=org.apache.catalina, org.apache.coyote, org.apache.tomcat
```

Once defined, you can change the level for all the loggers in the group with a single line:

```
logging.level.tomcat=TRACE
```

Spring Boot includes the following pre-defined logging groups that can be used out-of-the-box:

| Name | Loggers                                                      |
| ---- | ------------------------------------------------------------ |
| web  | `org.springframework.core.codec`, `org.springframework.http`, `org.springframework.web` |
| sql  | `org.springframework.jdbc.core`, `org.hibernate.SQL`         |

#### Custom Log Configuration



```xml
    <!-- 彩色日志 -->
    <!-- 彩色日志依赖的渲染类 -->
    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />
    <conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter" />
    <conversionRule conversionWord="wEx"
                    converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter" />
    <!-- 彩色日志格式 -->
    <property name="CONSOLE_LOG_PATTERN"
              value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}" />
    <!-- Console 输出设置 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>utf8</charset>
        </encoder>
    </appender>
```



## log4j2



相比与其他的日志系统，log4j2丢数据这种情况少；disruptor技术，在多线程环境下，性能高于logback等10倍以上；利用jdk1.5并发的特性，减少了死锁的发生；



### pom.xml



```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-web</artifactId>  
    <exclusions>
        <!-- 去掉springboot默认配置 -->  
        <exclusion>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-logging</artifactId>  
        </exclusion>  
    </exclusions>  
</dependency> 


<dependency> 
    <!-- 引入log4j2依赖 -->  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-log4j2</artifactId>  
</dependency> 


<dependency>  
    <!-- 加上这个才能辨认到log4j2.yml文件 -->
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-yaml</artifactId>
</dependency>

```



### 配置文件

如果自定义了文件名，需要在 bootstrap.yml 中配置



```yml
logging:
  config: classpath:xxxx.xml
  level:
    cn.jay.repository: trace
```

默认名 log4j2-spring.xml，就省下了在 application.yml 中配置



### 配置文件模版

log4j 是通过一个 .properties 的文件作为主配置文件的，而现在的 log4j2 则已经弃用了这种方式，采用的是 .xml，.json 或者 .jsn 这种方式来做，可能这也是技术发展的一个必然性，因为 properties 文件的可阅读性真的是有点差。这里给出博主自配的一个模版，供大家参考。



```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- Configuration 后面的 status，用于设置 log4j2 自身内部的信息输出，-->
<!-- 可以不设置，当设置成 trace 时，你会看到 log4j2 内部各种详细输出 -->
<!-- monitorInterval：Log4j 能够自动检测修改配置文件和重新配置本身，设置间隔秒数 -->
<configuration monitorInterval="5">
  <!-- 日志级别以及优先级排序: -->
<!-- 
OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL 
-->

  <!-- 变量配置 -->
  <Properties>
<!-- 
格式化输出：
%date 表示日期，
%thread表 示线程名，
%-5level：级别从左显示5个字符宽度 
%msg：日志消息，%n是换行符
%logger{36} 表示 Logger 名字最长36个字符 
-->
    <property name="LOG_PATTERN" value="%date{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n" />
    <!-- 定义日志存储的路径，不要配置相对路径 -->
    <property name="FILE_PATH" value="更换为你的日志路径" />
    <property name="FILE_NAME" value="更换为你的项目名" />
  </Properties>

  <appenders>

    <console name="Console" target="SYSTEM_OUT">
      <!--输出日志的格式-->
      <PatternLayout pattern="${LOG_PATTERN}"/>
      <!--控制台只输出level及其以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
      <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
    </console>

    <!--文件会打印出所有信息，这个log每次运行程序会自动清空，由append属性决定，适合临时测试用-->
    <File name="Filelog" fileName="${FILE_PATH}/test.log" append="false">
      <PatternLayout pattern="${LOG_PATTERN}"/>
    </File>

    <!-- 
这个会打印出所有的info及以下级别的信息，每次大小超过size，
则这size大小的日志会自动存入按 年份-月份 建立的文件夹下面并进行压缩，
作为存档-->
    <RollingFile name="RollingFileInfo" fileName="${FILE_PATH}/info.log" filePattern="${FILE_PATH}/${FILE_NAME}-INFO-%d{yyyy-MM-dd}_%i.log.gz">
      <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
      <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
      <PatternLayout pattern="${LOG_PATTERN}"/>
      <Policies>
        <!--interval属性用来指定多久滚动一次，默认是1 hour-->
        <TimeBasedTriggeringPolicy interval="1"/>
        <SizeBasedTriggeringPolicy size="10MB"/>
      </Policies>
      <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件开始覆盖-->
      <DefaultRolloverStrategy max="15"/>
    </RollingFile>

    <!-- 
这个会打印出所有的warn及以下级别的信息，每次大小超过size，
则这size大小的日志会自动存入按 年份-月份 建立的文件夹下面并进行压缩，
作为存档-->
    <RollingFile name="RollingFileWarn" fileName="${FILE_PATH}/warn.log" filePattern="${FILE_PATH}/${FILE_NAME}-WARN-%d{yyyy-MM-dd}_%i.log.gz">
      <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
      <ThresholdFilter level="warn" onMatch="ACCEPT" onMismatch="DENY"/>
      <PatternLayout pattern="${LOG_PATTERN}"/>
      <Policies>
        <!--interval属性用来指定多久滚动一次，默认是1 hour-->
        <TimeBasedTriggeringPolicy interval="1"/>
        <SizeBasedTriggeringPolicy size="10MB"/>
      </Policies>
      <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件开始覆盖-->
      <DefaultRolloverStrategy max="15"/>
    </RollingFile>

    <!-- 
这个会打印出所有的error及以下级别的信息，每次大小超过size，
则这size大小的日志会自动存入按 年份-月份 建立的文件夹下面并进行压缩，
作为存档-->
    <RollingFile name="RollingFileError" fileName="${FILE_PATH}/error.log" filePattern="${FILE_PATH}/${FILE_NAME}-ERROR-%d{yyyy-MM-dd}_%i.log.gz">
      <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
      <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY"/>
      <PatternLayout pattern="${LOG_PATTERN}"/>
      <Policies>
        <!--interval属性用来指定多久滚动一次，默认是1 hour-->
        <TimeBasedTriggeringPolicy interval="1"/>
        <SizeBasedTriggeringPolicy size="10MB"/>
      </Policies>
      <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件开始覆盖-->
      <DefaultRolloverStrategy max="15"/>
    </RollingFile>

  </appenders>

  <!--Logger节点用来单独指定日志的形式，比如要为指定包下的class指定不同的日志级别等。-->
  <!--然后定义loggers，只有定义了logger并引入的appender，appender才会生效-->
  <loggers>

    <!--过滤掉spring和mybatis的一些无用的DEBUG信息-->
    <logger name="org.mybatis" level="info" additivity="false">
      <AppenderRef ref="Console"/>
    </logger>
    <!--监控系统信息-->
    <!--若是additivity设为false，则 子Logger 只会在自己的appender里输出，而不会在 父Logger 的appender里输出。-->
    <Logger name="org.springframework" level="info" additivity="false">
      <AppenderRef ref="Console"/>
    </Logger>

    <root level="info">
      <appender-ref ref="Console"/>
      <appender-ref ref="Filelog"/>
      <appender-ref ref="RollingFileInfo"/>
      <appender-ref ref="RollingFileWarn"/>
      <appender-ref ref="RollingFileError"/>
    </root>
  </loggers>

</configuration>

```



### 配置参数简介

在这里简单介绍下常用的配置参数

#### 日志级别

> 机制：如果一条日志信息的级别大于等于配置文件的级别，就记录。

trace：追踪，就是程序推进一下，可以写个trace输出
debug：调试，一般作为最低级别，trace基本不用。
info：输出重要的信息，使用较多
warn：警告，有些信息不是错误信息，但也要给程序员一些提示。
error：错误信息。用的也很多。
fatal：致命错误。

#### 输出源

CONSOLE（输出到控制台）
FILE（输出到文件）

#### 格式

SimpleLayout：以简单的形式显示
HTMLLayout：以HTML表格显示
PatternLayout：自定义形式显示
PatternLayout：自定义日志布局：

```bash
%d{yyyy-MM-dd HH:mm:ss, SSS} : 日志生产时间,输出到毫秒的时间
%-5level : 输出日志级别，-5表示左对齐并且固定输出5个字符，如果不足在右边补0
%c : logger的名称(%logger)
%t : 输出当前线程名称
%p : 日志输出格式
%m : 日志内容，即 logger.info("message")
%n : 换行符
%C : Java类名(%F)
%L : 行号
%M : 方法名
%l : 输出语句所在的行数, 包括类名、方法名、文件名、行数
hostName : 本地机器名
hostAddress : 本地ip地址
```



### Log4j2配置详解

#### 根节点Configuration

有两个属性:

- status
- monitorinterval

有两个子节点:

- Appenders
- Loggers (表明可以定义多个Appender和Logger).



status 用来指定log4j本身的打印日志的级别.

monitorinterva l用于指定log4j自动重新配置的监测间隔时间，单位是s,最小是5s.



#### Appenders节点



常见的有三种子节点: `Console`、`RollingFile`、`File`

**Console 节点用来定义输出到控制台的 Appender.**

name：指定 Appender 的名字.
target：SYSTEM_OUT 或 SYSTEM_ERR,一般只设置默认: SYSTEM_OUT.
PatternLayout：输出格式，不设置默认为: `%m%n`.

**File 节点用来定义输出到指定位置的文件的 Appender.**

name：指定 Appender 的名字.
fileName：指定输出日志的目的文件带全路径的文件名.
PatternLayout：输出格式，不设置默认为: `%m%n`.

**RollingFile 节点用来定义超过指定条件自动删除旧的创建新的 Appender.**

name：指定Appender的名字.
fileName：指定输出日志的目的文件带全路径的文件名.
PatternLayout：输出格式，不设置默认为:%m%n.
filePattern：指定当发生Rolling时，文件的转移和重命名规则.
Policies：指定滚动日志的策略，就是什么时候进行新建日志文件输出日志.
TimeBasedTriggeringPolicy：Policies子节点，基于时间的滚动策略，interval属性用来指定多久滚动一次，默认是1   hour。modulate=true用来调整时间：比如现在是早上3am，interval是4，那么第一次滚动是在4am，接着是8am，12am…而不是7am.
SizeBasedTriggeringPolicy：Policies子节点，基于指定文件大小的滚动策略，size属性用来定义每个日志文件的大小.
DefaultRolloverStrategy：用来指定同一个文件夹下最多有几个日志文件时开始删除最旧的，创建新的(通过max属性)。



#### Loggers节点

常见的有两种: Root 和 Logger.

> Root节点用来指定项目的根日志，如果没有单独指定Logger，那么就会默认使用该Root日志输出

level:日志输出级别，共有8个级别，按照从低到高为：All  &lt; Trace &lt; Debug &lt; Info &lt; Warn &lt; Error &lt; Fatal &lt; Off

**AppenderRef：Root 的子节点**

用来指定该日志输出到哪个Appender.

- Logger 节点用来单独指定日志的形式，比如要为指定包下的class指定不同的日志级别等。
- level:日志输出级别，共有8个级别，按照从低到高为：All  &lt; Trace &lt; Debug &lt; Info &lt; Warn &lt; Error &lt; Fatal &lt; Off
- name:用来指定该Logger所适用的类或者类所在的包全路径,继承自Root节点.

**AppenderRef：Logger 的子节点**

用来指定该日志输出到哪个Appender，如果没有指定，就会默认继承自 Root。如果指定了，那么会在指定的这个 Appender 和 Root 的 Appender 中都会输出，此时我们可以设置 Logger 的 `additivity="false"` 只在自定义的 Appender 中进行输出。





参考：

-  [spring-boot-(2.1.2)-doc.26-Logging](https://docs.spring.io/spring-boot/docs/2.1.2.RELEASE/reference/htmlsingle/#boot-features-logging-format)
- [[让你的spring-boot应用日志随心所欲--spring boot日志深入分析](https://www.cnblogs.com/davidwang456/p/10997038.html)](https://www.cnblogs.com/davidwang456/p/10997038.html)
-  [Springboot整合log4j2日志全解](https://zhuanlan.zhihu.com/p/70090008)
-  [[Spring Boot 的彩色日志](https://www.cnblogs.com/SummerinShire/p/7826193.html)](https://www.cnblogs.com/SummerinShire/p/7826193.html)
-  [Spring Boot之Log4j2配置（总结）](https://blog.csdn.net/ahou2468/article/details/80486409)



<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
