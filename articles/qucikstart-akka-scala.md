# 快速入门 Akka Scala 指南

Akka 是一个用于在 JVM 上构建高并发、分布式和容错的事件驱动应用程序的运行时工具包。Akka 既可以用于 Java，也可以用于 Scala。本指南通过描述 Scala 版本的`Hello World`示例来介绍 Akka。如果你喜欢将 Akka 与 Java 结合使用，请切换到「[快速入门 Akka Java 指南](quickstart-akka-java.md)」。

Actors 是 Akka 的执行单元。Actor 模型是一种抽象，它让编写正确的并发、并行和分布式系统更容易。`Hello World`示例说明了 Akka 的基础知识。在 30 分钟内，你应该能够下载并运行示例，并使用本指南了解示例是如何构造的。这会让你初步了解 Akka 的魅力，希望这能够让你拥有深入了解 Akka 的兴趣！

在体验过这个示例之后，想深入了解 Akka，阅读「[Getting Started Guide](getting-started-guide/introduction-to-akka.md)」是一个很好的选择，可以继续学习更多关于 Akka 的知识。

[Akka 平台指南](https://developer.lightbend.com/docs/akka-platform-guide/)有更多关于Akka的概念和特点的讨论，并提供了一个 Akka 工具包的概述。

## 下载示例
Scala 版本的`Hello World` 示例是一个包含sbt （构建工具）发行版的压缩项目。你可以在 Linux、MacOS 或 Windows 上运行它。唯一的先决条件是安装 Java 8。

下载和解压示例：

- 下载[zip 文件](https://example.lightbend.com/v1/download/akka-quickstart-scala?name=akka-quickstart-scala)并将其解压缩到一个方便的位置：

- 在 Linux 和 OSX 系统上，打开终端并使用命令`unzip akka-quickstart-scala.zip`. 注意：在 OSX 上，如果您使用 Archiver 解压缩，您还必须使 sbt 文件可执行：

  ```sh
  $ chmod u+x ./sbt
  $ chmod u+x ./sbt-dist/bin/sbt
  ```

- 在 Windows 上，使用诸如文件资源管理器之类的工具来提取项目。

## 运行示例
运行 Hello World：

1. 在控制台中，将目录更改为解压缩项目的顶层。

   例如，如果您使用默认项目名称 akka-quickstart-scala，并将项目解压到您的根目录，则从根目录输入： `cd akka-quickstart-scala`

2. 在 OSX/Linux 上输入 `./sbt` 或在Windows上输入`sbt.bat` 以启动 sbt。

   sbt 会下载项目依赖。`>`提示符表明SBT在交互模式已经开始。

3. 在 sbt 提示符下，输入`reStart`.

   sbt 构建项目并运行 Hello World

输出*的东西*应该像这样（滚动到右边查看Actor 的输出）：

```
[2019-10-09 09:55:23,390] [INFO ] [com.example.Greeter$] [AkkaQuickStart-akka.actor.default-dispatcher-5]
[akka://AkkaQuickStart/user/greeter] - Hello Charles!
[2019-10-09 09:55:23,392] [INFO ] [com.example.GreeterBot$] [AkkaQuickStart-akka.actor.default-dispatcher-3]
[akka://AkkaQuickStart/user/Charles] - Greeting 1 for Charles
[2019-10-09 09:55:23,392] [INFO ] [com.example.Greeter$] [AkkaQuickStart-akka.actor.default-dispatcher-5]
[akka://AkkaQuickStart/user/greeter] - Hello Charles!
[2019-10-09 09:55:23,392] [INFO ] [com.example.GreeterBot$] [AkkaQuickStart-akka.actor.default-dispatcher-3]
[akka://AkkaQuickStart/user/Charles] - Greeting 2 for Charles
[2019-10-09 09:55:23,392] [INFO ] [com.example.Greeter$] [AkkaQuickStart-akka.actor.default-dispatcher-5]
[akka://AkkaQuickStart/user/greeter] - Hello Charles!
[2019-10-09 09:55:23,392] [INFO ] [com.example.GreeterBot$] [AkkaQuickStart-akka.actor.default-dispatcher-3]
[akka://AkkaQuickStart/user/Charles] - Greeting 3 for Charles
```

恭喜你，你刚刚运行了你的第一个 Akka 应用程序。

## Hello World 都做了什么？
该示例由三个参与者组成：

- Greet：接收命令并对某人发出`Greet`问候，然后返回`Greeted`以确认问候已经发出。
- GreeterBot：接收来自问候者的回复并发送一些额外的问候消息并收集回复，直到达到给定的最大消息数。
- GreeterMain：执行所有动作的指导 actor

## 使用 Actor 模型的好处
Akka 的以下特性使你能够以直观的方式解决困难的并发性和可伸缩性挑战：

- 事件驱动模型：`Event-driven model`，Actors 通过响应消息来执行工作。Actors 之间的通信是异步的，允许 Actors 发送消息并继续自己的工作，而不是阻塞等待响应。
- 强隔离原则：`Strong isolation principles`，与 Java 中的常规对象不同，Actors 在调用的方法方面，没有一个公共 API。相反，它的公共 API 是通过 Actors 处理的消息来定义的。这可以防止 Actors 之间共享状态；观察另一个 Actors 状态的唯一方法是向其发送请求状态的消息。
- 位置透明：`Location transparency`，系统通过工厂方法构造 Actors 并返回对实例的引用。因为位置无关紧要，所以 Actors 实例可以启动、停止、移动和重新启动，以向上和向下扩展以及从意外故障中恢复。
- 轻量级：`Lightweight`，每个实例只消耗几百个字节，这实际上允许数百万并发 Actors 存在于一个应用程序中。

让我们看看在`Hello World`示例中使用 Actors 和消息一起工作的一些最佳实践。

## 定义 Actors 和消息

每个actor都定义了一个类型为 T 的可接收的消息类型。类型类和类型对象可以生成完美的消息，因为它们是不可变的并且支持模式匹配，当匹配收到的消息时，我们将在 Actor 中利用这个优点。

`Hello World`的 Actors 使用三种不同的消息：

- `Greet`: 发送给问候者 actor 的问候指令。
- `Greeted`: 问候者 actor 返回的消息已经发出的确认。
- `SayHello`: 发送给 `GreeterMain` 的启动问候流程的指令。

在定义 Actors 及其消息时，请记住以下建议：

- 因为消息是 Actor 的公共 API，所以定义具有良好名称、丰富语义和特定于域的含义的消息是一个很好的实践，即使它们只是包装你的数据类型，这将使基于 Actor 的系统更容易使用、理解和调试。
- 消息应该是不可变的，因为它们在不同的线程之间共享。
- 将 Actor 的关联消息作为静态类放在 Actor 的类中是一个很好的实践，这使得理解 Actor 期望和处理的消息类型更加容易。
- 在对象的 apply 方法中执行 actor 的初始行为是一个很好的做法。

让我们看看`Greeter`，`GreeterBot`和`GreeterMain` 的对象和`Behavior` 是如何演示这些最佳实践的。

###  Greeter Actor
下面的代码段来自于`Greeter.java`，其实现了`Greeter Actor`：

```scala
object Greeter {
  final case class Greet(whom: String, replyTo: ActorRef[Greeted])
  final case class Greeted(whom: String, from: ActorRef[Greet])

  def apply(): Behavior[Greet] = Behaviors.receive { (context, message) =>
    context.log.info("Hello {}!", message.whom)
    message.replyTo ! Greeted(message.whom, context.self)
    Behaviors.same
  }
}
```

这段代码定义了两种消息类型，一种用于命令 Actor 向某人打招呼，另一种是 Actor 将用来确认它已经这样做了。该`Greet`类型不仅包含要问候的人的信息，还包含一个`ActorRef`对象来指向消息的发送者，以便`Greeter`Actor 可以发回确认消息。

Actor 的行为在`Behaviors.receive`行为工厂的帮助下被定义为`Greeter`。处理完消息后，返回新的保持不可变状态的`Behavior`来更新当前的状态，它可以是与当前`Behavior`不同的新的`Behavior`。由于在这个应用中，我们不需要更新任何状态，因此我们返回`Behaviors.same`，这意味着下一个行为“与当前`Behavior`相同”。

此`Behavior`处理的消息类型参数被声明为 class `Greet`，这意味着该`message`参数是此类型的。这就是为什么我们可以访问`whom`和`replyTo`成员而无需使用模式匹配。通常，一个 Actor 可能处理不止一种特定的消息类型，它们都通过扩展（`extends`）同一个特质来生成。

在最后一行，我们看到`Greeter`Actor 使用`!`操作符（发音为“bang”或“tell”）向另一个 Actor 发送消息。它是一个异步操作，不会阻塞调用者的线程。

由于`replyTo`地址被声明为 type `ActorRef[Greeted]`，编译器将只允许我们发送这种类型的消息，其他用法将是编译器错误。

Actor 接受的消息类型以及所有回复类型定义了此 Actor 所说的协议；在这个例子中，它是一个简单的请求-回复协议，但参与者可以在需要时为任意复杂的协议建模。该协议与它的行为被捆绑在一起并很好地封装在一个范围内——`Greeter`对象。

### The Greeter bot actor

```scala
object GreeterBot {

  def apply(max: Int): Behavior[Greeter.Greeted] = {
    bot(0, max)
  }

  private def bot(greetingCounter: Int, max: Int): Behavior[Greeter.Greeted] =
    Behaviors.receive { (context, message) =>
      val n = greetingCounter + 1
      context.log.info("Greeting {} for {}", n, message.whom)
      if (n == max) {
        Behaviors.stopped
      } else {
        message.from ! Greeter.Greet(message.whom, context.self)
        bot(n, max)
      }
    }
}
```

请注意此 Actor 是如何通过为`Greeted`生成新的`behavior`，而不是使用变量来，管理计数器的。由于actor实例一次处理一条消息，因此不需要使用例如`synchronized`或`AtomicInteger`来保护并发。

### The Greeter main actor

第三个 actor 生成`Greeter`和`GreeterBot`并开始交互，接下来我们将讨论创建 actor 以及`spawn`的内容。

```scala
object GreeterMain {
  final case class SayHello(name: String)

  def apply(): Behavior[SayHello] =
    Behaviors.setup { context =>
      val greeter = context.spawn(Greeter(), "greeter")

      Behaviors.receiveMessage { message =>
        val replyTo = context.spawn(GreeterBot(max = 3), message.name)
        greeter ! Greeter.Greet(message.name, replyTo)
        Behaviors.same
      }
    }
}
```

## 创建 Actors
到目前为止，我们已经了解了 Actors 和他们的消息的定义。现在，让我们更深入地了解位置透明（`location transparency`）的好处，看看如何创建 Actor 实例。

### 位置透明的好处
在 Akka 中，不能使用`new`关键字创建 Actor 的实例。相反，你应该使用 `spawn` 工厂方法创建 Actor 实例。工厂不返回 Actor 实例，而是返回指向 Actor 实例的引用`akka.actor.typed.ActorRef`。在分布式系统中，这种间接创建实例的方法增加了很多好处和灵活性。

在 Akka 中位置是无关紧要的。位置透明性意味着，无论是在当前运行的 Actor 的进程内，还是运行在远程计算机上，`ActorRef`都可以保持相同语义。如果需要，运行时可以通过在更改 Actor 的位置或整个应用程序拓扑来优化系统。这就启用了故障管理的“让它崩溃（`let it crash`）”模型，在该模型中，系统可以通过销毁有问题的 Actor 和重新启动健康的 Actor 来自我修复。

### Akka ActorSystem
`ActorSystem`是 Akka 的初始入口点。通常`ActorSystem`每个应用程序只创建一个。ActorSystem`有一个名字和一个监护 actor。您的应用程序的引导程序通常在监护 actor 中完成。

本例的监护 actor `ActorSystem`是`GreeterMain`。

```scala
val greeterMain: ActorSystem[GreeterMain.SayHello] = ActorSystem(GreeterMain(), "AkkaQuickStart")
```

它通过`Behaviors.setup`引导应用程序。

```scala
object GreeterMain {
  final case class SayHello(name: String)

  def apply(): Behavior[SayHello] = Behaviors.setup { context =>
      val greeter = context.spawn(Greeter(), "greeter")

      Behaviors.receiveMessage { message =>
        val replyTo = context.spawn(GreeterBot(max = 3), message.name)
        greeter ! Greeter.Greet(message.name, replyTo)
        Behaviors.same
      }
    }
}
```

其他角色是使用`ActorContext` 上的`spawn`方法创建。`GreeterMain`在启动时以这种方式创建一个`Greeter`actor，并在每次收到`SayHello`消息时创建一个新的 `GreeterBot` actor。

## 异步通信
 Actors 是反应式的和消息驱动的。Actor 在收到消息前不做任何事。Actors 使用异步消息进行通信。这样可以确保发送者不会一直等待接收者处理他们的消息。相反，发件人将邮件放在收件人的邮箱之后，就可以自由地做其他工作。Actor 的邮箱本质上是一个具有排序语义的消息队列。从同一个actor发送的多条消息的顺序被保留，但可以与另一个 Actor 发送的消息交错。

你可能想知道 Actor 在不处理消息的时候在做什么，实际上，它处于挂起状态，不消耗除内存之外的任何资源。这也展示了 Actors 的轻量级和高效性。

### 给 Actor 发送消息
使用`ActorRef`的`!`（bang）方法把消息到 actor 的邮箱。例如，向 Hello World 的主类`GreeterMain` Actor发送消息是这样的：

```scala
greeterMain ! SayHello("Charles")
```
`Greeter `actor 也发送回一条消息以确认它已经收到了问候：

```java
message.replyTo ! Greeted(message.whom, context.self)
```
我们已经研究了如何创建 Actor 和发送消息。现在，让我们来回顾一下`Main`类的全部内容。

## Main class

`Hello World`的 `AkkaQuickstart`类创建  `ActorSystem`  并赋予它监护者 actor（`GreeterMain`）。这个监护者是顶级 actor，它启动你的应用程序，它通常具备一个 `Behaviors.setup` 来定义如何引导初始化过程。

```scala
object AkkaQuickstart extends App {
  val greeterMain: ActorSystem[GreeterMain.SayHello] = ActorSystem(GreeterMain(), "AkkaQuickStart")

  greeterMain ! SayHello("Charles")
}
```
## 完整代码
看一下 actor 的 behavior、消息的定义以及如何启动`ActorSystem`：

```java
//#full-example
package $package$


import akka.actor.typed.ActorRef
import akka.actor.typed.ActorSystem
import akka.actor.typed.Behavior
import akka.actor.typed.scaladsl.Behaviors
import $package$.GreeterMain.SayHello

//#greeter-actor
object Greeter {
  final case class Greet(whom: String, replyTo: ActorRef[Greeted])
  final case class Greeted(whom: String, from: ActorRef[Greet])

  def apply(): Behavior[Greet] = Behaviors.receive { (context, message) =>
    context.log.info("Hello {}!", message.whom)
    //#greeter-send-messages
    message.replyTo ! Greeted(message.whom, context.self)
    //#greeter-send-messages
    Behaviors.same
  }
}
//#greeter-actor

//#greeter-bot
object GreeterBot {

  def apply(max: Int): Behavior[Greeter.Greeted] = {
    bot(0, max)
  }

  private def bot(greetingCounter: Int, max: Int): Behavior[Greeter.Greeted] =
    Behaviors.receive { (context, message) =>
      val n = greetingCounter + 1
      context.log.info("Greeting {} for {}", n, message.whom)
      if (n == max) {
        Behaviors.stopped
      } else {
        message.from ! Greeter.Greet(message.whom, context.self)
        bot(n, max)
      }
    }
}
//#greeter-bot

//#greeter-main
object GreeterMain {

  final case class SayHello(name: String)

  def apply(): Behavior[SayHello] =
    Behaviors.setup { context =>
      //#create-actors
      val greeter = context.spawn(Greeter(), "greeter")
      //#create-actors

      Behaviors.receiveMessage { message =>
        //#create-actors
        val replyTo = context.spawn(GreeterBot(max = 3), message.name)
        //#create-actors
        greeter ! Greeter.Greet(message.name, replyTo)
        Behaviors.same
      }
    }
}
//#greeter-main

//#main-class
object AkkaQuickstart extends App {
  //#actor-system
  val greeterMain: ActorSystem[GreeterMain.SayHello] = ActorSystem(GreeterMain(), "AkkaQuickStart")
  //#actor-system

  //#main-send-messages
  greeterMain ! SayHello("Charles")
  //#main-send-messages
}
//#main-class
//#full-example
```
另一个最佳实践是我们应该提供一些单元测试。

## 测试 Actors
`Hello World`示例中的测试展示了 JUnit 框架的使用。虽然测试的覆盖范围不完整，但它简单地展示了测试 Actor 代码是多么的容易，并提供了一些基本概念。你可以把它作为一个练习来增加你自己的知识。

### 测试类定义

```scala
class AkkaQuickstartSpec extends ScalaTestWithActorTestKit with AnyWordSpecLike {
```

通过扩展`ScalaTestWithActorTestKit`来支持 ScalaTest。对于其他测试框架，可以直接使用 testkit，请参阅[完整文档](https://doc.akka.io/docs/akka/2.6/typed/testing-async.html)。

我们将在测试案例中用它（ ScalaTest）来管理ActorTestKit 的生命周期。

### 测试方法

此测试使用 TestProbe 来询问和验证预期行为。让我们看一个源代码片段：

测试类使用的是`akka.test.javadsl.TestKit`，它是用于 Actor 和 Actor 系统集成测试的模块。这个类只使用了`TestKit`提供的一部分功能。

```scala
"reply to greeted" in {
  val replyProbe = createTestProbe[Greeted]()
  val underTest = spawn(Greeter())
  underTest ! Greet("Santa", replyProbe.ref)
  replyProbe.expectMessage(Greeted("Santa", underTest.ref))
}
```

一旦我们有了对 TestProbe 的引用，我们就将它作为`Greet`消息的一部分传递给 Greeter 。然后我们验证`Greeter`是否对已发生出的问候进行响应。

### 完整的测试代码

这是完整的测试代码：

```scala
package $package$

import akka.actor.testkit.typed.scaladsl.ScalaTestWithActorTestKit
import $package$.Greeter.Greet
import $package$.Greeter.Greeted
import org.scalatest.wordspec.AnyWordSpecLike

class AkkaQuickstartSpec extends ScalaTestWithActorTestKit with AnyWordSpecLike {

  "A Greeter" must {
    "reply to greeted" in {
      val replyProbe = createTestProbe[Greeted]()
      val underTest = spawn(Greeter())
      underTest ! Greet("Santa", replyProbe.ref)
      replyProbe.expectMessage(Greeted("Santa", underTest.ref))
    }
  }
}
```

示例代码只是触及了`ActorTestKit`. 可以在[此处](https://doc.akka.io/docs/akka/2.6/typed/testing-async.html)找到完整的概述。

## 运行应用程序

你可以通过命令行或者 IDE 来运行`Hello World`应用程序。在本指南的最后一个主题，我们描述了如何在 IntelliJ IDEA 中运行该示例。但是，在我们再次运行应用程序之前，让我们先快速的查看构建工具 sbt。

### 构建文件

sbt 使用一个`build.sbt`文件来处理项目。该项目的`build.sbt`文件如下所示：

```scala
name := "akka-quickstart-scala"

version := "1.0"

scalaVersion := "2.13.1"

lazy val akkaVersion = "$akka_version$"

libraryDependencies ++= Seq(
  "com.typesafe.akka" %% "akka-actor-typed" % akkaVersion,
  "ch.qos.logback" % "logback-classic" % "1.2.3",
  "com.typesafe.akka" %% "akka-actor-testkit-typed" % akkaVersion % Test,
  "org.scalatest" %% "scalatest" % "3.1.0" % Test
)
```

这个构建文件非常简单。本质上，它创建了一个项目`hello-akka-scala`并声明了项目依赖项。我们还必须声明要使用的 sbt 版本，这在文件中完成`project/build.properties`：

```properties
sbt.version=1.4.7
```

### 运行项目
和前面一样，从控制台运行应用程序：

1. OSX/Linux 中输入 `./sbt`或在Windows 上输入`sbt.bat`

sbt 将下载项目依赖。`>`提示符表明SBT的交互模式已经开始。

1. 在 sbt 提示符下，输入`reStart`.

输出应该如下所示（一直向右滚动以查看 Actor 输出）：

```
[2019-10-09 09:55:23,390] [INFO] [com.example.Greeter$] [AkkaQuickStart-akka.actor.default-dispatcher-5]
[akka://AkkaQuickStart/user/greeter] - Hello Charles!
[2019-10-09 09:55:23,392] [INFO] [com.example.GreeterBot$] [AkkaQuickStart-akka.actor.default-dispatcher-3]
[akka://AkkaQuickStart/user/Charles] - Greeting 1 for Charles
[2019-10-09 09:55:23,392] [INFO] [com.example.Greeter$] [AkkaQuickStart-akka.actor.default-dispatcher-5]
[akka://AkkaQuickStart/user/greeter] - Hello Charles!
[2019-10-09 09:55:23,392] [INFO] [com.example.GreeterBot$] [AkkaQuickStart-akka.actor.default-dispatcher-3]
[akka://AkkaQuickStart/user/Charles] - Greeting 2 for Charles
[2019-10-09 09:55:23,392] [INFO] [com.example.Greeter$] [AkkaQuickStart-akka.actor.default-dispatcher-5]
[akka://AkkaQuickStart/user/greeter] - Hello Charles!
[2019-10-09 09:55:23,392] [INFO] [com.example.GreeterBot$] [AkkaQuickStart-akka.actor.default-dispatcher-3]
[akka://AkkaQuickStart/user/Charles] - Greeting 3 for Charles
```
还记得`Greeter`Actor的实现使用了`ActorContext`吗？它提供了很多额外的信息。例如，它的日志输出包含对象行为的时间和名称。

要运行测试，请`test`在 sbt 提示符下输入。

### 下一步

如果您使用 IntelliJ，请尝试将示例项目与[IntelliJ IDEA](https://developer.lightbend.com/guides/akka-quickstart-scala/intellij-idea.html)集成。

要继续了解有关 Akka 和 Actor 系统的更多信息，请查看接下来的[入门指南](../README.md)。Happy hakking！

----------

**英文原文链接**：[Akka Quickstart with Java](https://developer.lightbend.com/guides/akka-quickstart-java/index.html).

----------
———— ☆☆☆ —— [返回 -> Akka 中文指南 <- 目录](https://github.com/guobinhit/akka-guide/blob/master/README.md) —— ☆☆☆ ————

