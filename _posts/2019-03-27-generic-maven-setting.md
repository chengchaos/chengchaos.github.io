---
title: 一些常见的 Maven 配置
key: 20190327
tags: maven
---

记录一些常见的 Maven 配置

<!--more-->


## pom.xml

```xml

    <!-- properties -->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <scala.version>2.11.12</scala.version>
        <spring-boot.version>2.0.6.RELEASE</spring-boot.version>
        <platform-bom.version>Cairo-SR5</platform-bom.version>
        <spring-cloud.version>Finchley.SR2</spring-cloud.version>
        <docker.image.prefix>sinogold</docker.image.prefix>
        <mysql.version>8.0.12</mysql.version>
    </properties>

    <!-- 仓库 -->
    <repositories>
        <repository>
            <id>aliyun</id>
            <url>https://maven.aliyun.com/repository/public</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
                <!-- 自己的仓库就应该 always update -->
                <!-- <enabled>true</enabled>-->
                <!-- <updatePolicy>always</updatePolicy>-->
            </snapshots>
        </repository>
        <!-- scala -->
        <repository>
            <id>scalaz</id>
            <name>scalaz</name>
            <url>http://dl.bintray.com/scalaz/releases</url>
        </repository>
        <repository>
            <id>mvnrepository</id>
            <name>Derbysoft Release Repository</name>
            <url>http://search.maven.org/remotecontent?filepath=</url>
        </repository>
        <repository>
            <id>jahia</id>
            <name>mvnrepository</name>
            <url>http://maven.jahia.org/maven2</url>
        </repository>
    </repositories>

    
    <dependencyManagement>
        <dependencies>

            <!-- spring-cloud -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            
            <!-- https://mvnrepository.com/artifact/org.apache.commons/commons-lang3 -->
            <dependency>
                <groupId>org.apache.commons</groupId>
                <artifactId>commons-lang3</artifactId>
                <version>3.11</version>
            </dependency>
            
            <dependency>
                <groupId>com.google.guava</groupId>
                <artifactId>guava</artifactId>
                <version>30.0-jre</version>
            </dependency>
            
            <dependency>
                <groupId>com.esotericsoftware</groupId>
                <artifactId>reflectasm</artifactId>
                <version>1.11.9</version>
            </dependency>

        </dependencies>
    </dependencyManagement>



    <build>
        <finalName>my-project</finalName>
        <!-- To define the plugin version in your parent POM -->
        <pluginManagement>
            <!--
            https://github.com/davidB/scala-maven-plugin
            -->
            <plugins>
                <plugin>
                    <groupId>net.alchim31.maven</groupId>
                    <artifactId>scala-maven-plugin</artifactId>
                    <version>4.4.0</version>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.8.1</version>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <version>2.22.2</version>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-resources-plugin</artifactId>
                    <version>3.2.0</version>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-source-plugin</artifactId>
                    <version>3.2.1</version>
                </plugin>
            </plugins>
        </pluginManagement>
        <!-- To use the plugin goals in your POM or parent POM -->

        <plugins>
            <!-- spring-boot -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>

            <!-- scala -->
            <plugin>
                <groupId>net.alchim31.maven</groupId>
                <artifactId>scala-maven-plugin</artifactId>
                <version>4.4.0</version>
                <executions>
                    <execution>
                        <id>compile-scala</id>
                        <phase>compile</phase>
                        <goals>
                            <goal>add-source</goal>
                            <goal>compile</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>test-compile-scala</id>
                        <phase>test-compile</phase>
                        <goals>
                            <goal>add-source</goal>
                            <goal>testCompile</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <scalaVersion>${scala.version}</scalaVersion>
                    <jvmArgs>
                        <jvmArg>-Xms64m</jvmArg>
                        <jvmArg>-Xmx1024m</jvmArg>
                    </jvmArgs>
                </configuration>
            </plugin>

            <!-- Docker maven plugin -->
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>1.0.0</version>
                <configuration>
                    <imageName>sinogold/${project.artifactId}</imageName>
                    <dockerDirectory>src/main/docker</dockerDirectory>
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                </configuration>
            </plugin>
            
            <!-- Java 编译 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>

            <!-- Java 测试 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.22.2</version>
                <configuration>
                    <skipTests>true</skipTests>
                </configuration>
            </plugin>

            <!-- 资源 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <version>3.2.0</version>
                <executions>
                    <execution>
                        <id>default-resources</id>
                        <phase>validate</phase>
                        <goals>
                            <goal>copy-resources</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>target/classes</outputDirectory>
                            <useDefaultDelimiters>false</useDefaultDelimiters>
                            <delimiters>
                                <delimiter>@*@</delimiter>
                            </delimiters>
                            <resources>
                                <resource>
                                    <directory>src/main/resources/</directory>
                                    <filtering>true</filtering>
                                    <includes>
                                        <include>**/*.yml</include>
                                        <include>**/*.properties</include>
                                        <include>**/*.xml</include>
                                    </includes>
                                </resource>
                            </resources>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

            <!-- 源码 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
                <version>3.2.1</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>jar-no-fork</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            
        </plugins>
    </build>


    <profiles>

        <!-- 开发环境 -->
        <profile>
            <id>dev</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <project.env>dev</project.env>
            </properties>
        </profile>


    </profiles>

```

配置 ~/.m2/settings.xml

```xml
  <servers>
    <server>
       <id>maven-releases</id>
       <username>admin</username>
       <password>password</password>
    </server>
    <server>
       <id>maven-snapshots</id>
       <username>admin</username>
       <password>password</password>
    </server>
  </servers>

  <mirrors>
    <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
  　　<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>        
    </mirror>
  </mirrors>
```

以上是 2020 年 12 月 1 日更新的。

<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
