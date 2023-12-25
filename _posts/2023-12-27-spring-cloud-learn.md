---
title: Spring Cloud Gateway 熔断降级
tags: Spring Cloud Gateway
article_header:
  type: cover
  image:
    src: /screenshot.jpg
---

一转眼 2023 年就要过去了，今天查了一下冬至是哪天，原来是 22 号上个周五，也已经过去了。趁着还有一点时间，把 spring 的熔断和降级复习复习。

<!--more-->

## 0x01 概念

服务降级：一般指在服务器压力剧增的时候，根据实际业务使用情况，对一些服务和页面有策略的不处理或者用一种简单的方式进行处理，从而释放服务器的资源以保证核心业务的正常高效运行。

服务降级是从整个系统的负荷情况出发和考虑的，为了预防某些功能出现负荷过载或者响应变慢的情况，在内部暂时舍弃一些非核心接口和数据的请求，而直接返回一个提前准备好的 `fallback` 信息。这样虽然提供的是一个有损的服务，但却可以保护了整个系统的稳定和可用性。

服务降级需要考虑：

- 那些为核心服务，哪些不是
- 降级策略
- 自动/手动降级

服务熔断：是应对为服务雪崩效应的一种链路保护机制。当调用链中的某个服务不可用或者响应时间太长，会进行服务熔断，不再对该节点服务调用，快速返回错误的响应信息，当检测到该节点调用正常后，再恢复调用链路。

Spring Cloud 框架里，熔断机制是通过 `Hystrix` 实现的。Hystrix 会监控为服务之间的调用状况，当失败的调用到一定的阈值（默认 20次/5秒）就会启动熔断机制。

服务熔断需要考虑：

- 如何判断服务变得不稳定
- 如何探知服务是否恢复

服务降级和服务熔断的区别

- 触发原因： 服务熔断是链路上摸个服务引起的，服务降级是从整体负载情况考虑的。
- 管理目标： 服务熔断是一个框架层级的处理，服务降级是业务层级的处理。
- 实现方式： 服务熔断通常自动恢复，服务降级通常人工控制。

## 0x02 示例

以项目中使用 spring cloud 版本为例： 

```xml
    <properties>
        <spring.bood-version>3.0.12</spring.bood-version>
        <spring-cloud.version>2022.0.0</spring-cloud.version>
        <java.version>17</java.version>
    </properties>
```

Step 1, 添加依赖：

```xml
    <dependencyManagement>
        <dependencies>
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
            <artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
        </dependency>
    </dependencies>
```

Step 2, application.yml 配置

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: web-flux-demo-2
          predicates:
            - Path=/demo/**
          uri: http://localhost:8080/
          filters:
            - StripPrefix=1
            # 降级配置
            - name: CircuitBreaker
              args:
                name: testOne
                # 降级接口地址
                fallbackUri: forward:/v1/fallback
```

Step 3, 在 Gateway 中实现一个熔断的 fallback 接口

```java
@RestController
public class Call1FallbackController {

    @GetMapping("/v1/fallback")
    public Map<String, String> fallback() {

        Map<String, String> result = new HashMap<>();
        result.put("code", "error");
        result.put("msg", "What the fuck can I do!");
        return result;
    }
}
```

Step  4, 定制

```java
package c.b.cheng.wfd2.config;

import io.github.resilience4j.circuitbreaker.CircuitBreakerConfig;
import io.github.resilience4j.timelimiter.TimeLimiterConfig;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cloud.circuitbreaker.resilience4j.ReactiveResilience4JCircuitBreakerFactory;
import org.springframework.cloud.circuitbreaker.resilience4j.Resilience4JConfigBuilder;
import org.springframework.cloud.client.circuitbreaker.Customizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.time.Duration;

/**
 * <p>
 * <strong>
 * Describe the function in one sentence.
 * </strong><br /><br />
 * As the title says.
 * </p>
 *
 * @author Cheng, Chao - 2023/12/26 00:17 <br />
 * @see Object
 * @since 1.0
 */
@Configuration
public class Resilience4JCircuitBreakerConfig {

    private static final Logger LOGGER = LoggerFactory.getLogger(Resilience4JCircuitBreakerConfig.class);

    @Bean
    public Customizer<ReactiveResilience4JCircuitBreakerFactory> defaultCustomizer() {
        LOGGER.info(">>>>>>>>> ");
        return factory -> factory.configureDefault(id -> new Resilience4JConfigBuilder(id)
                .timeLimiterConfig(
                        TimeLimiterConfig
                                .custom()
                                .timeoutDuration(Duration.ofSeconds(4))
                                .build())
                .circuitBreakerConfig(CircuitBreakerConfig.ofDefaults())
                .build());
    }
}

```

参考和抄袭:

- [spring-cloud-circuitbreaker-filter-factor](https://cloud.spring.io/spring-cloud-gateway/reference/html/#spring-cloud-circuitbreaker-filter-factory)
- [spring-cloud-circuitbreaker](https://cloud.spring.io/spring-cloud-circuitbreaker/reference/html/spring-cloud-circuitbreaker.html)
- [https://resilience4j.readme.io/docs/micrometer](https://resilience4j.readme.io/docs/micrometer)
- [Resilience4j-CircuitBreaker详解](https://www.jianshu.com/p/b6a3bf63be89)


