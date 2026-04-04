---
title: Gradle 学习笔记
key: 2022-11-13
tags: Gradle Groovy
---

建议关键字: gradle Groovy 构建工具 build tools


<!--more-->

## 0x01 简介

omitted...

## 0x02 安装

官网地址：<https://gradle.org/>

配置环境变量 `GRADLE_HOME`，添加到 `PATH` 

验证：

```bash
$ echo $GRADLE_HOME
/Users/chengchao/apps/gradle
$ gradle -v

Welcome to Gradle 7.5.1!

Here are the highlights of this release:
 - Support for Java 18
 - Support for building with Groovy 4
 - Much more responsive continuous builds
 - Improved diagnostics for dependency resolution

For more details see https://docs.gradle.org/7.5.1/release-notes.html


------------------------------------------------------------
Gradle 7.5.1
------------------------------------------------------------

Build time:   2022-08-05 21:17:56 UTC
Revision:     d1daa0cbf1a0103000b71484e1dbfe096e095918

Kotlin:       1.6.21
Groovy:       3.0.10
Ant:          Apache Ant(TM) version 1.10.11 compiled on July 10 2021
JVM:          17 (Oracle Corporation 17+35-LTS-2724)
OS:           Mac OS X 12.6.1 x86_64

```

## 0x03 了解 Groovy

Groovy 是基于 Java 虚拟机的一种动态语言。Gradle 使用 Groovy 构建和管理的。

- 分号是可选的
- 类/方法牧人是 public 的
- 自动给属性添加 getter/setter 方法
- 属性可以直接使用 `.` 号获取
- 方法中最后一个表达式的值会作为返回值返回
- `==` 等同于 `equals()`，不回有空指针异常
- 可选类型定义，类型自动推倒出来
- 自带 `assert` 语句
- 可选的括号，没有参数的话，方法可以不写括号
- 字符串可用单引号/双引号/三个双引号
- 集合 API 
- 闭包

Grovvy 简单示例

```grovvy
class ProjectVersion {

    private int major;
    private int minor;

    ProjectVersion(int major, int minor) {
        this.major = major
        this.minor = minor
    }

    int getMajor() {
        major
    }

    void setMajor(int major) {
        this.major = major
    }
}

ProjectVersion v1 = new ProjectVersion(1, 1)

println v1.minor

ProjectVersion v2 = null

println("v2 == v1 : " + (v2 == v1))

// 1, 可选的类型定义
def version = 1

// 2, assert
assert version == 1

// 3, 括号可选
println("version => " + version)
println "version = " + version

// assert version == 2

// 4, 字符串
def s1 = 'imooc'
def s2 = "gradle version is ${version}"
def s3 = """my
name is 
chengchaos 
"""

println s1
println s2
println s3

// 5, 集合 API
// list
def buildTools = ['ant', 'maven']
buildTools << 'gradle'

assert buildTools.getClass() == ArrayList
assert buildTools.size() == 3

// map

def buildYears = ['ant': 2000, 'maven': 2004]
buildYears.gradle = 2009
println buildYears.ant
println buildYears['gradle']

println buildYears.getClass()

// 闭包
def Closure c1 = { v ->
    println(v)
}

def c2 = {
    println "hello"
}

def static method1(Closure closure) {
    closure('param')
}

def static method2(Closure closure) {
    closure()
}

method1(c1)
method2(c2)

```

构建脚本示例

```grovvy
// 构建脚本中默认是都有个 Project 的实例
// Project 实例中有个 apply 方法，
// 方法的命名参数 plugin 的值为 java
apply plugin:'java'

// Project 有个属性叫 version 。
version= '0.1'


// repositories 同样也是 Project 的一个方法
// 使用 Closure 作为参数
repositories {
  mavenCentral() // 闭包
}

// 同样是一个方法， 同样是用 Closure 作为参数
dependencies {
  compile 'commons-codec:commons-codec:1.6'
}
```

## 0x04 简单 Gradle 项目示例

build.gradle 

```grovvy

// 构建脚本中默认是都有个 Project 的实例
plugins {
    id 'java'
}

group 'c.b.cheng'
version '1.0-SNAPSHOT'

sourceCompatibility = 17
targetCompatibility = 17

// create a single jar with all dependencies
task fatJar(type: Jar) {
    manifest {
        attributes 'Implementation-Title': 'Gradle Jar File Example',
                'Implementation-Version': archiveVersion,
                'Main-Class': 'c.b.cheng.example.Demo'
    }
    archiveBaseName = project.name + '-all'
    from {
        duplicatesStrategy = DuplicatesStrategy.EXCLUDE
        configurations.runtimeClasspath.findAll {
            // 打包所有依赖的 jar
            it.name.endsWith('.jar')
        }.collect {
            println 'add ' + it.name
            zipTree(it)
        }
    }
    with jar
}


repositories {
    mavenCentral()
}

dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.9.0// '
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.9.0'

    // https://mvnrepository.com/artifact/ch.qos.logback/logback-classic
    // implementation group: 'ch.qos.logback', name: 'logback-classic', version: '1.4.4'
    // https://mvnrepository.com/artifact/ch.qos.logback/logback-classic
    implementation 'ch.qos.logback:logback-classic:1.4.4'

}

test {
    useJUnitPlatform()
}


```

## 0x05 Gradle 简单使用

几个基本概念 

### Project 项目

一个项目代表一个正在构建的组件，当构建启动后，Gradle 会基于 build.gradle 实例化一个 `org.gradle.api.Project` 类对象，并且能够通过 `project` 变量使其隐式可用。

常见的属性：

- group
- name
- version

常见的方法：

- apply // 应用一个插件
- dependencies // 声明依赖哪些 jar
- repositories // 声明哪些仓库
- task  // 声明项目中的任务

属性的其他配置方式：

- ext
- gradle.properties

### Task 任务

Task 对应 `org.gradle.api.Task` 类。主要包括任务动作和任务依赖。任务动作定义了一个最小的工作单元。可以定义依赖于其他任务、动作序列和执行条件。

任务中的方法：

- dependsOn // 定义任务依赖
- doFirst // 在动作列表前边添加一个动作
- doLast / << // 在动作列表末尾添加一个动作

## 0x06 自定义任务

```gradle
def createDir = {
  path -> 
  File dir = new File(path)
  if (!dir.exists()) {
    dir.mkdirs()
  }
}
task makeJavaDir() {
  def paths = ['src/test/java', 'src/test/resources']
  doFirst{
    paths.forEach(createDir)
  }
}
```

## 0x07 构建的生命周期

- 初始化 // 初始化任务
- 配置 // 生成 task 的依赖顺序
- 执行 //

## 0x08 依赖管理

常用仓库：

- mavenLocal // 本机
- mavenCentral
- jcenter
- 自定义 maven 仓库
- 文件仓库 // 本地机器上的文件路径

使用阿里云镜像

对单个项目生效，在项目中的build.gradle修改内容

```grovvy
buildscript {
    repositories {
        maven {
            url 'https://maven.aliyun.com/nexus/content/groups/public/'
        }
        maven {
            url 'https://maven.aliyun.com/nexus/content/repositories/jcenter'
        }
    }
}

allprojects {
    repositories {
        maven {
            url 'https://maven.aliyun.com/nexus/content/groups/public/'
        }
        maven {
            url 'https://maven.aliyun.com/nexus/content/repositories/jcenter'
        }
    }
}
```

对所有项目生效，在 USER_HOME/.gradle/ 下创建 init.gradle 文件

```grovvy
allprojects {
    repositories {
        def ALIYUN_REPOSITORY_URL = 'https://maven.aliyun.com/nexus/content/groups/public'
        def ALIYUN_JCENTER_URL = 'https://maven.aliyun.com/nexus/content/repositories/jcenter'
        all {
            ArtifactRepository repo ->
                if (repo instanceof MavenArtifactRepository) {
                    def url = repo.url.toString()
                    if (url.startsWith('https://repo1.maven.org/maven2')) {
                        project.logger.lifecycle "Repository ${repo.url} replaced by $ALIYUN_REPOSITORY_URL."
                        remove repo
                    }
                    if (url.startsWith('https://jcenter.bintray.com/')) {
                        project.logger.lifecycle "Repository ${repo.url} replaced by $ALIYUN_JCENTER_URL."
                        remove repo
                    }
                }
        }
        maven {
            url ALIYUN_REPOSITORY_URL
            url ALIYUN_JCENTER_URL
        }
    }
}
```

## 0x09 管理依赖

通常的解决冲突的实现步骤： 

- 查看依赖报告
- 排除传递性依赖
- 强制使用一个版本

gradle 会帮助我们使用最高版本解决依赖冲突。

修改默认解决策略

```grovvy
configurations.all {
  resolutionStrategy {
    failOnVersionConflict()
  }
}
```

排出传递性依赖

```grovvy
compile('org.hibrenate:hibernate-core:3.6.3.Final') {
  exclude grouip: "org.slf4j", module: "slf4j-api"
  // transitive = false
}
```

强制指定一个版本

```grovvy
configurations.all {
  resolutionStrategy {
    force 'org.slf4j:slf4j-api:1.7.24'
  }
}
```

## 0x0a 多项目构建

omitted...and so forth.

## 参考链接

- [新一代构建工具gradle](https://www.imooc.com/learn/833)
- [Gradle和Maven使用阿里云国内镜像](https://www.jianshu.com/p/32bc688e1b69)


EOF
