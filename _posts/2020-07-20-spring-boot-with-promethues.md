---
title: SpringBoot 使用 prometheus 监控
key: 2020-07-20
tags: spring-boot prometheus
---


## 介绍

### Prometheus 

Prometheus 是一个根据应用的 metrics 来进行监控的开源工具。相信很多工程都在使用它来进行监控，有关详细介绍可以查看官网： [https://prometheus.io/docs/introduction/overview/](https://prometheus.io/docs/introduction/overview/)


<!--more-->


我的安装步骤就是下载，解压，编写启动脚本。

```bash
$ cd /data/prometheus
$ ln -s prometheus-2.20.0-rc.0.linux-amd64 defaults
$
$ cat bin/start-prometheus.sh
#!/usr/bin/env bash
cd /data/prometheus
nohup defaults/prometheus  \
        --config.file="defaults/prometheus.yml" > logs/output.log 2>&1 &

```

### node_exporter

https://github.com/prometheus/node_exporter/releases/download/v1.0.1/node_exporter-1.0.1.linux-amd64.tar.gz



### Grafana

Grafana 是一个开源的分析和监控的界面。

官网地址 [htts://grafana.com/](https://grafana.com)

下载： https://grafana.com/grafana/download


Standalone Linux Binaries(64 Bit)SHA256: 4b6d6ce3670b281919dac8da4bf6d644bc8403ceae215e4fd10db0f2d1e5718e

```bash 
wget https://dl.grafana.com/oss/release/grafana-7.1.0.linux-amd64.tar.gz
tar -zxvf grafana-7.1.0.linux-amd64.tar.gz
```

Red Hat, CentOS, RHEL, and Fedora(64 Bit)SHA256: 36010e3f22c314317e74e5a5ec7a58f0c3432f9f6b8c8dbf95f9b5ed6a6cbca5

```bash 
wget https://dl.grafana.com/oss/release/grafana-7.1.0-1.x86_64.rpm
sudo yum install grafana-7.1.0-1.x86_64.rpm
```

启动：

```bash
$ sudo systemctl start grafana-server
$ systemctl status grafana-server
● grafana-server.service - Grafana instance
   Loaded: loaded (/usr/lib/systemd/system/grafana-server.service; disabled; vendor preset: disabled)
   Active: active (running) since Mon 2020-07-20 10:45:25 CST; 42s ago
     Docs: http://docs.grafana.org
 Main PID: 107134 (grafana-server)
    Tasks: 10
   Memory: 15.4M
   CGroup: /system.slice/grafana-server.service
           └─107134 /usr/sbin/grafana-server --config=/etc/grafana/grafana.ini --pidfile...

Jul 20 10:45:25 guojin-test-1 grafana-server[107134]: t=2020-07-20T10:45:25+0800 lvl=in..."
Jul 20 10:45:25 guojin-test-1 grafana-server[107134]: t=2020-07-20T10:45:25+0800 lvl=in..."
Jul 20 10:45:25 guojin-test-1 grafana-server[107134]: t=2020-07-20T10:45:25+0800 lvl=in..."
Jul 20 10:45:25 guojin-test-1 grafana-server[107134]: t=2020-07-20T10:45:25+0800 lvl=in..."
Jul 20 10:45:25 guojin-test-1 grafana-server[107134]: t=2020-07-20T10:45:25+0800 lvl=in...n
Jul 20 10:45:25 guojin-test-1 grafana-server[107134]: t=2020-07-20T10:45:25+0800 lvl=in...s
Jul 20 10:45:25 guojin-test-1 grafana-server[107134]: t=2020-07-20T10:45:25+0800 lvl=in..."
Jul 20 10:45:25 guojin-test-1 grafana-server[107134]: t=2020-07-20T10:45:25+0800 lvl=in...s
Jul 20 10:45:25 guojin-test-1 systemd[1]: Started Grafana instance.
Jul 20 10:45:25 guojin-test-1 grafana-server[107134]: t=2020-07-20T10:45:25+0800 lvl=in...=
Hint: Some lines were ellipsized, use -l to show in full
```

修改配置：

https://www.cnblogs.com/shhnwangjian/p/6911415.html

**以deb或者rpm安装的，则默认的配置文件是/etc/grafana/grafana.ini**

默认密码： admin / admin 

可以修改： [...](Mima2016)


添加数据源， 选 Prometheus。

去 Grafana 的官方面板 （[https://grafana.com/dashboards](https://grafana.com/dashboards)) 查找别人配置好的面板。


比如 ： 8919

然后在 Import 界面中数据 ID， Load 好以后配置数据源为我们的 Prometheus。就可以看见了。



## spring-boot

### 添加依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
	
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
	

    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-registry-prometheus</artifactId>
        <version>1.5.2</version>
    </dependency>
	
	<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```



### 配置文件

配置文件中加入配置，这里就只进行一些简单配置，`management.metrics.tags.application` 属性是本文配合 Grafana 的 Dashboard 设置的，如下所示：

```properties
spring.application.name=springboot_prometheus
management.endpoints.web.exposure.include=*
management.metrics.tags.application=${spring.application.name}
```

或者 yaml 文件

```yml

spring:
  application:
    name: my-prometheus

server:
  port: 7070

management:
  endpoints:
    web:
      exposure:
        include: '*'
  metrics:
    tags:
      application: ${spring.application.name}
```


### Application

修改启动类

```java

@SpringBootApplication 
public class Springboot2PrometheusApplication {

	public static void main(String[] args) {
		SpringApplication.run(Springboot2PrometheusApplication.class, args);
	}
	@Bean
	MeterRegistryCustomizer<MeterRegistry> configurer(
			@Value("${spring.application.name}") String applicationName) {
		return (registry) -> registry.config()
		        .commonTags("application", applicationName);
	}
}

```

启动项目，可以看到一些指标： http://localhost:7070/actuator/prometheus 



在 Prometheus 中添加 Spring-Boot 应用的配置：

```yaml
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:9090']
  # localhost node_exporter
  - job_name: 'node'
    static_configs:
    - targets: ['localhost:9100']
  # spring-boot-demo
  - job_name: 'my-prometheus'
    scrape_interval: 5s
    metrics_path: '/actuator/prometheus'
    static_configs:
    - targets: ['172.16.16.27:7070']

```


配置到 Grafana 中，以 ID 为 4701 的 Dashboard 为例：

https://grafana.com/dashboards/4701


在 Grafana 中点击 import 按钮，填写 4701，点击 Load 按钮。然后导入数据源。





参考： 

- https://www.ituring.com.cn/article/507548
---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>





