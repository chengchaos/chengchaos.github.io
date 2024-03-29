---
title: java 8 到 java 17 升级指北 
key: 2022-08-30
tags: Java maven
---

2014 年发布的 java SE 8 和 2017 年发布的 java EE 8，至今还是使用最广泛的 java 版本，大部分 java 开发者对于 java 8 之后的升级总是敬而远之，这跟 java 9 以后的破坏性升级和 oracle 的商用协议有关，但随着 9 月  java 17的发布，我们有更多理由去升级和在新项目中使用更新的 java 了。

<!--more-->

## 为什么要升级？

- java 9 之后的 Java 改变了更新策略，java 11 是 8 之后的第一个 LTS 版本，之后每隔半年更新一个小版本，三年更新一个 LTS 版本，所以 java 17 是下一个 LTS 版本。
- 最显著的改善是几乎免费获得的性能提升。java 8 默认 GC 是 Parallel GC，java 9 之后默认是 G1 GC，且就算是同一个 GC，新版本中的表现也会比旧版本性能好，我们的程序触发 full GC 的次数和 GC 造成的程序暂停会更短。关于这一点，有一篇文章分析了性能 How much faster is Java 17? 。除此之外，每一个新的 Java 版本，尤其是 LTS 版本，都包含改进，例如解决安全漏洞、改进性能和添加新功能。让 java 保持最新有助于项目的健壮性和安全性。开发人员通常也更想在日常工作中使用新技术。
- OracleJdk 在 11 版本之后商用是需要付费的，17 这个版本又改回了商用免费，OpenJdk 和 OracleJdk 之间又可以自由选择了
- spring 刚刚官宣 2022 年即将发布的 spring framework 6.0 和 springboot 3.0 版本最低要求 java 17，且 kafka 3.0 版本之后也会弃用 java 8，升级已经是一个趋势，未来更多框架和中间件会弃用 java 8，作为开发人员也不能停止脚步

## 升级的注意事项

人们在 java 8 这个版本不愿意升级，除了怕影响项目稳定性，还有就是 java 9 之后的发布频率太快，但在多了解一点 java 9 之后的更新策略后就会知道非 LTS 版本是不需要升级，也不建议升级的，只有每隔三年发布的 LTS 版本才有必要升级(另外 java 17 之后 oracle 有个提议改动 LTS 版本发布频率，之后的 LTS 版本可能每两年发布一次)；另一方面是有很多破坏性更改，升级后旧项目可能直接报错。但在 java 8 发布至今的 7 年后，升级的解决方案已经很成熟了，这些问题不应该还是我们升级的阻力

关于升级时可能遇到的问题我做了个汇总，以版本区分，升级时，也建议一步步升级，比如先升级到 java11，没问题再升级到 java17，便于发现升级时的问题。以下解决方案基于 maven

### 第一步建议先升级依赖项

如果你的项目基于 java 8，在升级前最好先升级依赖项，从 java 8 升级到 java 17 是一个很大的跨越，依赖项不升级则出问题的概率会比较高，maven 可以用 `mvn versions:display-dependency-updates` 命令检查依赖项更新，输出会类似这样

![20211116101812](https://fulu-item11-zjk.oss-cn-zhangjiakou.aliyuncs.com/images/20211116101812.png)

然后可以把依赖项升级到输出的对应版本，大部分包升级不会出问题，如果有问题，建议去出问题的依赖官方仓库寻找解决方案。这个命令是直接查询maven远程仓库，如果依赖项多的话会运行比较长的时间

## 各版本升级需要修改的内容

### java 11

java 11 删除了这些原本在 jdk 中的包：

1. javaFX
2. jdk 自带的一些字体，主要影响 Apache POI 这类依赖字体的库，解决方法是自行在操作系统安装字体，比如 Ubuntu，需要安装 `fontconfig` 包 `apt install fontconfig`
3. JMC:Java Mission Control，java 自带的性能分析工具，自 java11 后从jdk删除，可自行下载
4. 删除 java EE 和 CORBA 模块，SE 中删除 java EE 相关的包是因为这些包已经由 java EE 提供，而且由于 oracle 的政策，一些包的命名空间也改变了，例如 JAXB 包下的 `javax.xml.bind.*` 更改为 `jakarta.xml.bind.*`，下图列举了包名的改动，如果项目使用了这些包，需要在代码和pom.xml中更改相应包名

![20211115172941](https://fulu-item11-zjk.oss-cn-zhangjiakou.aliyuncs.com/images/20211115172941.png)

### java 14

删除了 CMS GC，对于老项目或针对 CMS 专门调优过的项目，建议升级后使用 G1 GC

### java 15

Nashorn JavaScript Engine 在这个版本被移除，如果使用了则需要手动添加这个依赖项

```xml
<dependency>
    <groupId>org.openjdk.nashorn</groupId>
    <artifactId>nashorn-core</artifactId>
    <version>15.2</version>
</dependency>
```

### java 16

java 16 也是一个改动很大的版本，这个版本默认对 jdk 内部的很多 api 做了强封装，默认情况下不可访问（可以通过选项 `--illegal-access` 更改这个行为，但官方不建议），这个主要影响一些工具，比如 lombok，而 lombok 在 java 16 发布后不久更新了版本解决这个问题。

如果实在找不到兼容的方法，则在 pom.xml 修改 compiler plugin 参数可以解决问题：

```xml
<plugin>
 <groupId>org.apache.maven.plugins</groupId>
 <artifactId>maven-compiler-plugin</artifactId>
 <version>3.8.1</version>
 <configuration>
  <fork>true</fork>
  <compilerArgs>
   <arg>-J--add-opens=jdk.compiler/com.sun.tools.javac.comp=ALL-UNNAMED</arg>
   <arg>-J--add-opens=jdk.compiler/com.sun.tools.javac.file=ALL-UNNAMED</arg>
   <arg>-J--add-opens=jdk.compiler/com.sun.tools.javac.main=ALL-UNNAMED</arg>
   <arg>-J--add-opens=jdk.compiler/com.sun.tools.javac.model=ALL-UNNAMED</arg>
   <arg>-J--add-opens=jdk.compiler/com.sun.tools.javac.parser=ALL-UNNAMED</arg>
   <arg>-J--add-opens=jdk.compiler/com.sun.tools.javac.processing=ALL-UNNAMED</arg>
   <arg>-J--add-opens=jdk.compiler/com.sun.tools.javac.tree=ALL-UNNAMED</arg>
   <arg>-J--add-opens=jdk.compiler/com.sun.tools.javac.util=ALL-UNNAMED</arg>
   <arg>-J--add-opens=jdk.compiler/com.sun.tools.javac.jvm=ALL-UNNAMED</arg>
  </compilerArgs>
 </configuration>
</plugin>
```

如果升级到 java 16，但 lombok 没有更新，则会报一个让你一头雾水的错误：

```sh
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.8.1:compile (default-compile) on project broken: Compilation failure -> [Help 1]
```

除此之外没有更多信息了，这一点可以说是一个很大的坑

### java 17

主要删除了一些实验性特性和老旧的 API:

1. applet API 被弃用，估计也没有什么项目用到这个
2. 实验性的 AOT 和 JIT 被删除，最近 AOT 的特性还是很火的，想使用这个特性的可以使用 graalVM，spring native 是 spring 基于 graalVM 的实现，使用 spring native 的 java 程序的启动时间会缩短到毫秒级，但也牺牲了一些运行时优化，可以说是java在云原生时代的进化，这里就不过多介绍了。
3. 还有一个最大的变化是之前的 `--illegal-access`参 数不在可用，如果在 java 17 使用这个参数访问受限的 api 则会报出 `InaccessibleObjectException`，大多数情况下只要升级了依赖项是不会碰到这个情况的，但如果出现问题，则可以使用 `--add-opens` 来对不可访问的 api 授权。

## 升级完成后可以做的事情

对开发人员来说最想做的自然是使用新的特性，包括 `var` `records` `instanceof` `switch` 这些新关键字和旧语法的改进，以及 `Stream` 和 `Optional` 等 API 的改进等，此处不在赘述。

## 结语

这里总结的是一些我自己升级过程中遇到的问题，只要将依赖项同步升级，基本可以解决升级会遇到的所有问题。当然不是所有项目都适合升级的，这里需要根据项目的情况仔细斟酌，如果是新项目，想跟上技术迭代的脚步，还是非常推荐升级到 java 17 的。

参考文章及链接：

- <https://www.optaplanner.org/blog/2021/09/15/HowMuchFasterIsJava17.html>
- <https://www.infoq.com/articles/why-how-upgrade-java17/>
- <https://github.com/johanjanssen/JavaUpgrades>

原文连接: [java 8 - java 17 升级指北](https://www.cnblogs.com/fulu/p/15787771.html)

## Appendix

```bash
java -Xmx1g -XshowSettings:all -jar app.jar
```

~~EOF~~


