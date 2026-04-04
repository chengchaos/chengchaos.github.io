---
title: spring cloud 注册到 Eureka
key: 20190325
tags: spring-boot spring-cloud eureka
---

一个测试，spring-boot 项目注册到 Eureka 中。

<!--more-->



## pom.xml

需要添加的依赖：


```xml

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.3.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>

		<java.version>1.8</java.version>
		<scala.version>2.11.12</scala.version>
		<!--<spring-boot.version>2.0.6.RELEASE</spring-boot.version>-->
		<spring-boot.version>2.0.1.RELEASE</spring-boot.version>

		<!-- 开罗 -->
		<platform-bom.version>Cairo-SR5</platform-bom.version>
		<!-- 芬奇利 -->
		<spring-cloud.version>Finchley.SR2</spring-cloud.version>
	</properties>

	<dependencyManagement>
		<dependencies>
			<!-- spring cloud -->
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>


		</dependencies>
	</dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>
	</dependencies>


	<build>
        <plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

```

## application.yml

```yml


spring:
  application:
    name: my-scala002
  cloud:
    service-registry:
      auto-registration:
        enabled: false
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true

```

## 整合 Ribbon

```xml

```












<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
