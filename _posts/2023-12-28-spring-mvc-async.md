---
title: Spring MVC 中的异步支持
key: 2023-12-28
tags: Spring Cloud Asynchronized
---

不管未来会怎样，眼下 Java 开发还是我的饭碗，在有时间的时候，整理一下这些内容免得记得。

<!--more-->

## 0x01 使用 @Async 

```java
@Configuration
@EnableAsync
public class AsyncConfig {

}
```
## 参考和抄袭:

- [spring-cloud-circuitbreaker-filter-factor](https://cloud.spring.io/spring-cloud-gateway/reference/html/#spring-cloud-circuitbreaker-filter-factory)
- [spring-cloud-circuitbreaker](https://cloud.spring.io/spring-cloud-circuitbreaker/reference/html/spring-cloud-circuitbreaker.html)
- [https://resilience4j.readme.io/docs/micrometer](https://resilience4j.readme.io/docs/micrometer)
- [Resilience4j-CircuitBreaker详解](https://www.jianshu.com/p/b6a3bf63be89)


