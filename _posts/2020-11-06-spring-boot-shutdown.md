---
title: Springboot 优雅停止服务的几种方法
key: 2020-11-06
tags: spring-boot
---

在使用 SpringBoot 的时候，都要涉及到服务的停止和启动，当我们停止服务的时候，很多时候大家都是 kill -9 直接把程序进程杀掉，这样程序不会执行优雅的关闭。而且一些没有执行完的程序就会直接退出。

我们很多时候都需要安全的将服务停止，也就是把没有处理完的工作继续处理完成。比如停止一些依赖的服务，输出一些日志，发一些信号给其他的应用系统，这个在保证系统的高可用是非常有必要的。那么咱么就来看一下几种停止 SpringBoot 的方法。

<!--more-->

## 1, 使用 Actuator

第一种就是 Springboot 提供的 actuator 的功能，它可以执行 shutdown, health, info 等，默认情况下，actuator 的 shutdown 是 disable 的，我们需要打开它。首先引入 acturator 的 maven 依赖。

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

然后将 shutdown 节点打开，也将 /actuator/shutdown 暴露 web 访问也设置上，除了 shutdown 之外还有 health, info 的web访问都打开的话将 `management.endpoints.web.exposure.include=*` 就可以。将如下配置设置到application.yml 里边。设置一下服务的端口号为 8081。

```yml
spring:
  devtools:
    add-properties: true

server:
  port: 8081

management:
  endpoints:
    web:
      exposure:
        include: shutdown
  endpoint:
    shutdown:
      enabled: true
```

然后在 Spring boot 的工程中添加一个 ShutDownConfig 类：

```java
package luxe.chaos.train.springboottrain.config;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

@Configuration
public class ShutDownConfig {

    private static final Logger LOGGER = LoggerFactory.getLogger(ShutDownConfig.class);

    @Bean
    public TerminateBean terminateBean() {
        return new TerminateBean();
    }

    public static class TerminateBean {

        @PostConstruct
        public void postConstruct() {
            LOGGER.warn("TerminateBean is Construct! ");
        }
        @PreDestroy
        public void preDestroy() {
            LOGGER.warn("TerminateBean is Destroyed! ");
        }
    }
}

```

启动工程后，执行 ：

```bash
curl -X POST http://localhost:8081/actuator/shutdown
```

可以看到日志已经被打印出来了。

```log

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.3.5.RELEASE)

10:55:35.551 + INFO  l.c.t.s.SpringBootTrainApplication -line 55 - Starting SpringBootTrainApplication on CPX-GBXKB031GX5 with PID 18608 (C:\works\git-repo\github.com\chengchaos\spring-cloud-demo\spring-boot-train\target\classes started by c.b.cheng in C:\works\git-repo\github.com\chengchaos\spring-cloud-demo\spring-boot-train) - [restartedMain]
10:55:35.553 + INFO  l.c.t.s.SpringBootTrainApplication -line 651 - No active profile set, falling back to default profiles: default - [restartedMain]
10:55:35.596 + INFO  o.s.b.d.e.DevToolsPropertyDefaultsPostProcessor -line 225 - Devtools property defaults active! Set 'spring.devtools.add-properties' to 'false' to disable - [restartedMain]
10:55:35.596 + INFO  o.s.b.d.e.DevToolsPropertyDefaultsPostProcessor -line 225 - For additional web related logging consider setting the 'logging.level.web' property to 'DEBUG' - [restartedMain]
10:55:36.888 + INFO  o.s.b.w.e.t.TomcatWebServer -line 108 - Tomcat initialized with port(s): 8081 (http) - [restartedMain]
10:55:36.897 + INFO  o.a.c.c.StandardService -line 173 - Starting service [Tomcat] - [restartedMain]
10:55:36.897 + INFO  o.a.c.c.StandardEngine -line 173 - Starting Servlet engine: [Apache Tomcat/9.0.39] - [restartedMain]
10:55:36.968 + INFO  o.a.c.c.C.[.[.[/] -line 173 - Initializing Spring embedded WebApplicationContext - [restartedMain]
10:55:36.968 + INFO  o.s.b.w.s.c.ServletWebServerApplicationContext -line 285 - Root WebApplicationContext: initialization completed in 1372 ms - [restartedMain]
10:55:37.120 + INFO  o.s.s.c.ThreadPoolTaskExecutor -line 181 - Initializing ExecutorService - [restartedMain]
10:55:37.121 + INFO  o.s.s.c.ThreadPoolTaskExecutor -line 181 - Initializing ExecutorService 'threadPoolTaskExecutor' - [restartedMain]
10:55:37.126 + WARN  l.c.t.s.c.ShutDownConfig -line 27 - TerminateBean is Construct!  - [restartedMain]
10:55:37.215 + INFO  l.c.t.s.c.AsyncSupportWebMvcConfigurer -line 43 - Configure async support ... set default timeout 30 Seconds. - [restartedMain]
10:55:37.371 + INFO  o.s.b.d.a.OptionalLiveReloadServer -line 58 - LiveReload server is running on port 35729 - [restartedMain]
10:55:37.376 + INFO  o.s.b.a.e.w.EndpointLinksResolver -line 58 - Exposing 1 endpoint(s) beneath base path '/actuator' - [restartedMain]
10:55:37.411 + INFO  o.s.b.w.e.t.TomcatWebServer -line 220 - Tomcat started on port(s): 8081 (http) with context path '' - [restartedMain]
10:55:37.422 + INFO  l.c.t.s.SpringBootTrainApplication -line 61 - Started SpringBootTrainApplication in 2.293 seconds (JVM running for 2.734) - [restartedMain]
10:55:47.762 + INFO  o.a.c.c.C.[.[.[/] -line 173 - Initializing Spring DispatcherServlet 'dispatcherServlet' - [http-nio-8081-exec-1]
10:55:47.762 + INFO  o.s.w.s.DispatcherServlet -line 525 - Initializing Servlet 'dispatcherServlet' - [http-nio-8081-exec-1]
10:55:47.767 + INFO  o.s.w.s.DispatcherServlet -line 547 - Completed initialization in 5 ms - [http-nio-8081-exec-1]
10:55:48.663 + WARN  l.c.t.s.c.ShutDownConfig -line 31 - TerminateBean is Destroyed!  - [Thread-8]
10:55:48.663 + INFO  o.s.s.c.ThreadPoolTaskExecutor -line 218 - Shutting down ExecutorService 'threadPoolTaskExecutor' - [Thread-8]

Process finished with exit code 1

```

## 2, 使用 ConfigurableApplicationContext.close() 方法

第二种方法也比较简单，获取程序启动时候的context，然后关闭主程序启动时的context。这样程序在关闭的时候也会调用PreDestroy注解。如下方法在程序启动十秒后进行关闭。

```java
package luxe.chaos.train.springboottrain;

import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

import java.util.concurrent.TimeUnit;

@SpringBootApplication
public class SpringBootTrainApplication {

    private static final Logger LOGGER = LoggerFactory.getLogger(SpringBootTrainApplication.class);


    public static void main(String[] args) {
        method1(args);
//        method2(args);
//       method3(args);
    }

    private static void method1(String[] args) {
        ConfigurableApplicationContext ctx = SpringApplication.run(SpringBootTrainApplication.class, args);

        try {
            TimeUnit.SECONDS.sleep(10L);
        } catch (InterruptedException e) {
            LOGGER.error(StringUtils.EMPTY, e);
        }

        ctx.close();
    }

}

```

## 3, 使用 pid 文件

在 springboot 启动的时候将进程号写入一个 app.pid 文件，生成的路径是可以指定的，可以通过命令

```bash
cat /Users/huangqingshi/app.id | xargs kill
```

命令直接停止服务，这个时候 bean 对象的 `PreDestroy` 方法也会调用的。这种方法大家使用的比较普遍。写一个 start.sh 用于启动 springboot 程序，然后写一个停止程序将服务停止。　　

```java
    /* generate a pid in a specified path, while use command to shutdown pid :
     * 'cat /Users/huangqingshi/app.pid | xargs kill'
     */
    private static void method2(String[] args) {
        SpringApplication application = new SpringApplication(SpringBootTrainApplication.class);
        application.addListeners(new ApplicationPidFileWriter("c:/works/temp/app.pid"));
        application.run();
    }
```

## 4, 使用 SpringApplication.exit() 方法

通过调用一个 `SpringApplication.exit()` 方法也可以退出程序，同时将生成一个退出码，这个退出码可以传递给所有的 context。这个就是一个 JVM 的钩子，通过调用这个方法的话会把所有 `PreDestroy` 的方法执行并停止，并且传递给具体的退出码给所有 Context。通过调用 `System.exit(exitCode)` 可以将这个错误码也传给 JVM。程序执行完后最后会输出：Process finished with exit code 0，给JVM一个SIGNAL。

```java
    private static void method3(String[] args) {
        ConfigurableApplicationContext ctx = SpringApplication.run(SpringBootTrainApplication.class, args);

        try {
            TimeUnit.SECONDS.sleep(10L);
        } catch (InterruptedException e) {
            LOGGER.error(StringUtils.EMPTY, e);
        }

        exitApplication(ctx);
    }

    private static void exitApplication(ConfigurableApplicationContext context) {
        int exitCode = SpringApplication.exit(context, (ExitCodeGenerator) () -> 0);
        System.exit(exitCode);
    }
```

## 5, 自己写 Controller

```java
package com.hqs.springboot.shutdowndemo.controller;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author huangqingshi
 * @Date 2019-08-17
 */
@RestController
public class ShutDownController implements ApplicationContextAware {

    private ApplicationContext context;

    @PostMapping("/shutDownContext")
    public String shutDownContext() {
        ConfigurableApplicationContext ctx = (ConfigurableApplicationContext) context;
        ctx.close();
        return "context is shutdown";
    }

    @GetMapping("/")
    public String getIndex() {
        return "OK";
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        context = applicationContext;
    }
}
```

参考：

- [Springboot 优雅停止服务的几种方法](www.cnblogs.com/huangqingshi/p/11370291.html)

EOF

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
