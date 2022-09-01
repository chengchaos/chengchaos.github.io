---

title: Feign 的客户端组件
key: 2021-08-29
tags: git
---

客户端组件是 Feign 中一个非常重要的组件, 负责做种的 HTTP 请求的执行, 它的核心逻辑是: 发送 Request 到服务器, 在接收到 Response 后进行解码, 最后返回结果.

<!--more-->

feign.Client 接口是代表客户端的顶层接口, 只有一个抽象方法:

```java

package feign;

/**
 * Submits HTTP {@link Request requests}. Implementations are expected to be thread-safe.
 */
public interface Client {
      /**
   * Executes a request against its {@link Request#url() url} and returns a response.
   *
   * @param request safe to replay.
   * @param options options to apply to this request.
   * @return connected response, {@link Response.Body} is absent or unread.
   * @throws IOException on a network error connecting to {@link Request#url()}.
   */
  Response execute(Request request, Options options) throws IOException;
  
}
```

不同的 feign.Client 客户端实现类其内部提交 http 请求的技术是不同的. 常见的有:

- `feign.Client.Default` : 默认实现, 使用 JDK 的 `HttpURLConnection` 实现.
- `ApacheHttpClient` 需要引入 : io.github.openfeign/feign-httpclient ; org.apache.httpcomponents/httpclient
- `OkHttpClient`
- `FeignBlockingLoadBalancerClient` : 内部使用 Ribbon 负载均衡技术实现 Http 请求处理.
- `RetryableFeignBlockingLoadBalancerClient`

## Feign 使用的流程

1. 在启动类上使用 `@EnableFeignClients` 注解开启 Feign 的装配和远程代理实例的创建.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(FeignClientsRegistrar.class)
public @interface EnableFeignClients {
}
```

在 `@EnableFeignClients` 注解源码中可以看到导入了 `FeignClientsRegistrar` 类, 次类用于扫描 `@FeignClient` 注解了的接口.

2. 对 `@FeignClient` 注解的接口的扫描后, 创建远程调用的动态代理实例.

EOF
