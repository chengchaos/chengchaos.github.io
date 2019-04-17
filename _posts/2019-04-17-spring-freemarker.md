---
title: Spring-boot 使用 Freemarkar
key: 20190417
tags: spring-boot freemarker
---

全文参考： [spring boot 配置 freemarker](https://www.cnblogs.com/jtlgb/p/8548436.html)

<!--more-->

## pom.xml

```xml
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.4.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>cn.futuremove</groupId>
	<artifactId>mock-tbox</artifactId>
	<version>0.0.1-SNAPSHOT</version>
    <packaging>war</packaging>

	<name>mock-tbox</name>
	<description>Demo project for Spring Boot</description>

	<properties>

		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>

        <java.version>1.8</java.version>
        <scala.version>2.11.12</scala.version>
        <spring-boot.version>2.1.4.RELEASE</spring-boot.version>

        <!-- 开罗 -->
        <platform-bom.version>Cairo-SR5</platform-bom.version>
        <!-- 芬奇利 -->
        <spring-cloud.version>Finchley.SR2</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-freemarker</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
	</dependencies>
```

## application.yml

```yml

server:
  port: 8081
spring:
  application:
    name: mock-tbox
  cloud:
    service-registry:
      auto-registration:
        enabled: false

  freemarker:
    # req访问 request
    request-context-attribute: req
    suffix: .htm
    content-type: text/html
    enabled: true
    cache: false
    # 模板加载路径 按需配置
    template-loader-path:
      - classpath:/templates
    charset: UTF-8
    settings:
      number_format: '0.##'


management:
  endpoints:
    web:
      exposure:
        include: '*'
    enabled-by-default: false
  endpoint:
    beans:
      enabled: true
    configprops:
      enabled: true
    env:
      enabled: true
    health:
      enabled: true
      show-details: always
    mappings:
      enabled: true


```

## 模板文件位置

```
+ resources
| + templates
| - - echo.htm
```

## Controller

```java
package cn.futuremove.mocktbox.controller;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.context.request.async.WebAsyncTask;

import javax.servlet.http.HttpServletRequest;

@Controller
public class EchoController {


    private static final Logger LOGGER = LoggerFactory.getLogger(EchoController.class);

    private static final long TIMEOUT = 5000L;

    @GetMapping(value="/echo")
    public WebAsyncTask<String> echo(HttpServletRequest request) {

        return new WebAsyncTask<>(TIMEOUT, ()->{

            LOGGER.info(" ... ");

            request.setAttribute("name", "程超");
            return "echo";
        });

    }

}

```

## 模板文件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
嗨：${name}
</body>
</html>
```

## freemarker 的配置 （propertity格式配置文件）

```bash
 # FREEMARKER (FreeMarkerAutoConfiguration)
spring.freemarker.allow-request-override=false # Set whether HttpServletRequest attributes are allowed to override (hide) controller generated model attributes of the same name.
spring.freemarker.allow-session-override=false # Set whether HttpSession attributes are allowed to override (hide) controller generated model attributes of the same name.
spring.freemarker.cache=false # Enable template caching.
spring.freemarker.charset=UTF-8 # Template encoding.
spring.freemarker.check-template-location=true # Check that the templates location exists.
spring.freemarker.content-type=text/html # Content-Type value.
spring.freemarker.enabled=true # Enable MVC view resolution for this technology.
spring.freemarker.expose-request-attributes=false # Set whether all request attributes should be added to the model prior to merging with the template.
spring.freemarker.expose-session-attributes=false # Set whether all HttpSession attributes should be added to the model prior to merging with the template.
spring.freemarker.expose-spring-macro-helpers=true # Set whether to expose a RequestContext for use by Spring's macro library, under the name "springMacroRequestContext".
spring.freemarker.prefer-file-system-access=true # Prefer file system access for template loading. File system access enables hot detection of template changes.
spring.freemarker.prefix= # Prefix that gets prepended to view names when building a URL.
spring.freemarker.request-context-attribute= # Name of the RequestContext attribute for all views.
spring.freemarker.settings.*= # Well-known FreeMarker keys which will be passed to FreeMarker's Configuration.
spring.freemarker.suffix= # Suffix that gets appended to view names when building a URL.
spring.freemarker.template-loader-path=classpath:/templates/ # Comma-separated list of template paths.
spring.freemarker.view-names= # White list of view names that can be resolved.
```


另外需要使用thymeleaf 可以使用如下配置（yml 格式配置文件方式）

```yml
##视图模型
spring:
  thymeleaf:
    prefix: classpath:/templates/
    suffix: .html
    cache: false
    encoding: utf-8
    content-type: text/html
    check-template-location: true
```

## 最后

pom 文件中标记为 war 包。


<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
