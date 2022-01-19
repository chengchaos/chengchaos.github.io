---
title: Spring Cloud Gateway
key: 2021-12-07
tags: spring-cloud-gateway

---

有些事情，你没有办绕过去。

<!--more-->

- 路由（route）。路由是网关最基础的部分，路由信息由一个ID、一个目的url、一组断言工厂和一组Filter组成。如果路由断言为真，则说明请求的url和配置的路由匹配。
- 断言（Predicate）。Java8中的断言函数。SpringCloudGateway中的断言函数输入类型是Spring5.0框架中的ServerWebExchange。SpringCloudGateway中的断言函数允许开发者去定义匹配来自于HttpRequest中的任何信息，比如请求头和参数等。
- 过滤器（filter）。一个标准的SpringwebFilter。SpringCloudGateway中的Filter分为两种类型的Filter，分别是GatewayFilter和GlobalFilter。过滤器Filter将会对请求和响应进行修改处理。


SpringCloudGateway的核心处理流程如图172所示，Gateway的客户端会向SpringCloudGateway发起请求，请求首先会被HttpWebHandlerAdapter进行提取组装成网关的上下文，然后网关的上下文会传递到DispatcherHandler。DispatcherHandler是所有请求的分发处理器，DispatcherHandler主要负责分发请求对应的处理器，比如将请求分发到对应RoutePredicateHandlerMapping（路由断言处理映射器）。路由断言处理映射器主要用于路由的查找，以及找到路由后返回对应的FilteringWebHandler。FilteringWebHandler主要负责组装Filter链表并调用Filter执行一系列的Filter处理，然后把请求转到后端对应的代理服务处理，处理完毕之后，将Response返回到Gateway客户端。

SpringCloudGateway的路由匹配的功能是以SpringWebFlux中的HandlerMapping为基础实现的。SpringCloudGateway也是由许多的路由断言工厂组成的。当HttpRequest请求进入SpringCloudGateway的时候，网关中的路由断言工厂会根据配置的路由规则，对HttpRequest请求进行断言匹配。匹配成功则进行下一步处理，否则断言失败直接返回错误信息。下面我们介绍一下SpringCloudGateway中经常使用的路由断言工厂。

### After 路由断言工厂

After Route Predicate Factory 中会取一个 UTC 时间格式的时间参数，当请求进来的当前时间在配置的UTC时间之后，则会成功匹配，否则不能成功匹配。

### Before 路由断言工厂

Before 路由断言工厂会取一个 UTC 时间格式的时间参数，当请求进来的当前时间在路由断言工厂之前会成功匹配，否则不能成功匹配。

### Between 路由断言工厂

Between 路由断言工厂会取一个UTC时间格式的时间参数，当请求进来的当前时间在配置的UTC时间工厂之间会成功匹配，否则不能成功匹配。

### Cookie 路由断言工厂

Cookie 路由断言工厂会取两个参数——cookie名称对应的key和value。当请求中携带的cookie和Cookied断言工厂中配置的cookie一致，则路由匹配成功，否则匹配不成功。

### Header 路由断言工厂

Header 路由断言工厂用于根据配置的路由header信息进行断言匹配路由，匹配成功进行转发，否则不进行转发。

```java
builder.routes()
.route("header_route", r -> r.header("X-Request-Id", "wokao")
    .uri("http://localhost:8701/test/head/"))
    .build();
```

### Host 路由断言工厂

Host 路由断言工厂根据配置的 Host，对请求中的 Host 进行断言处理，断言成功则进行路由转发，否则不转发。

```java
builder.routes()
.route("host_route", r -> r.host("**.baidu.com")
    .uri("http://www.jd.com"))
    .build();
```

### Method 路由断言工厂

Method 路由断言工厂会根据路由信息配置的 method 对请求方法是 Get 或者 Post 等进行断言匹配，匹配成功则进行转发，否则处理失败。

```java
return builder.routes()
    .route("method_route", r -> r.method("GET").uri("http://jd.com"))
    .build();
```


### Query 路由断言工厂

Query 路由断言工厂会从请求中获取两个参数，将请求中参数和 Query 断言路由中的配置进行匹配，比如 `http://localhost:8080/?foo=bar`中的 `foo=baz` 和下面的 `r.query("foo"，"baz")` 配置一致则转发成功，否则转发失败。

```java
return builder.routes()
    .route("query_route", r -> r.query("foo", "baz").uri("http://jd.com"))
    .build();
```

### REmoteAddr 路由断言工厂

RemoteAddr 路由断言工厂配置一个 IPv4 或 IPv6 网段的字符串或者 IP。当请求 IP 地址在网段之内或者和配置的IP相同，则表示匹配成功，成功转发，否则不能转发。例如 192.1680.1/16 是一个网段，其中 192.1680.1 是 IP 地址，16 是子网掩码，当然也可以直接配置一个 IP。

```java
return builder.routes("remoteaddr_route", r -> r.remoteAddr("127.0.0.1")
    .uri("http://www.jd.com"))
    .build();
```

## Spring Cloud Gateway 的内置 Filter

Spring Cloud Gatewat 中内置很多的路由过滤工厂，当然可以自己根据实际应用场景需要定制的自己的路由过滤器工厂。路由过滤器允许以某种方式修改请求进来的 http 请求或返回的 http 响应。路由过滤器主要作用于需要处理的特定路由。Spring Cloud Gateway 提供了很多种类的过滤器工厂，过滤器的实现类将近二十多个。总得来说，可以分为七类：Header、Parameter、Path、Status、Redirect跳转、Hytrix熔断和 RateLimiter。下面介绍一下 Spring Cloud Gateway 中的 Filter 工厂。

### AddRequestHeader 过滤器工厂

AddRequestHeader 过滤器工厂用于对匹配上的请求加上 header。


```java
return builder.routes()
    .route("add_request_header_route", r -> r.path("/test")
        .filters(f -> f.AddRequestHeader("X-RequestAcme", "ValueB"))
        .uri("http://localhost:8087/test/head"))
    .build();
```

### AddRequestParameter 过滤器

AddRequestParameter 过滤器作用是对匹配上的请求路由添加请求参数。

```java
return builder.routes()
    .route("add_requst_parameter_route", 
        r -> r.path("/AddRequestParameter")
            .filters(f -> f.AddRequestParameter("example", "ValueC"))
            .uri("http://localhost:8701/test/arp"))
    .build();
```

### RewritePath 过滤器

RewritePath 过滤器可以实现 Zuul 的 StripPrefix 功能

```yml
zuul:
  routes:
    demo:
      sensitiveHeaders: Access-Control-Allow-Origin,Access-Control-Allow-Mehtods
        path: /demo/**
        stripPrefix: true
```

这里的 `stripPrefix` 为 true， 表示所有 /demo/xxx 的请求转发给 `http://demo.com/xxx` 去掉 demo 的前缀。

```java
return builder.routers()
    .route("rewritepath_route", r -> r.path("/foo/**")
        .filters(f -> f.rewritePath("/foo/(?<segment>.*)", "/$\\{segment}"))
        .uri("http://www.baidu.com"))
    .build();
```

### AddResponseHeader 过滤器

AddResponseHeader 过滤器工厂的作用是对从网关返回的响应添加 Header。

```java
return builder.routes()
    .route("add_response_header_route", r -> r.path("/test")
        .filters(f -> f.AddResponseHeader("X-Response-Foo", "Bar"))
        .uri("http://www.jd.com"))
    .build();
```
### Prefix 过滤器

- StripPrefixGatewayFilterFactory 是针对请求 url 前缀进行处理的 filter 工厂，用于去除前缀。
- PrefixPathGatewayFilterFactory 是用于增加前缀。

### Retry 过滤器

```java
return builder.routes()
    .route("retry_route", r -> r.path("/test/retry")
        .filters(f -> f.retry(config -> config.setRetries(2)
            .setStatuses(HttpStatus.INTERNAL_SERVER_ERROR)))
        .uri("http://locahost:8762/retry?key=abc&count=2"))
    .build();
```

### Hystrix 过滤器

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

```yml
spring:
  cloud:
    gateway:
      routes:
        - id: prefix_route
          uir: http://localhost:7801/test/hystrix?issleep=true
          predicates:
            - Path=/test/hystrix
          filters:
            - name: Hystrix # Hystrix Filter 的名称
              args: # HyStrix 配置参数
                name: fallbackcmd # HystrixCommand 的名称
                fallbackUri: forward:/fallback # fallback 对应的 uri
# Hystrix 的 fallbackcmd 时间
hystrix.command.fallbackcmd.execution.isolation.thread.timeoutInMilliseconds: 5000
```

其中 `http://localhost:7801/test/hystrix?issleep=true` 是后端服务需要路由熔断处理的 URL。

```java
@RestController
public class FallbackController {
    @GetMapping("/fallback")
    public String fallback() {
        return "Spring Cloud Gateway Fallback";
    }

    @GetMapping("/test/hystrix/")
    public String index(@RequestParam("issleep") boolean issleep) throws InterruptedException {
        logger.info("issleep is {}", issleep);
        // isSleep == true 为开始睡眠，睡眠时间大于 Gateway 中 fallback 设置的时间
        if (isSleep) {
            TimeUnit.MINUTES.sleep(10L);
        }
        return "No Sleep";
    }
}

```

## Gateway 基于服务发现的路由规则

我们知道 Spring Cloud 对 Zuul 进行封装处理之后，当通过 Zuul 访问后端微服务时，基于服务发现的默认路由规则是：`http://zuul_host:zuul_port/微服务在Eureka上的serviceId/**`，Spring Cloud Gateway 在设计的时候考虑从 Zuul 迁移到 Gateway 的兼容性和迁移成本等，Gateway 基于服务发现的路由规则和 Zuul 的设计类似，但是也有很大差别。但是 Spring Cloud Gateway 基于服务发现的路由规则，在不同注册中心下其差异如下：

- 如果把 Gateway 注册到 Eureka 上，通过网关转发服务调用，访问网关的 URL 是 `http://Gateway_HOST:Gateway_PORT/大写的serviceId/*`，其中服务名默认必须是大写，否则会抛404错误，如果服务名要用小写访问，可以在属性配置文件里面加 `spring.cloud.gateway.discovery.locator.lowerCaseServiceId=true` 配置解决。
- 如果把 Gateway 注册到 Zookeeper 上，通过网关转发服务调用，服务名默认小写，因此不需要做任何处理。
- 如果把 Gateway 注册到 Consul 上，通过网关转发服务调用，服务名默认小写，也不需要做人为修改。


主要配置：

- `spring.cloud.gateway.discovery.locator.enabled` ：是否与服务发现组件进行结合，通过 serviceId 转发到具体的服务实例。默认为 false，若为 true 便开启基于服务发现的路由规则。
- `spring.cloud.gateway.discovery.locator.lowerCaseServiceId=true` ：当注册中心为 Eureka 时，设置为 true 表示开启用小写的 serviceId 进行基于服务路由的转发。



978-7-111-66557-1

150 是小的

大的， 大约有十几个。









