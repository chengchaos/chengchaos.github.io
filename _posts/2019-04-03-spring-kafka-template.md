---
title: Spring Kafka 教程
key: 20190403
tags: bigdata spring kafka spring-kafka KafkaTemplate
---

Apache Kafka 是目前非常具有吸引力的一个分布式消息系统, Spring 是每个 Java 开发人员耳熟能详的流行的开发框架.

这里讲介绍什么是 **Spring Kafka**, 如何使用 **KafkaTemplate** 来想 kafka brokers 产生消息, 以及如何使用 "listener container" 从 Kafka 中消费消息.

原文: [Spring Kafka Tutorial – Getting Started with the Spring for Apache Kafka](https://howtoprogram.xyz/2016/09/23/spring-kafka-tutorial/)


<!--more-->

![spring kafka](https://howtoprogram.xyz/wp-content/uploads/2016/09/Spring-Kafka.png)


## Spring Kafka 基础

先浏览一下 **Spring Kafka** 的各种组件

### 1.1 发送消息

和 JmsTemplat 或者 JdbcTemplat 类似, Spring Kafka 提供了名为 "template" 的对象叫 [KafkaTemplate](http://docs.spring.io/spring-kafka/api/org/springframework/kafka/core/KafkaTemplate.html). 它封装了一个 Kafka 生产者并提供了许多方便易用的方法来向 Kafka brokers 发送消息. 下面是一些方法的声明:

```java
ListenableFuture<SendResult<K, V>> send(String topic, V data);

ListenableFuture<SendResult<K, V>> send(String topic, K key, V data);
 
ListenableFuture<SendResult<K, V>> send(String topic, int partition, V data);
 
ListenableFuture<SendResult<K, V>> send(String topic, int partition, K key, V data);
 
ListenableFuture<SendResult<K, V>> send(Message<?> message);

```

更多的信息参考[这里](http://docs.spring.io/spring-kafka/api/org/springframework/kafka/core/KafkaTemplate.html)

### 1.2. 接收消息

想接收消息, 我们需要:

- 配置 MessageListenerContainer
- 提供一个 Message Listener 或者使用 `@KafkaListener` 注解

#### 1.2.1. MessageListenserContainer

在 Spring Kafka 中有两个 MessageListenserContainer 的实现:

- KafkaMessageListenerContainer
- ConcurrentMessageListenserContainer

区别在于 KafkaMessageListenerContainer 允许我们使用单线程从 Kafka topics 中消费消息, 而 ConcurrentMessageListnrContainer 则允许我们使用多线程风格从 Kafka topics 中消费消息.

### 1.2.2. @KafkaListener 注解

Spring Kafka 提供了 `@KafkaListener` 注解来标注一个方法, 明确第监听一个 Kafka topics. 例如:

```java

public class Listener {
    @KafkaListener(id="id01", topics="Topic1")
    public void listen(String data) {
    
    }
}

```

## Spring Kafka 示例

现在我们演示一下用 Spring Kafka API 向/从 Kafka topics 中发送和接收消息.

### 2.1.准备

- Apacke Kafka 0.9/0.10 已经安装
- JDK7/8 已经安装
- IDE (Eclipse 或者 IntelliJ)
- 构建工具 (Maven 或者 Gradle)

### 源码结构

源码已经放在了 [GitHub](https://github.com/howtoprogram/apache-kafka-examples/tree/master/spring-kafka-example) 上面了, 或者直接[下载](https://github.com/howtoprogram/apache-kafka-examples/archive/master.zip) .

获得到源码以后, 可以导入源码到 Eclipse 运行测试.

这样导入:

- 菜单:  **Import –> Maven –> Existing Maven Projects**
- 浏览到源码的位置

点击 **Finish** 完成导入.

### 库依赖

仅仅有一个依赖:

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>1.1.0.RELEASE</version>
</dependency>
```

However(然而), 我们的例子使用 Spring Boot 类构建, 并且还需要 JUnit 等依赖:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
 
    <groupId>com.howtoprogram</groupId>
    <artifactId>spring-kafka-example</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>
 
    <name>spring-kafka-example</name>
    <description>Demo project for Spring Boot</description>
 
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.0.RELEASE</version>
        <relativePath /> <!-- lookup parent from repository -->
    </parent>
 
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>
 
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
            <version>1.1.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka-test</artifactId>
            <version>1.1.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
 
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Brixton.SR5</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
 
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
 
</project>
```

### 类说明

#### 2.4.1 SpringKafkaExampleApplication

此类是 Spring Boot 应用的入口:

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
 
@SpringBootApplication
public class SpringKafkaExampleApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringKafkaExampleApplication.class, args);
    }
}
```

#### 2.4.2 ProducerConfig

这个类包含了 KafkaTemplat 的配置, 我们需要明确指定 Kafka 生产者需要用到的一些属性.


```java
@Configuration
@EnableKafka
public class KafkaProducerConfig {
    @Bean
    public ProducerFactory<String, String> producerFactory() {
        return new DefaultKafkaProducerFactory<>(producerConfigs());
    }
 
    @Bean
    public Map<String, Object> producerConfigs() {
        Map<String, Object> props = new HashMap<>();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ProducerConfig.RETRIES_CONFIG, 0);
        props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
        props.put(ProducerConfig.LINGER_MS_CONFIG, 1);
        props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432);
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        return props;
    }
 
    @Bean
    public KafkaTemplate<String, String> kafkaTemplate() {
        return new KafkaTemplate<String, String>(producerFactory());
    }
}
```


#### 2.4.3. KafkaConsumerConfig

这里定义了从 Kafka broker 消费消息的配置.

```java

@Configuration
@EnableKafka
public class KafkaConsumerConfig {
    @Bean
    KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<String, String>> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        factory.setConcurrency(3);
        factory.getContainerProperties().setPollTimeout(3000);
        return factory;
    }
 
    @Bean
    public ConsumerFactory<String, String> consumerFactory() {
        return new DefaultKafkaConsumerFactory<>(consumerConfigs());
    }
 
    @Bean
    public Map<String, Object> consumerConfigs() {
        Map<String, Object> propsMap = new HashMap<>();
        propsMap.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        propsMap.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
        propsMap.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "100");
        propsMap.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, "15000");
        propsMap.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        propsMap.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        propsMap.put(ConsumerConfig.GROUP_ID_CONFIG, "group1");
        propsMap.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        return propsMap;
    }
 
    @Bean
    public Listener listener() {
        return new Listener();
    }
}
```

创建 KafkaListenerContainer 需要定义 KafkaListenerContainerFactory. 这个例子中, 我们使用 ConcurrentKafkaListenerContainer 多线程风格消费消息.

> 注意: broker 的地址是: **localhost:9092**

我们定义一个 Listenr (监听器) 将一个方法用 `@KafkaListener` 注解标注起来, 使它可以监听和处理消息.

```java
public class Listener {
 
    public final CountDownLatch countDownLatch1 = new CountDownLatch(1);
 
    @KafkaListener(id = "foo", topics = "topic1", group = "group1")
    public void listen(ConsumerRecord<?, ?> record) {
        System.out.println(record);
        countDownLatch1.countDown();
    }
 
}
```

这个监听器具有 `id = foo` 和  `group = group1` 属性, 并且将消费 topic 名称为 "topic1" 的消息, 它内部还有一个 CountDownLatch 变量(名叫 countDownLatch1) 用于接下来的单元测试.

#### 2.4.4 SpringKafkaExampleApplicationTest

这是一个单元测试类用来测试我们的例子.

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringKafkaExampleApplicationTests {
 
    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;
    @Autowired
    private Listener listener;
 
    @Test
    public void contextLoads() throws InterruptedException {
 
        ListenableFuture<SendResult<String, String>> future = kafkaTemplate.send("topic1", "ABC");
        future.addCallback(new ListenableFutureCallback<SendResult<String, String>>() {
            @Override
            public void onSuccess(SendResult<String, String> result) {
                System.out.println("success");
            }
 
            @Override
            public void onFailure(Throwable ex) {
                System.out.println("failed");
            }
        });
        System.out.println(Thread.currentThread().getId());
        assertThat(this.listener.countDownLatch1.await(60, TimeUnit.SECONDS)).isTrue();
 
    }
 
}
```

为了发送消息到 Kafka topic, 我们注入了一个 KafkaTemplate 对象(使用 `@Autowire`), 我们还注入了监听器来验证这个结果.

我们向 KafkaTemplate 对象注册了一个 ListenableFutureCallback 对象来验证消息发送给 topic 是否成功.

#### 2.4.5 运行

第一步: 确保 Kafka broker 是运行状态的, 并监听 **localhost:9092**

第二步: 运行测试类.


## 3. Summary

We have just gotten through a Spring Kafka tutorial. Spring Kafka supports us in integrating Kafka with our Spring application easily and a simple example as well. In future posts, I’s like to provide more examples on using Spring Kafka such as: multi-threaded consumers, multiple KafkaListenerContainerFactory, etc. Recently, I have some more article on Apache Kafka. If you’re interested in, you can refer to the following links:

<p><a href="https://howtoprogram.xyz/big-data-technologies/apache-kafka-tutorial/" target="_blank">Apache Kafka Tutorial</a></p>
<p><a href="https://howtoprogram.xyz/2016/04/30/getting-started-apache-kafka-0-9/" target="_blank">Getting started with Apache Kafka 0.9</a></p>
<p><a href="https://howtoprogram.xyz/2016/07/21/using-apache-kafka-docker/" target="_blank">Using Apache Kafka Docker</a></p>
<p><a href="https://howtoprogram.xyz/2016/05/29/create-multi-threaded-apache-kafka-consumer/" target="_blank">Create Multi-threaded Apache Kafka Consumer</a></p>
<p><a href="https://howtoprogram.xyz/2016/07/08/apache-kafka-command-line-interface/">Apache Kafka Command Line Interface</a></p>
<p><a href="https://howtoprogram.xyz/2016/06/04/write-apache-kafka-custom-partitioner/" target="_blank">Write An Apache Kafka Custom Partitioner</a></p>
<p><a href="https://howtoprogram.xyz/2016/06/06/write-custom-serializer-apache-kafka/" target="_blank">How To Write A Custom Serializer in Apache Kafka</a></p>
<p><a href="https://howtoprogram.xyz/2016/07/10/apache-kafka-connect-example/" target="_blank">Apache Kafka Connect Example</a></p>
<p><a href="https://howtoprogram.xyz/2016/07/30/apache-kafka-connect-mqtt-source-tutorial/" target="_blank">Apache Kafka Connect MQTT Source Tutorial</a></p>
<p><a href="https://howtoprogram.xyz/2016/08/06/apache-flume-kafka-source-and-hdfs-sink/" target="_blank">Apache Flume Kafka Source And HDFS Sink Tutorial</a></p>
<p><a href="https://howtoprogram.xyz/2016/09/25/spring-kafka-multi-threaded-message-consumption/" target="_blank">Spring Kafka – Multi-threaded Message Consumption</a></p>




<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
