---
title: 照猫画虎的 Prometheus 和 AlertManager 安装配置
key: 2023-08-29
tags: prometheus alertmanager
---

> Search suggest: prometheus alertmanager 监控 告警

这是一个照猫画虎的安装笔记, just so so。

<!--more-->

## 0x01 Prometheus 安装

### 0x01-01 二进制安装

1, 下载.

去 prometheus 的 github 网站上下载. [地址](https://github.com/prometheus/prometheus/releases/)

2, 解压和安装.

```bash
mkdir -p /works/prometheus/{bin,conf,data}
```

复制二进制文件到 bin 目录, 复制 yml 文件到 conf 目录.

3, 配置

```yaml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"
  - "demo-prometheus_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
  - job_name: 'demo-prometheus'
    metrics_path: "/actuator/prometheus"
    static_configs:
      - targets: ["localhost:8096"]
```

配置为 systemctl 管理

```bash
cd /works/prometheus/conf/
touch prometheus.service
vim prometheus.service

[Unit]
Description=Prometheus Node Exporter
Documentation=https://prometheus.io/
After=network.target
 
[Service]
Type=simple
User=chengchao
ExecStart=/works/prometheus/bin/prometheus \
  --config.file=/works/prometheus/conf/prometheus.yml \
  --storage.tsdb.path=/works/prometheus/data \
  --storage.tsdb.retention=15d \
  --query.max-concurrency=20 \
  --query.timeout=2m \
  --web.read-timeout=5m  \
  --web.max-connections=10 \
  --web.enable-lifecycle
Restart=always
RestartSec=1
[Install]
WantedBy=multi-user.target

sudo cp prometheus.service /lib/systemd/system/
sudo systemctl start prometheus
sudo systemctl status prometheus

lsof -i:9090

COMMAND    PID      USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
prometheu 6109 chengchao    7u  IPv6  86537      0t0  TCP *:websm (LISTEN)
prometheu 6109 chengchao   10u  IPv6  85780      0t0  TCP localhost:47338->localhost:websm (ESTABLISHED)
prometheu 6109 chengchao   11u  IPv6  86545      0t0  TCP localhost:websm->localhost:47338 (ESTABLISHED)

ps -ef | grep prometheus

chengch+  4830  4200  0 Aug29 pts/2    00:04:46 java -jar demo-prometheus-0.0.1-SNAPSHOT.jar
chengch+  6109     1  0 23:43 ?        00:00:00 /works/prometheus/bin/prometheus --config.file=/works/prometheus/conf/prometheus.yml --storage.tsdb.path=/works/prometheus/data --storage.tsdb.retention=15d --query.max-concurrency=20 --query.timeout=2m --web.read-timeout=5m --web.max-connections=10 --web.enable-lifecycle
chengch+  6132  6000  0 23:46 pts/4    00:00:00 grep --color=auto prometheus

```

## 0x02 AlertManager 安装


1, 下载.

去 AlertManager 的 github 网站上下载. [地址](https://github.com/prometheus/alertmanager/releases/)

2, 解压和安装.

```bash
mkdir -p /works/alertmanager/{bin,conf,data}
```

复制二进制文件到 bin 目录, 复制 yml 文件到 conf 目录.

3, 配置

```yaml
route:
  group_by: ['HttpDemoPrometheusDown']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  receiver: 'web.hook'
receivers:
  - name: 'web.hook'
    webhook_configs:
      - url: 'http://127.0.0.1:5001/wx'
        send_resolved: true
#
#inhibit_rules:
#  - source_match:
#      severity: 'critical'
#    target_match:
#      severity: 'warning'
#    equal: ['alertname', 'dev', 'instance']

```

配置为 systemctl 管理

```bash
cd /works/alertmanager/conf/
touch alertmanager.service
vim alertmanager.service


[Unit]
Description=alertmanager
Documentation=https://prometheus.io/
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
User=chengchao
ExecStart=/works/alertmanager/bin/alertmanager \
  --storage.path=/works/alertmanager/data/ \
  --config.file=/works/alertmanager/conf/alertmanager.yml \
  --web.external-url=http://192.168.1.251
Restart=always
RestartSec=1
# Restart=on-failure

[Install]
WantedBy=multi-user.target


sudo cp alertmanager.service /lib/systemd/system/
sudo systemctl start alertmanager
sudo systemctl status alertmanager

lsof -i:9093
COMMAND    PID      USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
alertmana 4542 chengchao    8u  IPv6  66668      0t0  TCP *:copycat (LISTEN)

ps -ef | grep alertmanager

chengch+  4542     1  0 Aug29 ?        00:03:26 /works/alertmanager/bin/alertmanager --storage.path=/works/alertmanager/data/ --config.file=/works/alertmanager/conf/alertmanager.yml --web.external-url=http://192.168.1.251
chengch+  6243  6000  0 23:59 pts/4    00:00:00 grep --color=auto alertmanager
```

## 0x03 Java 项目信息收集

1, 在 pom.xml 文件中添加依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <!-- https://mvnrepository.com/artifact/io.micrometer/micrometer-registry-prometheus -->
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-registry-prometheus</artifactId>
            <version>1.11.3</version>
        </dependency>

        <!-- 可选, 用于进程内存使用图表 -->
        <!-- https://mvnrepository.com/artifact/io.github.mweirauch/micrometer-jvm-extras -->
        <dependency>
            <groupId>io.github.mweirauch</groupId>
            <artifactId>micrometer-jvm-extras</artifactId>
            <version>0.2.2</version>
        </dependency>
````

2, 修改 spring boot 配置(application.yml)

```yaml
server:
  port: 8096

spring:
  application:
    name: demo-prometheus

management:
  endpoint:
    health:
      show-details: always
  endpoints:
    web:
      exposure:
        include: 'prometheus, health'  # 暴露/actuator/prometheus
  metrics:
    tags:
      application: ${spring.application.name}  # 暴露的数据中添加 application label
```

3, 在 Prometheus 配置中添加 scrape_config

```yaml
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
  - job_name: 'demo-prometheus'
    metrics_path: "/actuator/prometheus"
    static_configs:
      - targets: ["localhost:8096"]
```

## 0x04 AlertManager 配置 Web Hook

1, 配置 Prometheus 和 Alert Manager 对接

修改 prometheus.yml 文件, 添加 altermanagerr 的地址

```yaml
# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']
```

2, 在 Prometheus 中添加告警规则

在实际环境中，告警规则肯定有很多，比如对服务器异常进行告警，就有宕机、CPU 使用率超过 100%、内存使用率超过 80%、硬盘使用率超过 80% 等等。

所以，我们需要创建一个文件夹，针对每个监控对象，创建一个配置文件。

这里直接在 Prometheus 的程序的 conf 目录下创建一个 alert_rules 文件夹，用于存放所有的告警规则。

这里测试 web hook , 我们创建了一个 demo-prometheus_rules.yml

```bash
cat demo-prometheus_rules.yml
groups:
- name: demo-prometheus-alert-rule
  rules:
  - alert: HttpDemoPrometheusDown
    expr: sum(up{job="demo-prometheus"}) == 0
    for: 5s
    labels:
      severity: critical
```

修改 prometheus.yml 文件, 增加 role 文件

```yaml

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"
  - "demo-prometheus_rules.yml"

```

让 prometheus 重新加载配置文件

```bash
curl -X POST http://localhost:9090/-/reload
```

## 0x05 开发 Webhook 服务

待续



## Appendix

以下是参考链接。

- [prometheus / prometheus](https://github.com/prometheus/prometheus/releases/)
- [prometheus / alertmanager](https://github.com/prometheus/alertmanager/releases/)
- [从零搭建Prometheus监控报警系统](https://www.cnblogs.com/chenqionghe/p/10494868.html)
- [Prometheus完整的部署方案+实战实例](https://zhuanlan.zhihu.com/p/355884791)
- [运维监控系列（14）-Alertmanager添加webhook告警通知功能](https://blog.csdn.net/qq_43437874/article/details/120411586)
- [配置alertmanager通过webhook告警到企业微信群](https://www.cnblogs.com/sunnytomorrow/p/16789717.html)
- [AlertManager的详细配置](https://www.cnblogs.com/kebibuluan/p/14928490.html)
- [Prometheus监控神器-Alertmanager篇(1)](https://zhuanlan.zhihu.com/p/179292686)
- [prometheus监控java项目（jvm等）：k8s外、k8s内](https://www.cnblogs.com/uncleyong/p/15693542.html)
- [Prometheus设置systemctl管理](https://www.cnblogs.com/minseo/p/13395851.html)

EOF