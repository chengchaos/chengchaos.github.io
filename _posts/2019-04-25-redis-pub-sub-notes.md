---
title: redis 发布订阅模式笔记
key: 20190425
tags: redis putsub
---

通常来说，发布订阅（又叫 pub/sub）的特点是订阅者（listener）负责订阅频道（channel），发布者（publisher）负责向频道发送二进制字符串消息（binary string message）。每当有消息被发送给特定的频道，频道的**所有订阅者**都会收到消息。

<!--more-->

## 命令


```bash

# 订阅一个或者多个频道
SUBSCRIBE
SUBSCRIBE channel [channel ...] 

# 退订给定的一个或者多个频道
# 如果不指定频道名称，则退订所有频道
UNSUBSCRIBE
UNSUBSCRIBE [channel [channel ...]]

# 订阅给定模式的频道
PSUBSCRIBE
PSUBSCRIBE pattern [pattern ...]

# 退订给定模式的频道
# 如果不指定模式，则退订所有频道
PUNSUBSCRIBE 
PUNSUBSCRIBE [pattern [pattern ...]

# 向给定的频道发送消息
PUBLISH
PUBLISH <channel> <message>
```

## Java

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>io.lettuce</groupId>
                    <artifactId>lettuce-core</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
		
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>2.9.0</version>
        </dependency>
```

### Jedis

Jedis中的JedisPubSub抽象类提供了订阅和取消的功能。想处理订阅和取消订阅某些channel的相关事件，我们得扩展JedisPubSub类并实现相关的方法：


```java
package com.demo.redis;

import org.apache.log4j.Logger;
import redis.clients.jedis.JedisPubSub;

public class Subscriber extends JedisPubSub {//注意这里继承了抽象类JedisPubSub

    private static final Logger LOGGER = Logger.getLogger(Subscriber.class);

    @Override
    public void onMessage(String channel, String message) {
    	LOGGER.info(String.format("Message. Channel: %s, Msg: %s", channel, message));
    }

    @Override
    public void onPMessage(String pattern, String channel, String message) {
    	LOGGER.info(String.format("PMessage. Pattern: %s, Channel: %s, Msg: %s", 
    	    pattern, channel, message));
    }

    @Override
    public void onSubscribe(String channel, int subscribedChannels) {
    	LOGGER.info("onSubscribe");
    }

    @Override
    public void onUnsubscribe(String channel, int subscribedChannels) {
    	LOGGER.info("onUnsubscribe");
    }

    @Override
    public void onPUnsubscribe(String pattern, int subscribedChannels) {
    	LOGGER.info("onPUnsubscribe");
    }

    @Override
    public void onPSubscribe(String pattern, int subscribedChannels) {
    	LOGGER.info("onPSubscribe");
    }
}
```

有了订阅者，我们还需要一个发布者：

```java

package com.demo.redis;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import org.apache.log4j.Logger;
import redis.clients.jedis.Jedis;

public class Publisher {

    private static final Logger LOGGER = Logger.getLogger(Publisher.class);
    private final Jedis publisherJedis;
    private final String channel;

    public Publisher(Jedis publisherJedis, String channel) {
        this.publisherJedis = publisherJedis;
        this.channel = channel;
    }

    /**
     * 不停的读取输入，然后发布到channel上面，遇到quit则停止发布。
     */
    public void startPublish() {
    	LOGGER.info("Type your message (quit for terminate)");
        try {
            BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
            while (true) {
                String line = reader.readLine();
                if (!"quit".equals(line)) {
                    publisherJedis.publish(channel, line);
                } else {
                    break;
                }
            }
        } catch (IOException e) {
            LOGGER.error("IO failure while reading input", e);
        }
    }
}
```

为简单起见，这个发布者接收控制台的输入，然后将输入的消息发布到指定的channel上面，如果输入quit，则停止发布消息。

接下来是主函数：

```java
package com.demo.redis;

import org.apache.log4j.Logger;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

public class Program {
    
    public static final String CHANNEL_NAME = "MyChannel";
    //我这里的Redis是一个集群，192.168.56.101和192.168.56.102都可以使用
    public static final String REDIS_HOST = "192.168.56.101";
    public static final int REDIS_PORT = 7000;
    
    private final static Logger LOGGER = Logger.getLogger(Program.class);
    private final static JedisPoolConfig POOL_CONFIG = new JedisPoolConfig();
    private final static JedisPool JEDIS_POOL = 
            new JedisPool(POOL_CONFIG, REDIS_HOST, REDIS_PORT, 0);
    
    public static void main(String[] args) throws Exception {
        final Jedis subscriberJedis = JEDIS_POOL.getResource();
        final Jedis publisherJedis = JEDIS_POOL.getResource();
        final Subscriber subscriber = new Subscriber();
        //订阅线程：接收消息
        new Thread(new Runnable() {
            public void run() {
                try {
                    LOGGER.info("Subscribing to \"MyChannel\". This thread will be blocked.");
                    //使用subscriber订阅CHANNEL_NAME上的消息，这一句之后，线程进入订阅模式，阻塞。
                    subscriberJedis.subscribe(subscriber, CHANNEL_NAME);
                    
                    //当unsubscribe()方法被调用时，才执行以下代码
                    LOGGER.info("Subscription ended.");
                } catch (Exception e) {
                    LOGGER.error("Subscribing failed.", e);
                }
            }
        }).start();
        
        //主线程：发布消息到CHANNEL_NAME频道上
        new Publisher(publisherJedis, CHANNEL_NAME).startPublish();
        publisherJedis.close();
        
        //Unsubscribe
        subscriber.unsubscribe();
        subscriberJedis.close();
    }
}
```

参考： https://my.oschina.net/itblog/blog/601284


### Spring


config

```java

/**
 * @author zhangsi
 * @date created in 2018/3/1 15:30
 */
@Configuration
public class RedisConfig {
private static final Logger LOGGER = LoggerFactory.getLogger(RedisConfig.class);

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Object>(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        RedisSerializer<String> stringSerializer = new StringRedisSerializer();
        RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();
        template.setConnectionFactory(redisConnectionFactory);
        template.setKeySerializer(stringSerializer);
        template.setValueSerializer(stringSerializer);
        template.setHashKeySerializer(stringSerializer);
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;
    }

    @Bean
    public Jedis jedis(RedisProperties redisProperties) {

//        Field jedisField = ReflectionUtils.findField(JedisConnection.class, "jedis");
//
//        ReflectionUtils.makeAccessible(jedisField);
//
//        Jedis jedis = (Jedis) ReflectionUtils.getField(jedisField, redisConnectionFactory.getConnection());

        Jedis jedis = new Jedis(redisProperties.getHost(), redisProperties.getPort());

        if (redisProperties.getPassword() != null) {
            jedis.auth(redisProperties.getPassword());
        }

        return jedis;
    }


    /**
     * 本模块中，创建 jedis 客户端的目的是为了使用原生的 api 来完成上锁和解锁的操作
     * 见 {@code CtrlCommandHelper#tryGetDistributedLock() } 和
     * {@code CtrlCommandHelper.releaseDistributedLock() }方法
     *
     * 因为 RedisTemplate 提供的 api 不足以完成原子操作的同时并返回执行结果
     * spring boot2 后 redis 默认的客户端由 jedis 变为 Lettuce
     * 所以理论上可以不创建 jedis 客户端，所以建议通过从
     * RedisTemplate 中获取 nativeConnection 来获取 lettuce 原生 api 来完成加锁和解锁的操作。
     *
     * @see  cn.futuremove.tsp.vehicle.control.service.CommandHelper#tryGetDistributedLock
     * @see  cn.futuremove.tsp.vehicle.control.service.CommandHelper#releaseDistributedLock
     */
    @Bean
    public JedisPool jedisPool(RedisProperties redisProperties) {


        JedisPool jedisPool = new JedisPool(new GenericObjectPoolConfig(),
                redisProperties.getHost(),
                redisProperties.getPort() == 0 ? Protocol.DEFAULT_PORT : redisProperties.getPort(),
                Protocol.DEFAULT_TIMEOUT,
                redisProperties.getPassword());

        LOGGER.debug("JedisPool : {}", jedisPool);

        return jedisPool;
    }

    @Bean
    @ConditionalOnProperty(value = {"spring.redis.cluster"})
    public JedisCluster jedisCluster(RedisProperties redisProperties) {

        List<String> nodes = redisProperties.getCluster().getNodes();

        Set<HostAndPort> nodeSet = new HashSet<>(nodes.size());

        for (String node : nodes) {

            String[] nodeArr = node.split(":");

            nodeSet.add(new HostAndPort(nodeArr[0], Integer.parseInt(nodeArr[1])));
        }

        JedisCluster jedisCluster = new JedisCluster(nodeSet);

        LOGGER.debug("JedisCluster : {}", jedisCluster);

        return jedisCluster;
    }

    @Autowired
    private RedisConnectionFactory redisConnectionFactory;

    @Autowired
    private OfflineNotifyRedisMessageListener offlineNotifyRedisMessageListener;

    @Autowired
    private CommandResultMessageListener commandResultMessageListener;

    /**
     *
     * @return
     */
    @Bean
    public ThreadPoolTaskScheduler threadPoolTaskScheduler() {
        ThreadPoolTaskScheduler scheduler =
                new ThreadPoolTaskScheduler();
        scheduler.setPoolSize(20);
        //scheduler.setPoolSize(50);
        scheduler.setThreadGroupName("chaos");
        scheduler.setThreadNamePrefix("chaos-");
        return scheduler;
    }

    /**
     * 定义 Redis 的监听容器
     * @return offlineNotifyRedisMessageListener
     */
    @Bean
    public RedisMessageListenerContainer redisMessageListenerContainer() {

        RedisMessageListenerContainer container =
                new RedisMessageListenerContainer();

        container.setConnectionFactory(redisConnectionFactory);
        container.setTaskExecutor(threadPoolTaskScheduler());

        Topic appOfflineTopic = new ChannelTopic("app_offline");
        container.addMessageListener(offlineNotifyRedisMessageListener, appOfflineTopic);

        Topic ctrlNotify = new ChannelTopic(RedisKeyType.VEHICLE_CTRL_NOTIFY_CHANNEL_TOPIC);
        container.addMessageListener(commandResultMessageListener, ctrlNotify);

        return container;
    }
}
```

OfflineNotifyRedisMessageListener

```java

/**
 * <p>
 * <strong>
 *     一个 Redis 消息监听器
 * </strong><br /><br />
 * 用于监听 Redis 中的消息, 当有用户异地登录, 通知原来登录的用户,
 * 并提示下线.
 *
 *
 * </p>
 *
 * @author chengchao - 2018-12-15 16:43 <br />
 * @see [相关类方法]
 * @since [产品模块版本]
 */
@Component
public class OfflineNotifyRedisMessageListener implements MessageListener {

    private static final Logger LOGGER = LoggerFactory.getLogger(OfflineNotifyRedisMessageListener.class);

    @Override
    public void onMessage(Message message, byte[] bytes) {

        String topic = new String(message.getChannel(), StandardCharsets.UTF_8);
        String body = new String(message.getBody(), StandardCharsets.UTF_8);

        if (StringUtils.isAnyBlank(body, topic)) {
            LOGGER.warn("body or topic is null or empty !");
            return;
        }
        LOGGER.debug("topic: {}, message body: {}", topic, body);


        VehicleControlContext.sendOfflineNotify(body);
    }
}


```


public 

```java

private void sendOfflineNotifyMessage(BoundValueOperations<String, String> operations) {
        String userLoginInfos = operations.get();

        if (StringUtils.isBlank(userLoginInfos)) {
           return;
        }
        RedisUserDto redisUserDto;

        try {
            redisUserDto = JSON.parseObject(userLoginInfos, RedisUserDto.class);
            if (Objects.isNull(redisUserDto)) {
                return;
            }
        } catch (Exception e) {
            // 解析失败, 无所谓
            LOGGER.warn("exception : {} -> {}", userLoginInfos, e.getMessage());
            return;
        }

        String token = redisUserDto.getToken();
        if (StringUtils.isBlank(token)) {
            return;
        }

        redisTemplate.convertAndSend("app_offline", redisUserDto.getUser().getId() + ":"+ token);

    }
```




<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
