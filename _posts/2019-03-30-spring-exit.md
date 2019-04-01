---
title: Spring-Boot 退出服务（exit）时调用自定义的销毁方法
key: 20190330
tags: spring-boot 
---

参考： [SpringBoot之退出服务（exit）时调用自定义的销毁方法](https://blog.csdn.net/zknxx/article/details/52204036)

<!--more-->

我们在工作中有时候可能会遇到这样场景，需要在退出容器的时候执行某些操作。Spring-Boot中有两种方法可以供我们来选择：

- 实现 `DisposableBean` 接口
- 使用 `@PreDestroy` 注解



## 实现 `DisposableBean` 接口


```java 

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.boot.ExitCodeGenerator;
import org.springframework.stereotype.Component;

@Component
public class MyDisposableBean implements DisposableBean, ExitCodeGenerator {

    private static final Logger LOGGER = LoggerFactory.getLogger(MylDisposableBean.class);

    @Override
    public void destroy() throws Exception {

        LOGGER.info("系统关闭，清理资源……");

    }

    @Override
    public int getExitCode() {
        return 0;
    }
}

```


## 使用 `@PreDestroy` 注解

```java

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

import javax.annotation.PreDestroy;

@Component
public class MyDestoryer {

    private static final Logger LOGGER = LoggerFactory.getLogger(MyDestoryer.class);

    @PreDestroy
    public void destory() {

        LOGGER.info("系统关闭，清理资源……");
    }
}


```



<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
