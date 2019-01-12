---
title: Actor 学习笔记
key: 20190108
tags: scala, akka, actor
---

维基百科这样定义Actor模型：

> 
> 在计算科学领域，Actor模型是一个并行计算（Concurrent Computation）模型，它把 actor 作为并行计算的基本元素来对待：为响应一个接收到的消息，一个 actor 能够自己做出一些决策，如创建更多的 actor，或发送更多的消息，或者确定如何去响应接收到的下一个消息。 
> 

<!--more-->

我们先看一下 Akka 中 Actor 的定义：

```scala

trait Actor {
 
  import Actor._
 
  type Receive = Actor.Receive
 
  implicit val context: ActorContext = {
    val contextStack = ActorCell.contextStack.get
    if ((contextStack.isEmpty) || (contextStack.head eq null))
      throw ActorInitializationException(
        s"You cannot create an instance of [${getClass.getName}] explicitly using the constructor (new). " +
          "You have to use one of the 'actorOf' factory methods to create a new actor. See the documentation.")
    val c = contextStack.head
    ActorCell.contextStack.set(null :: contextStack)
    c
  }
 
  implicit final val self = context.self //MUST BE A VAL, TRUST ME
 
  final def sender(): ActorRef = context.sender()

  /* 这个是在子类中一定要实现的抽象方法 */ 
  def receive: Actor.Receive 
 
  protected[akka] def aroundReceive(receive: Actor.Receive, msg: Any): Unit = receive.applyOrElse(msg, unhandled)
 
  protected[akka] def aroundPreStart(): Unit = preStart()
 
  protected[akka] def aroundPostStop(): Unit = postStop()
 
  protected[akka] def aroundPreRestart(reason: Throwable, message: Option[Any]): Unit = preRestart(reason, message)
 
  protected[akka] def aroundPostRestart(reason: Throwable): Unit = postRestart(reason)
 
  def supervisorStrategy: SupervisorStrategy = SupervisorStrategy.defaultStrategy
 
  /* when changing this you MUST also change UntypedActorDocTest */
  /* 启动Actor之前需要执行的操作，默认为空实现，可以重写该方法 */
  @throws(classOf[Exception]) 
  def preStart(): Unit = () 
 
  /* when changing this you MUST also change UntypedActorDocTest */
  /* 终止Actor之前需要执行的操作，默认为空实现，可以重写该方法 */
  @throws(classOf[Exception]) 
  def postStop(): Unit = () 
 
  /* when changing this you MUST also change UntypedActorDocTest */
  @throws(classOf[Exception]) 
  def preRestart(reason: Throwable, message: Option[Any]): Unit = { 
    /* 重启Actor之前需要执行的操作，默认终止该Actor所监督的所有子Actor，
     * 然后调用postStop()方法，可以重写该方法
     */
    context.children foreach { child ⇒
      context.unwatch(child)
      context.stop(child)
    }
    postStop()
  }
 
  /* when changing this you MUST also change UntypedActorDocTest */
  @throws(classOf[Exception]) 
  def postRestart(reason: Throwable): Unit = {  
    /* 重启Actor之前需要执行的操作，默认执行preStart()的实现逻辑，
     * 可以重写该方法 
     */
    preStart()
  }
 
  def unhandled(message: Any): Unit = {
    message match {
      case Terminated(dead) ⇒ throw new DeathPactException(dead)
      case _                ⇒ context.system.eventStream.publish(UnhandledMessage(message, sender(), self))
    }
  }
}

```

参考文档：

- [Akka框架基本要点介绍](http://shiyanjun.cn/archives/1168.html)

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
