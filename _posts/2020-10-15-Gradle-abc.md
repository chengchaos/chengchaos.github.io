---
title: Gradle 基础入门
key: 2020-10-15
tags: Gradle
---

## 简介

Gradle 是一种开源自动化构建工具，支持多语言环境，受 Ant、Maven 思想的影响，集二者之大成，相比 Ant 的不规范，Maven 的配置复杂、生命周期限制严重，Gradle 既规范也更灵活，可以使用DSL (领域特定语言，如Groovy 或 Kotlin）编写构建脚本，脚本更短小精悍

它的特性有：

- 高度定制化：模块化可扩展，更灵活
- 构建迅速：支持并行构建，自动复用之前任务构建的结果以提高效率
- 功能强大：支持多语言环境，包含 Java、Android、C++、Groovy、Javascript 等项目的构建

> Ant、Maven 有的 Gradle也有，Gradle有的它们不一定有；
>
> Ant、Maven能干的，Gradle 都能干，而且干得更好

<!--more-->

## 安装

下载地址： [https://gradle.org/releases/](https://gradle.org/releases/)

以 Linux 举例：

```bash

cd /opt
wget https://services.gradle.org/distributions/gradle-6.4.1-bin.zip
sudo unzip -oq gradle-6.4.1-bin.zip #解压到/opt/gradle-6.4.1
#切换root权限，设置环境变量
sudo su -
cat >> /etc/profile <<EOF
export PATH=\$PATH:/opt/gradle-6.4.1/bin
export GRADLE_USER_HOME=/opt/gradle-6.4.1 #Gradle的缓存会存在此目录下的caches中
EOF
exit
source /etc/profile

#测试
hellxz@debian:~$ gradle -v

------------------------------------------------------------
Gradle 6.4.1
------------------------------------------------------------

Build time:   2020-05-15 19:43:40 UTC
Revision:     1a04183c502614b5c80e33d603074e0b4a2777c5

Kotlin:       1.3.71
Groovy:       2.5.10
Ant:          Apache Ant(TM) version 1.10.7 compiled on September 1 2019
JVM:          11.0.7 (Debian 11.0.7+10-post-Debian-3deb10u1)
OS:           Linux 4.19.0-9-amd64 amd64
```

## hello world

build.gradle

```gradle
task hello {
      parintln "Hello World!"
}

gradle -q hello
```

> ` -q ` 参数的作用是静默输出，使输出更加清晰

以上脚本定义了一个独立的 task 叫做 hello，并且加入了一个 action，当执行 `gradle hello` 时，Gradle 执行叫做 hello 的 task，即执行了我们提供的 action。

这个 action 时一个包含了一些 Grovvy 代码的闭包（Closure）。

> task 块内可以定义前置、后置执行的方法（闭包）`doFirst` 、 `doLast`， 注意，定义多个时不能保证执行顺序。

## 构建基础

### projects 和 tasks

projects 和 tasks 是 Gradle 中最重要的两个概念。

任何一个 Gradle 构建都是由一个 project 或多个 projects 组成。每个 project 可以是一个 jar 包或 war 应用，也可以是一个由许多其他项目中产生的 jar 工程的 zip 压缩包。

每个 project 都由多个 tasks 组成。每个 task 都代表了构建过程中的一个原子性操作。比如： 编译、打包、生成 javadoc、发布到某个仓库等。

换种说法，project 相当于一个构建工程的一个模块，task 是模块中的要给操作。

### 调用 Grovvy

在 build.gradle （可以才能为 build script，构建配置脚本）中调用 Groovy 的类库：

build.gradle

```gradle
task upper {
      String str = 'this is a simple test'
      println "原始值：" + str
      println "转大写后：" + str.toUpperCase()
}

gradle -q upper

Hello World
原始值：this is a simple test
转大写后：THIS IS A SIMPLE TEST

```

### 定义项目 version/group

在 Maven 中，可以明确定义项目版本，构建时会将这个版本包含在 war 或 jar 等制品的文件名称中，推送到 Maven 私服中也需要设置 group artifactId version 信息，那么 Gradle 中如何定义呢？

Gradle 中，对应 Maven 的三个参数，将 artifactId 变成了 `rootProject.name`，那么只需额外定义 group 与 version

在 build.gradle 中设置

```gradle
version = "0.0.1"
group = "com.cnblogs.hellxz"
```

Gradle 配置中还有一个特殊的配置文件，`gradle.properties`，我们可以在里边配置变量供 build.gradle 读取

```gradle
version=0.0.1
group=com.cnblogs.hellxz
```












参考：

- [【Gradle教程】Gradle 基础入门](https://www.cnblogs.com/hellxz/p/helloworld-gradle.html)

EOF

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
