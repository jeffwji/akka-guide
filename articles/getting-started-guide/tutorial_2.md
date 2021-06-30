# 第 2 部分: 创建第一个 Actor
## 简介

了解了actor的层次结构和Behavior后，剩下的问题是如何将我们的 IoT 系统的顶级组件映射到actor。该 user guardian 可以是代表整个应用程序的 actor。换句话说，我们的物联网系统中将只有一个顶级 actor。创建和管理设备和仪表板的组件将是此actor的子项。我们将示例用例架构图重构为actor 树：

![actor-tree](../../images/getting-started-guide/tutorial_2/actor-tree.png)

我们将用几行代码来定义第一个 actor IotSupervisor 以开始我们的教程：

1. 在`com.example`包中创建一个`IotSupervisor`源文件。
2. 将以下代码粘贴到新文件中以定义 IotSupervisor。

```scala
package com.example

import akka.actor.typed.Behavior
import akka.actor.typed.PostStop
import akka.actor.typed.Signal
import akka.actor.typed.scaladsl.AbstractBehavior
import akka.actor.typed.scaladsl.ActorContext
import akka.actor.typed.scaladsl.Behaviors

object IotSupervisor {
  def apply(): Behavior[Nothing] =
    Behaviors.setup[Nothing](context => new IotSupervisor(context))
}

class IotSupervisor(context: ActorContext[Nothing]) extends AbstractBehavior[Nothing](context) {
  context.log.info("IoT Application started")

  override def onMessage(msg: Nothing): Behavior[Nothing] = {
    // No need to handle any messages
    Behaviors.unhandled
  }

  override def onSignal: PartialFunction[Signal, Behavior[Nothing]] = {
    case PostStop =>
      context.log.info("IoT Application stopped")
      this
  }
}
```
代码类似于我们在前面的实验中使用的 Actor 示例，但请注意我们通过 context.log 来使用 Akka 的内置日志记录工具而不是`println()`。

要提供创建 Actor system的主入口点，请将以下代码添加到新的`IotMain`类中。

```java
package com.example

import akka.actor.typed.ActorSystem

object IotApp {

  def main(args: Array[String]): Unit = {
    // Create ActorSystem and top level supervisor
    ActorSystem[Nothing](IotSupervisor(), "iot-system")
  }

}
```
这个应用程序除了打印它的启动信息之外，几乎没有任何作用。但是，我们已经完成了第一个 Actor 的创建工作，并且准备好添加其他 Actor 了。

## 下一步是什么？
在下面的章节中，我们将通过以下方式逐步扩展应用程序：

- 创建设备的表示。
- 创建设备管理组件。
- 向设备组添加查询功能。

----------

[第 3 部分：使用设备 Actor](tutorial_3.md)



**英文原文链接**：[Part 2: Creating the First Actor](https://doc.akka.io/docs/akka/current/guide/tutorial_2.html).

----------
———— ☆☆☆ —— [返回 -> Akka 中文指南 <- 目录](https://github.com/guobinhit/akka-guide/blob/master/README.md) —— ☆☆☆ ————