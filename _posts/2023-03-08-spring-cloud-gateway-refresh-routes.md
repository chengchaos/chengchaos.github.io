---
title: Spring cloud gateway 动态路由 方案1
key: 2023-03-08
tags: spring cloud gateway refresh route
---

> Suggest search： spring cloud gateway refresh routes

Spring gateway 配置路由主要有两种方式，一种是用 yaml 配置文件，一种是写代码里，这两种方式都是不支持动态配置的。

<!--more-->

下面先来看看 gateway 是如何加载这些配置信息的。

## 0x01 路由初始化

无论是 yaml 还是使用 java 代码，路由的配置最终都是被封装到 `RouteDefinition` 对象中。 一个 `RouteDefinition` 有一个唯一的 ID， 如果不指定，默认是 UUID。 多个 `RouteDefinition` 组成了 Gateway 的路由系统。

所有路由信息在系统启动时就被加载装配好了，并存到了内存里。

查看 `GatewayAutoConfiguration` 类的源码，可以看一下 `propertiesRouteDefinitionLocator(GatewayProperties properties)` 方法，它是装配 yaml 文件的，它返回的是 `PropertiesRouteDefinitionLocator` ，该类实现了 `RouteDefinitionLocator` 接口。

`RouteDefinitionLocator` 接口就是路由器的装载器， 只有一个方法，就是获取路由信息：


```
public interface RouteDefinitionLocator {
    Flux<RouteDefinition> getRouteDefinitions();
}

```


这个 `RouteDefinitionLocator` 接口有多个实现类， 分别对应不同配置方式的路由装载：

- `CachingRouteDefinitionLocator`: 是 `RouteDefinitionLocator` 包装类，提供缓存功能。
- `CompositeRouteDefinitionLocator`: 组合过关 RouteDefinitionLocator 的实现，为 RouteDefinitionLocator 提供统一的入口。
- `PropertiesRouteDefinitionLocator`: 从配置文件读取 RouteDefinition.
- `DiscoveryClientRouteDefinitionLocator`: 从注册中心读取 RouteDefinition.
- `RedisRouteDefinitionRepository`: Redis ? 

通过这几个实现类，再结合上面的 `AutoConfiguration` 里面的 `@Primary` 信息，就知道加载配置信息的顺序。

```text
PropertiesRouteDefinitionLocator-->|配置文件加载初始化| CompositeRouteDefinitionLocator
RouteDefinitionRepository-->|存储器中加载初始化| CompositeRouteDefinitionLocator
DiscoveryClientRouteDefinitionLocator-->|注册中心加载初始化| CompositeRouteDefinitionLocator
```


> 参考： <https://www.jianshu.com/p/b02c7495eb5e>


```
public class CachingRouteLocator
        implements Ordered, RouteLocator, ApplicationListener<RefreshRoutesEvent>, ApplicationEventPublisherAware {

    private static final Log log = LogFactory.getLog(CachingRouteLocator.class);

    private static final String CACHE_KEY = "routes";

    private final RouteLocator delegate;

    private final Flux<Route> routes;

    private final Map<String, List> cache = new ConcurrentHashMap<>();

    private ApplicationEventPublisher applicationEventPublisher;

    public CachingRouteLocator(RouteLocator delegate) {
        this.delegate = delegate;
        routes = CacheFlux.lookup(cache, CACHE_KEY, Route.class).onCacheMissResume(this::fetch);
    }

    private Flux<Route> fetch() {
        return this.delegate.getRoutes().sort(AnnotationAwareOrderComparator.INSTANCE);
    }

    @Override
    public Flux<Route> getRoutes() {
        return this.routes;
    }
    // omitted ...
}
```

这是第一顺序，就是从 `CachingRouteLocator` 中获取路由信息，我们可以打开该类进行一下 Debug 来验证。

在 `getRoues()` 方法中打上断点，不管发起什么请求，必然会走上面的断点处。请求一次走一次。这是将路由信息缓存到了 Map 中。配置信息一旦请求过一次，就会被缓存到 `CachingRouteLocator` 类中，再次发起请求后，会直接从 cache 中读取。

如果想动态刷新配置信息，就需要发起一个 `RefreshRoutesEvent` 的事件，上面的 cache 会监听该事件，并重新拉取路由配置信息。

通过下面的代码，可以看到如果没有 `RouteDefinitionRepository` 的实例，则默认用 `InMemoryRouteDefinitionRepository`。而做动态路由的关键就在这里。即通过自定义的 `RouteDefinitionRepository` 类，来提供路由配置信息。

```
@Configuration(proxyBeanMethods = false)
@ConditionalOnProperty(name = "spring.cloud.gateway.enabled", matchIfMissing = true)
@EnableConfigurationProperties
@AutoConfigureBefore({ HttpHandlerAutoConfiguration.class, WebFluxAutoConfiguration.class })
@AutoConfigureAfter({ GatewayReactiveLoadBalancerClientAutoConfiguration.class,
        GatewayClassPathWarningAutoConfiguration.class })
@ConditionalOnClass(DispatcherHandler.class)
public class GatewayAutoConfiguration {

	@Bean
	@ConditionalOnMissingBean(RouteDefinitionRepository.class)
	public InMemoryRouteDefinitionRepository inMemoryRouteDefinitionRepository() {
		return new InMemoryRouteDefinitionRepository();
	}
}
```

例如：

```
import org.springframework.cloud.gateway.route.RouteDefinition;
import org.springframework.cloud.gateway.route.RouteDefinitionRepository;
import org.springframework.stereotype.Component;
import reactor.core.publisher.Flux;

import java.util.ArrayList;
import java.util.LinkedHashMap;

@Component
public class MyRouteDefinitionRepository implements RouteDefinitionRepository {

    public static final String gateway_routes = "gateway_routes";

    private final Map<String, RouteDefinition> routes = synchronizedMap(new LinkedHashMap<>());

    @Override
    public Flux<RouteDefinition> getRouteDefinitions() {
        List<RouteDefinition> routeDefinitions = new ArrayList<>();
        redisTEmplate.opsForHash()
                .values(geteway_routes)
                .stream()
                .forEach(routeDefinition -> routeDefinitions.add(JSON.parseObject(routeDefinition, RouteDefinition.class)));
        return Flux.fromIterable(routeDefinitions);
    }
    
    @Override
    public Mono<Void> save(Mono<RouteDefinition> route) {
        return null;
    }
    
    @Override
    public Mono<Void> delete(Mono<RouteDefinition> routeId) {
        return null;
    }
}
```

在 `getRouteDefinitions` 方法返回你自定义的路由配置信息即可。这里可以用数据库、nosql 等等任意你喜欢的方式来提供。而且配置信息修改后，发起一次 `RefreshRoutesEvent` 事件即可让配置生效。这就是动态配置路由的核心所在，下面来看具体代码实现。

## 0x02 基于数据库/缓存的动态路由

假设有这样一个配置：

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: header
          uri: http://localhost:8888/header
          filters:
            - AddRequestHeader=header, addHeader
            - AddRequestParameter=param, addParam
          predicates:
            - Path=/jd
```

封装到 java 对象中：

```
import jakarta.annotation.Resource;
import org.springframework.cloud.gateway.event.RefreshRoutesEvent;
import org.springframework.cloud.gateway.filter.FilterDefinition;
import org.springframework.cloud.gateway.handler.predicate.PredicateDefinition;
import org.springframework.cloud.gateway.route.RouteDefinition;
import org.springframework.cloud.gateway.support.NameUtils;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.context.ApplicationEventPublisherAware;
import org.springframework.stereotype.Service;
import org.springframework.web.util.UriComponentsBuilder;

import java.util.Arrays;
import java.util.HashMap;

@Service
public class DynamicRouteService implements ApplicationEventPublisherAware {
    
    @Resource
    private RouteDefinitionWriter routeDefinitionWriter;
    
    @Resource
    private StringRedisTemplate redisTemplate;
    
    private ApplicationEventPublisher publisher;
    
    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }
    
    public void notifyChange() {
        this.publisher.publishEvent(new RefreshRoutesEvent(this));
        
    }

    public void init() {
        RouteDefinition definition = new RouteDefinition();
        definition.setId("id");
        URI uri = UriComponentsBuilder.fromHttpUrl("http://localhost:8888/header");
        definition.setUri(uri);

        // 第一个断言
        PredicateDefinition predicate = new PredicateDefinition();
        predicate.setName("Path");

        Map<String, String> predicateParams = new HashMap<>();
        predicateParams.put("pattern", "/jd");
        predicate.setArgs(predicateParams);

        // 定义 Filter
        FilterDefinition filter = new FilterDefinition();
        filter.setName("AddRequestHeader");
        Map<String, String> filterParams = new HashMap<>();
        // _genkey_ 前缀是固定的
        // org.springframework.cloud.gateway.support.Name
        filterParams.put("_genkey_0", "header");
        filterParams.put("_genkey_1", "addHeader");
        filter.setArgs(filterParams);

        FilterDefinition filter2 = new FilterDefinition();
        filter.setName("AddRequestParameter");
        Map<String, String> filter2Params = new HashMap<>();
        filter2Params.put(NameUtils.GENERATED_NAME_PREFIX+ "0","param");
        filter2Params.put(NameUtils.GENERATED_NAME_PREFIX+ "1", "addParam");
        filter2.setArgs(filter2Params);

        definition.setFilters(Arrays.asList(filter, filter2));
        definition.setPredicates(Arrays.asList(predicate));

        redisTemplate.opsForHash()
                .put("GATEWAY_ROUTES", "key", JSON.toJSONString(definition));

    }
}
```

刷新时，调用一下 `notifyChanged()` 方法，就能完成新配置的替换了。

## 0x03 通过REST接口

gateway 是自带接口能增删改查配置的，这个网上有比较多的教程，随便找个看看就明白了。譬如：

```
package com.example.demo.services;

import jakarta.annotation.Resource;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cloud.gateway.event.RefreshRoutesEvent;
import org.springframework.cloud.gateway.route.RouteDefinition;
import org.springframework.cloud.gateway.route.RouteDefinitionWriter;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.context.ApplicationEventPublisherAware;
import org.springframework.stereotype.Service;
import reactor.core.publisher.Mono;

/**
 * <p>
 * <strong>
 * Describe the function in one sentence.
 * </strong><br /><br />
 * As the title says.
 * </p>
 *
 * @author Cheng, Chao - 2023/3/6 22:29 <br />
 * @see Object
 * @since 1.0
 */
@Service
public class DynamicRouteService implements ApplicationEventPublisherAware {

    private static final Logger logger = LoggerFactory.getLogger(DynamicRouteService.class);

    private ApplicationEventPublisher applicationEventPublisher;

    @Resource
    private RouteDefinitionWriter routeDefinitionWriter;

    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.applicationEventPublisher = applicationEventPublisher;
    }

    private void notifyChange() {
        this.applicationEventPublisher.publishEvent(new RefreshRoutesEvent(this));
    }

    /**
     * 增加路由
     * @param definition
     * @return
     */
    public String add(RouteDefinition definition) {
        routeDefinitionWriter.save(Mono.just(definition)).subscribe();
        notifyChange();
        return "success";
    }

    public void update(RouteDefinition definition) {
        try {
            this.routeDefinitionWriter.delete(Mono.just(definition.getId()));
        } catch (Exception e) {
            logger.error("update fail, not find route(id : {}", definition.getId());
            return;
        }

        try {
            routeDefinitionWriter.save(Mono.just(definition)).subscribe();
            notifyChange();
        } catch (Exception e) {
            logger.error("update route failure", e);
        }
    }


    public void delete(String id) {
        try {
            this.routeDefinitionWriter.delete(Mono.just(id)).block();
            this.notifyChange();
        } catch (Exception e) {
            logger.error("delete route failure", e);
        }
    }
}

```

## 0x04 参考

- <https://blog.csdn.net/tianyaleixiaowu/article/details/83412301>

EOF


