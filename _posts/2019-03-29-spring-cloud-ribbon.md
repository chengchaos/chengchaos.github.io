---
title: Spring-Cloud 使用 Ribbon
key: 20190325
tags: spring-boot spring-cloud ribbon
---

记录一下 spring-cloud 中使用 ribbon

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

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
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

RestTemplateConfig

```java

package com.example.myjsp.conf;

import org.apache.http.client.HttpClient;
import org.apache.http.conn.HttpClientConnectionManager;
import org.apache.http.impl.client.HttpClientBuilder;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.commons.httpclient.DefaultApacheHttpClientConnectionManagerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.http.client.ClientHttpRequestFactory;
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;

/**
 * <p>
 * <strong>
 * 用一句话描述功能
 * </strong><br /><br />
 * 功能的详细描述
 * </p>
 *
 * @author chengchaos[as]Administrator - 2019/3/28 0028 下午 9:08 <br />
 * @since 1.1.0
 */
@Configuration
public class RestTemplateConfig {

    @Bean(name="loadBalanced")
    @LoadBalanced
    public RestTemplate loadBalanced() {

        return new RestTemplate(clientHttpRequestFactory());
    }

    @Bean(name="normal")
    @Primary  // 在同样的 DataSource 中，首先使用被标注的 DataSource .
    public RestTemplate normal() {

        return new RestTemplate(clientHttpRequestFactory());
    }


    private HttpClientConnectionManager httpClientConnectionManager() {

        DefaultApacheHttpClientConnectionManagerFactory
                defaultApacheHttpClientConnectionManagerFactory =
                new DefaultApacheHttpClientConnectionManagerFactory();

        return defaultApacheHttpClientConnectionManagerFactory
                .newConnectionManager(true
                        , 500
                        , 50);
    }

    private HttpClient httpClient() {
        HttpClientBuilder httpClientBuilder = HttpClientBuilder.create();
        httpClientBuilder.setConnectionManager(httpClientConnectionManager());
        return httpClientBuilder.build();
    }

    private ClientHttpRequestFactory clientHttpRequestFactory() {
        HttpComponentsClientHttpRequestFactory
                clientHttpRequestFactory =
                new HttpComponentsClientHttpRequestFactory();
        clientHttpRequestFactory.setHttpClient(httpClient());
        clientHttpRequestFactory.setConnectTimeout(30000);
        clientHttpRequestFactory.setReadTimeout(30000);
        return clientHttpRequestFactory;
    }
}

```

PageController

```java
@Controller
public class PageController {


    private static final Logger LOGGER = LoggerFactory.getLogger(PageController.class);

    @Autowired
    @Qualifier(value="loadBalanced")
    private RestTemplate loadBalanced;

    @Autowired
    private RestTemplate restTemplate;


    @GetMapping(value="/v1/some-rest")
    @ResponseBody
    @HystrixCommand(fallbackMethod = "someRestFallback",
    ignoreExceptions = {IllegalArgumentException.class})
    public Map<String, Object> someRest(String name) {

        String url = "http://my-scala002/v1/say-hello?name={name}";

        Map<String, Object> uriVariables = Collections.<String, Object>singletonMap("name", name);

        URI uri = UriComponentsBuilder.fromUriString(url)
                .build()
                .expand(uriVariables)
                .encode()
                .toUri();
        LOGGER.info("uri ==> {}", uri);

        return this.loadBalanced.getForObject(uri, Map.class);
    }

    public Map<String, Object> someRestFallback(String name, Throwable throwable) {

        LOGGER.error("someRestFallback ", throwable);

        Map<String, Object> result = new HashMap<>();

        result.put("name", "what ");
        result.put("time", new Date());

        return result;
    }
}
```


## application.yml

```yml


spring:
  application:
    name: my-scala001
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
ribbon:
  eager-load:
    enabled: true
    clients:
      - MY-SCALA002


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










<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
