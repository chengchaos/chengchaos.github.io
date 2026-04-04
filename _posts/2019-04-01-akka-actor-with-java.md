---
title: 用 Java 写一个简单的 Actor 应用
key: 20190330
tags: akka actor
---

假定现在由这么一个场景：老板嗅到了市场上的一个商机，准备开启一个新项目，他将要求传达给了经理，经理根据相应的需求，来安排适合的的员工进行工作。
这个例子很简单，现在我们来模拟一下这个场景：


<!--more-->

## 环境搭建

```xml

        <dependency>
            <groupId>org.scala-lang.modules</groupId>
            <artifactId>scala-java8-compat_2.11</artifactId>
            <version>RELEASE</version>
        </dependency>

        <!-- Akka -->
        <dependency>
            <groupId>com.typesafe.akka</groupId>
            <artifactId>akka-actor_2.11</artifactId>
            <version>2.4.20</version>
        </dependency>

        <dependency>
            <groupId>com.typesafe.akka</groupId>
            <artifactId>akka-http-core_2.11</artifactId>
            <version>2.4.11.2</version>
        </dependency>

        <dependency>
            <groupId>com.typesafe.akka</groupId>
            <artifactId>akka-slf4j_2.11</artifactId>
            <version>2.4.20</version>
        </dependency>

        <dependency>
            <groupId>com.typesafe.akka</groupId>
            <artifactId>akka-remote_2.11</artifactId>
            <version>2.4.20</version>
        </dependency>


        <!-- https://mvnrepository.com/artifact/com.syncthemall/boilerpipe -->
        <dependency>
            <groupId>com.syncthemall</groupId>
            <artifactId>boilerpipe</artifactId>
            <version>1.2.2</version>
        </dependency>


        <dependency>
            <groupId>com.typesafe.akka</groupId>
            <artifactId>akka-testkit_2.11</artifactId>
            <version>2.4.20</version>
            <scope>test</scope>
        </dependency>


        <!-- 阿里巴巴 fastjson -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.45</version>
        </dependency>

        <!-- logback-classic -->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.3</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
```

## 开始


### 消息

```java
package com.example.myscala002.akka;

import akka.actor.ActorPath;

import java.util.StringJoiner;

/**
 * <p>
 * <strong>
 * 1.首先我们来创建一些消息：
 * </strong><br /><br />
 * </p>
 *
 * @author chengchaos[as]Administrator - 2019/4/1 0001 下午 5:13 <br />
 * @since 1.1.0
 */
public class Message {

    private String content;

    public String getContent() {
        return content;
    }

    public Message(String content) {
        this.content = content;
    }


    @Override
    public String toString() {
        return new StringJoiner(", ", Message.class.getSimpleName() + "[", "]")
                .add("content='" + content + "'")
                .toString();
    }

    public static class Business extends Message {

        public Business(String content) {
            super(content);
        }
    }


    public static class Meeting extends Message {

        public Meeting(String content) {
            super(content);
        }
    }


    public static class Confirm extends Message {

        private ActorPath actorPath;

        public Confirm(String content, ActorPath actorPath) {
            super(content);
            this.actorPath = actorPath;
        }



        public ActorPath getActorPath() {
            return actorPath;
        }


    }


    public static class DoAction extends Message {

        public DoAction(String content) {
            super(content);
        }
    }


    public static class Done extends Message {

        public Done(String content) {
            super(content);
        }
    }
}


```

### Boos

```java

package com.example.myscala002.akka;

import akka.actor.AbstractActor;
import akka.actor.ActorRef;
import akka.actor.ActorSelection;
import akka.actor.Props;
import akka.event.Logging;
import akka.event.LoggingAdapter;
import akka.japi.pf.ReceiveBuilder;
import akka.pattern.Patterns;
import akka.util.Timeout;
import scala.PartialFunction;
import scala.concurrent.Future;
import scala.runtime.BoxedUnit;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.CompletionStage;
import java.util.concurrent.TimeUnit;


public class BossActor extends AbstractActor {

    private LoggingAdapter logging = Logging.getLogger(context().system(), this);

    private volatile int taskCount = 0;

    @Override
    public PartialFunction<Object, BoxedUnit> receive() {
        return ReceiveBuilder.create()
                .match(Message.Business.class, (b) -> {
                    logging.info("BossActor message ==> {}, 我们必须得干点啥，快！", b);
                    logging.info("BossActor self.path.address ==> {}", self().path().address());
                    List<ActorRef> managerActorList = new ArrayList<>(3);

                    for (int i = 1; i <= 3; i++){
                        ActorRef manager = context().actorOf(Props.create(ManagerActor.class), "manager_"+ i);
                        managerActorList.add(manager);
                    }

                    for (ActorRef manager: managerActorList){
                        Message.Meeting meeting = new Message.Meeting("来 22 楼上开会，有个重大利好！");
                        Future<Object> ask = Patterns.ask(manager, meeting, Timeout.apply(5, TimeUnit.SECONDS));

                        final CompletionStage<Object> cs = scala.compat.java8.FutureConverters.toJava(ask);

                        CompletableFuture<Object> cf = cs.toCompletableFuture();

                        cf.thenAcceptAsync(obj -> {
                            if (obj instanceof Message.Confirm) {
                                Message.Confirm c = (Message.Confirm) obj;
                                logging.info("收到 Confirm ==> {}", c);
                                //
                                logging.info("c.actorPath.parent.toString ==> {}",
                                        c.getActorPath().parent().toSerializationFormat());
                                //这里c.actorPath是绝对路径,你也可以根据相对路径得到相应的ActorRef
                                ActorSelection mgr = context().actorSelection(c.getActorPath());
                                logging.info("mgr is ==>{}", mgr);
                                mgr.tell(new Message.DoAction("开始干活！"), self());
                            }
                        });
                    }
                })
                .match(Message.Done.class, d -> {
                    taskCount += 1;
                    logging.info("BossActor: taskCount ==>; Done ==> {}", taskCount, d);
                    if (taskCount >= 3) {
                        logging.info("项目完成，我们开始分钱！");
                        context().system().terminate();
                    }
                })
                .matchAny(x -> logging.info("{} 不修电脑！", this))
                .build();

    }
}

```
### Manager

```java

package com.example.myscala002.akka;

import akka.actor.AbstractActor;
import akka.actor.ActorRef;
import akka.actor.IllegalActorStateException;
import akka.actor.Props;
import akka.event.Logging;
import akka.event.LoggingAdapter;
import akka.japi.pf.ReceiveBuilder;
import akka.util.Timeout;
import org.hibernate.validator.internal.util.logging.LoggerFactory;
import scala.PartialFunction;
import scala.runtime.BoxedUnit;

import java.util.concurrent.TimeUnit;

public class ManagerActor extends AbstractActor {

    private final LoggingAdapter logging = Logging.getLogger(context().system(), this);


    @Override
    public PartialFunction<Object, BoxedUnit> receive() {
        return ReceiveBuilder.create()

                .match(Message.Meeting.class, (m) -> {
                    logging.info("ManagerActor 收到 Meeting ==> {}", m);
                    sender().tell(new Message.Confirm("收到", self().path()), self());
                })

                .match(Message.DoAction.class, (a) -> {
                    logging.info("ManagerActor 收到 DoAction ==> {}", a);
                    ActorRef workerActor = context().actorOf(Props.create(WorkerActor.class), "worker");
                    workerActor.forward(a, context());
                })
                .matchAny(x -> logging.info("{} 不修电脑！", this))
                .build();
    }


}

```


### Worker


```java
package com.example.myscala002.akka;

import akka.actor.AbstractActor;
import akka.event.Logging;
import akka.event.LoggingAdapter;
import akka.japi.pf.ReceiveBuilder;
import scala.PartialFunction;
import scala.runtime.BoxedUnit;

public class WorkerActor  extends AbstractActor {

    private final LoggingAdapter logging = Logging.getLogger(context().system(), this);

    @Override
    public PartialFunction<Object, BoxedUnit> receive() {

        return ReceiveBuilder.create()
                .match(Message.DoAction.class, (a) -> {

                    logging.info("我收到任务单了 ==> {}", a);
                    sender().tell(new Message.Done("好了，我干完了。"), self());

                })
                .matchAny(x -> logging.info("{} 不修电脑！", this))
                .build();
    }
}

```

### 启动类

```java
package com.example.myscala002.akka;

import akka.actor.ActorRef;
import akka.actor.ActorSystem;
import akka.actor.Props;

public class AkkaMain {

    public static void main(String[] args) {

        ActorSystem actorSystem = ActorSystem.create();

        ActorRef bossActor = actorSystem.actorOf(Props.create(BossActor.class));

        // Fitness industry has great prospects
        bossActor.tell(new Message.Business("健身产业前景广阔"), ActorRef.noSender());


    }
}


```


## 解释

// TODO: 待补充。



参考： https://www.jianshu.com/p/f8252ae64063


<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
