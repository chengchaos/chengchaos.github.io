---
title: Solr 8 安装
key: 2021-10-16
tags: linux solr8
---

用于本地开发测试.

<!--more-->

## 下载

[https://solr.apache.org/downloads.html](https://solr.apache.org/downloads.html)

JDK 1.8 + Tomcat

## 安装

```bash
tar -zxvf solr-8.11.0.tgz -C /works/local/
cd /works/apps
ln -s /works/local/solr-8.11.0 solr8
cd /works/apps/solr8/server

cp -r solr-webapp/webapp /works/apps/tomcat8/webapps/
cd /works/apps/tomcat8/webapps/
mv webapp solr

cd /works/apps/solr8/server/lib
cp metrics* /works/apps/tomcat8/webapps/solr/WEB-INF/lib/
cp http2* /works/apps/tomcat8/webapps/solr/WEB-INF/lib/

```

配置 webapp/solr/WEB-INF/web.xml

```xml
<env-entry>
    <env-entry-name>solr/home</env-entry-name>
    <env-entry-value>/works/apps/solr8/solrhome</env-entry-value>
    <env-entry-type>java.lang.String</env-entry-type>
</env-entry>
```

```bash
mkdir -p /works/apps/solr8/solrhome

mkdir /works/apps/tomcat8/webapps/solr/WEB-INF/classes
cd /works/apps/solr8/server/resources
cp * /works/apps/tomcat8/webapps/solr/WEB-INF/classes/


cd /works/apps/solr8/example/example-DIH/solr/solr
mkdir -p /works/apps/solr8/solrhome/core1/
cp -r * /works/apps/solr8/solrhome/core1/
vim /works/apps/solr8/solrhome/core1/core.properties
name=core1

```

EOF
