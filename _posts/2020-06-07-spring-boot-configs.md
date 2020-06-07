---
title: Spring Boot 读取配置文件
key: 2020-06-07
tags: spring spring-boot 
---





记录如何在开发 Spring Boot 程序时，从配置文件中读取自定义配置信息。



<!--more-->



## 1. 使用 Value annotation

```java
@Value("${username}")
String userName;
```

如果不加上 `${}`，注入的就是原始值。




## 2. 使用 Environment

EnvironmentDemo.java:

```java
package luxe.chaos.demo.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.env.Environment;
import org.springframework.stereotype.Component;

@Component
public class EnvironmentDemo {

    private Environment environment;

    @Autowired
    public void setEnvironment(Environment environment) {
        this.environment = environment;
    }

    private String serverName;

    public String getServerName() {
        if (serverName == null) {
            serverName = environment.getProperty("spring.jpa.database-platform");
        }
        return serverName;
    }
}

```

在 Spring Boot 2.0  中对配置属性加载的时候会除了像 1.x 版本时候那样移除特殊字符外，还会将配置均以全小写的方式进行匹配和加载。所以，下面的 4 种配置方式都是等价的：

```properties
spring.jpa.databaseplatform=mysql
spring.jpa.database-platform=mysql
spring.jpa.databasePlatform=mysql
spring.JPA.database_platform=mysql
```



## 3. 使用 ConfigurationProperties 读取并与 Bean 绑定

配置文件:

```yml
project:
  report:
    server-name: report-server
    hosts:
      - host: 192.168.1.11
        port: 8080
      - host: 192.168.2.11
        port: 8081
      - host: 192.168.3.33
        port: 8801
```

java 代码: ReportProperties 

```java
package luxe.chaos.demo.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.List;

@Component
@ConfigurationProperties(prefix = "project.report")
public class ReportProperties {

    private String serverName;

    private List<Hosts> hosts;

    public static class Hosts {
        String host;
        Integer port;

        public String getHost() {
            return host;
        }

        public void setHost(String host) {
            this.host = host;
        }

        public Integer getPort() {
            return port;
        }

        public void setPort(Integer port) {
            this.port = port;
        }
    }

    public String getServerName() {
        return serverName;
    }

    public void setServerName(String serverName) {
        this.serverName = serverName;
    }

    public List<Hosts> getHosts() {
        return hosts;
    }

    public void setHosts(List<Hosts> hosts) {
        this.hosts = hosts;
    }
}
```



## 4. 从其他配置文件中读取

使用 `@PropertySource` Annotation 和 `@Value` Annotation.

一个例子: 

my-website.properties

```properties
url=https://www.chengchaos.cn
```
MyWebsite.java

```java
package luxe.chaos.demo.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;

@Component
@PropertySource("classpath:my-website.properties")
public class MyWebsite {

    @Value("${url}")
    private String url;

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }
}

```


java 代码: DemoServer01App 

```java
package luxe.chaos.demo;

import luxe.chaos.demo.config.EnvironmentDemo;
import luxe.chaos.demo.config.MyWebsite;
import luxe.chaos.demo.config.ReportProperties;
import luxe.chaos.demo.grpc.HelloWorldServer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

@SpringBootApplication
public class DemoServer01App {

    private static final Logger LOGGER = LoggerFactory.getLogger(DemoServer01App.class);


    public static void main(String[] args) throws Exception {
        ConfigurableApplicationContext ctx = SpringApplication.run(DemoServer01App.class, args);
        LOGGER.info("it worded!");

        ReportProperties report = ctx.getBean(ReportProperties.class);

        report.getHosts()
                .forEach(host -> LOGGER.info(">>> {}:{}", host.getHost(), host.getPort()));

        MyWebsite myWebsite = ctx.getBean(MyWebsite.class);
        LOGGER.info(">>> my-website url -=> {}", myWebsite.getUrl());

        EnvironmentDemo environmentDemo = ctx.getBean(EnvironmentDemo.class);

        String serverName = environmentDemo.getServerName();

        LOGGER.info(">>> server-name -=> {}", serverName);

        new Thread(HelloWorldServer::serverStart);

    }
}

```

.

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
