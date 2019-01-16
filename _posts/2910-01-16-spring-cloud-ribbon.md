---
title: Spring Cloud Ribbon 學習筆記
key: 20190116
tags: java, spring cloud, spring cloud ribbon
---

Spring Cloud Ribbon 是一个基於 HTTP 和 TCP 的客戶端負載均衡工具，它基於 Netflix Ribbon 實現。通過 Spring Cloud 的封裝，可以讓我們輕鬆地將面嚮服務的 REST 請求自動轉換成客戶端負載均衡的服務調用。


<!--more-->

## 客户端负载均衡

所有客户端节点都维护着自己要访问的服务端清单，而这些服务端的清单来自于服务注册中心，客户端负载均衡中也需要心跳去维护服务端清单的健康性。只是这个步骤需要与服务注册中西你配合完成。

在 Spring Cloud 实现的服务治理框架中，默认会创建针对各个服务治理框架的 Ribbon 自动化整合配置，比如 Eureka 中的 
`org.springframework.cloud.netflix.ribbon.eureka.RibbonEurekaAutoConfiguration`, Consul 中的 
`org.springframework.cloud.consul.discovery.RibbonConsulAutoConfiguration`。

通过 Spring Cloud Ribbon 的封装，我们在微服务架构中使用客户端负载均衡调用非常简单，只需要如下两步：

- 服务提供者启动多个服务实例并注册到一个注册中心或者是多个相关联的服务注册中西。
- 服务消费者直接通过调用被 `@LoadBalanced` 注解修饰过的 `RestTemplate` 来实现面向服务的接口调用。


## 详细配置

### 自动化配置

由于 Ribbon 中定义额每一个接口都有许多不同的策略实现，同时这些接口之间又有一定的依赖关系，这使得第一次使用时很难上手。Spring Cloud Ribbon 中的自动化配置可以解决，在引入 Spring Cloud Ribbon 的以来以后，可以自动化构建下面的接口的实现：

- `IClientConfig` : Ribbon 的客户端配置，默认采用 `com.netflix.client.config.DefaultClientConfigImpl` 实现。
- `IRule` : Ribbon 的负载均衡策略，默认采用 `com.netflix.loadbalancer.ZoneAvoidanceRule` 实现，该策略能够在多区域环境下选出最佳区域的实例进行访问。
- `IPing` : Ribbon 的实例检查策略，默认再用 `com.netflix.loadbalancer.NoOpPing` 实现，该检查策略是一个特殊的实现，实际上它并不会检查实例是否可用，而是适中返回 `true` ，默认认为所有服务实例都是可用的。
- `ServerList<Server>` : 服务实例清单的维护机制，默认采用 `com.netflix.loadbalancer.ConfigurationBasedServerList` 实现。
- `ServerListFilter<Server>` : 服务实例清单过滤机制，默认采用  `org.springframewrok.cloud.netflix.ribbon.ZonePreferenceServerListFilter` 实现，该策略能够优先过滤出与请求调用方处于同区域的服务实例。
- `ILoadBalancer` : 负载均衡器，默认采用 `com.netflix.loadbalancer.ZoneAwareLoadBalancer` 是心啊，她具备了区域感知的能力。

上面这些自动化配置内容仅在没有引入 `Spring Cloud Eureka` 等服务治理框架时会如此，在同时引入 Eureka 和 Ribbon 依赖时，自动化配置会有一些不同。

通过自动化配置的实现，我么可以轻松地实现客户端负载均衡。同时，针对一些个性化需求，我们也可以方便地替换上面的这些默认实现，只需在 Spring Boot 应用中创建对应的实现实例就可以覆盖这些默认的配置实现。例如：

```java
@Configuration
public class MyRibbonConfiguration {
    @Bean
    public IPing ribbonPing(IClientConfig config) {
        return new PingUrl();
    }
}

```

由于创建了 `PingUrl` 实例，所以默认的 `NoOpPing` 就不会被创建。


另外，也可以通过使用 `@RibbonClient` 注解来实现更细粒度的客户端配置，比如：

```java
@Configuration
@RibbonClient(name = "Hello-service", configuration = HelloServiceConfiguration.class)
public class RibbonConfiguration {

}
```

 上面的代码实现了为 `hello-service` 服务使用 `HelloServiceConfiguration` 中的配置。

。。。

### 参数配置

对于 Ribbon 的参数配置通常有两种方式：全局配置以及指定客户端配置。

#### 全局配置

全局配置只需使用 `ribbon.<key>=<value>` 格式进行配置即可。其中 `<key>` 带表了 Ribbon 客户端配置的参数吗， `<value>` 代表了队医你个参数的值。比如，以下是全局配置 Ribbon 创建连接的超时时间：

```bash
ribbon.ConnectTimeout=250
```

全局配置可以作为默认值进行设置，当指定客户端配置了响应 key 的值时，将覆盖全局配置的内容。

#### 指定客户端的配置

指定客户端的配置方式采用  `<client>.ribbon.<key>=<value>` 的格式进行配置。其中 `<key>` 和 `<value>` 的含义和全局配置相同，而 `<client>` 代表了客户端的名称。

例如：假设有一个服务消费只通过 RestTemplate 来访问 hello-service 服务的 `/hello` 接口。此时我们会这样调用：

```java
restTemplate.getForEntity("http://hello-service/hello", String.class)
    .getBody();
```

如果没有服务治理框架的帮助，我们应该为该客户端指定具体的实例清单：

```bash
hello-service.ribbon.listOfServers=localhost:8001,localhost:8002,localhost:8003
```

对于 Ribbon 参数的 key 以及 value 类型的定义，可以通过查看 `com.netflix.client.config.CommonClientConfigKey` 类获得跟魏详细的配置内容。


### 与 Eureka 结合

当在 Spring Cloud 的应用中同时引入 Spring Cloud Ribbon 和 Spring Cloud Eureka 依赖时，会触发 Eureka 中实现的 Ribbon 的自动化配置。这时 `ServerList` 的维护机制实现将被 `com.netflix.niws.loadbalancer.DiscoveryEnabledNIWSServerList` 的实例锁覆盖，













If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
