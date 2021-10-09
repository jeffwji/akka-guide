# Actor 生命周期
## 依赖

为了使用 Akka Actor 类型，你需要将以下依赖添加到你的项目中：

```xml
<!-- Maven -->
<properties>
  <scala.binary.version>2.13</scala.binary.version>
</properties>
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>com.typesafe.akka</groupId>
      <artifactId>akka-bom_${scala.binary.version}</artifactId>
      <version>2.6.15</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
<dependencies>
  <dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-actor-typed_${scala.binary.version}</artifactId>
  </dependency>
</dependencies>

<!-- Gradle -->
def versions = [
  ScalaBinary: "2.13"
]
dependencies {
  implementation platform("com.typesafe.akka:akka-bom_${versions.ScalaBinary}:2.6.15")

  implementation "com.typesafe.akka:akka-actor-typed_${versions.ScalaBinary}"
}

<!-- sbt -->
val AkkaVersion = "2.6.15"
libraryDependencies += "com.typesafe.akka" %% "akka-actor-typed" % AkkaVersion
```

## 介绍

Actor 是一种有状态的资源，必须显式启动和停止。

需要注意的是，Actor 在不再被引用时不会自动停止，每个创建的 Actor 也必须显式销毁。唯一的简化是停止父 Actor 时也会递归地停止该父级创建的所有子 Actor。`ActorSystem`关闭时，所有Actor也会自动停止。

```
提示:

ActorSystem 是一种用于分配线程的重量级结构，所以请为每个逻辑应用程序创建一个。通常每个JVM 进程只需要一个 ActorSystem。
```

## 创建Actors

一个 Actor 可以创建或生成任意数量的子 Actor，而子 Actor 又可以生成自己的子 Actor，从而形成一个 Actor 层次。「[ActorSystem](https://doc.akka.io/japi/akka/2.5/?akka/actor/typed/ActorSystem.html)」承载层次结构，并且在`ActorSystem`层次结构的顶部只能有一个根 Actor。一个子 Actor 的生命周期是与其父 Actor 联系在一起的，一个子 Actor 可以在任何时候停止自己或被停止，但永远不能比父 Actor 活得更久。

### ActorContext

可以出于多种目的访问 ActorContext，例如：

- 监督和产生子Actor
- 观察其它Actor并接收`Terminated(otherActor)`事件当被观察的Actor永久停止时。
- 记录日志
- 创建消息适配器
- 与另一个Actor的请求-响应交互（ask）
- 访问`self` ActorRef

如果一个行为需要使用`ActorContext`，例如使用 `context.self`产生子actor，可以通过在`Behaviors.setup`中使用包装构造来获得：

```scala
object HelloWorldMain {

  final case class SayHello(name: String)

  def apply(): Behavior[SayHello] =
    Behaviors.setup { context =>
      val greeter = context.spawn(HelloWorld(), "greeter")

      Behaviors.receiveMessage { message =>
        val replyTo = context.spawn(HelloWorldBot(max = 3), message.name)
        greeter ! HelloWorld.Greet(message.name, replyTo)
        Behaviors.same
      }
    }
}
```

#### ActorContext 的线程安全性

`ActorContext`中的许多方法不是线程安全的，并且

- 不能被`scala.concurrent.Future`中的回调线程访问
- 不得在多个 actor 实例之间共享
- 只能在普通actor消息处理线程中使用

### 守护者 Actor
顶级 Actor，也称为 user 守护者 Actor，与`ActorSystem`一起创建。发送到 Actor 系统的消息被定向到根 Actor。根 Actor 是由用于创建`ActorSystem`的行为生成的，例如下面的示例中名为`HelloWorldMain`的Actor：

```scala
val system: ActorSystem[HelloWorldMain.SayHello] =
  ActorSystem(HelloWorldMain(), "hello")

system ! HelloWorldMain.SayHello("World")
system ! HelloWorldMain.SayHello("Akka")
```

对于非常简单的应用程序，监护人可能包含实际的应用程序逻辑和消息处理。但是一旦应用程序将处理多个问题，监护人就应该只负责引导，生成子系统，并监视它们的生命周期。

一旦监护人Actor停止，这也将停止`ActorSystem`.

当`ActorSystem.terminate`被调用时，[协调关闭](coordinated-shutdown.md)流程将按特定顺序停止Actor和服务。

### 繁衍子级

子 Actor 是由「[ActorContext](https://doc.akka.io/japi/akka/2.5/?akka/actor/typed/javadsl/ActorContext.html)」“繁殖（`spawn`）”产生的。在下面的示例中，当根 Actor 启动时，它生成一个由`HelloWorld`行为描述的子 Actor。此外，当根 Actor 收到`SayHello`消息时，它将创建由`HelloWorldBot`行为定义的子 Actor：

```scala
object HelloWorldMain {

  final case class SayHello(name: String)

  def apply(): Behavior[SayHello] =
    Behaviors.setup { context =>
      val greeter = context.spawn(HelloWorld(), "greeter")

      Behaviors.receiveMessage { message =>
        val replyTo = context.spawn(HelloWorldBot(max = 3), message.name)
        greeter ! HelloWorld.Greet(message.name, replyTo)
        Behaviors.same
      }
    }

}
```

要在生成 Actor 时指定调度器，请使用「[DispatcherSelector](DispatcherSelector.md)」。如果未指定，则 Actor 将使用默认调度器，有关详细信息，请参阅「[默认调度器](dispatchers.md#default-dispatcher)」。

```scala
def apply(): Behavior[SayHello] =
  Behaviors.setup { context =>
    val dispatcherPath = "akka.actor.default-blocking-io-dispatcher"

    val props = DispatcherSelector.fromConfig(dispatcherPath)
    val greeter = context.spawn(HelloWorld(), "greeter", props)

    Behaviors.receiveMessage { message =>
      val replyTo = context.spawn(HelloWorldBot(max = 3), message.name)

      greeter ! HelloWorld.Greet(message.name, replyTo)
      Behaviors.same
    }
  }
```

通过参阅「[Actors](actors.md#introduction)」，以了解上述示例。

### SpawnProtocol

守护者(Guardian) Actor 应该负责初始化任务并创建应用程序的初始 Actor，但有时你可能希望从guardian actor 的外部生成新的 Actor。例如，为每个 HTTP 请求创建一个 Actor。

在您的行为（Behavior）中实现这并不难，因为这是一个常见的模式，所以有一个预定义的消息协议（`SpawnProtocol.Command`）和行为的实现（`SpawnProtocol`）可以用作`ActorSystem`的守护者 Actor，可以与`Behaviors.setup`结合使用以启动一些初始任务或 Actor。然后，可以通过`tell`或 `ask`来调用具有系统 Actor引用的 `SpawnProtocol.Spawn`从而从外部生成子Actor。使用`ask`时，这类似于在经典 Actor 中使用`ActorSystem.actorOf`，不同之处在于它返回一个 `Future`的`ActorRef`。

守护者行为可定义为：

```scala
import akka.actor.typed.Behavior
import akka.actor.typed.SpawnProtocol
import akka.actor.typed.scaladsl.Behaviors
import akka.actor.typed.scaladsl.LoggerOps

object HelloWorldMain {
  def apply(): Behavior[SpawnProtocol.Command] =
    Behaviors.setup { context =>
      // Start initial tasks
      // context.spawn(...)

      SpawnProtocol()
    }
}
```

而`ActorSystem`可以在`main`的行为中来创建，并生成其他 Actor：

```scala
import akka.actor.typed.ActorRef
import akka.actor.typed.ActorSystem
import akka.actor.typed.Props
import akka.util.Timeout


implicit val system: ActorSystem[SpawnProtocol.Command] =
  ActorSystem(HelloWorldMain(), "hello")

// needed in implicit scope for ask (?)
import akka.actor.typed.scaladsl.AskPattern._
implicit val ec: ExecutionContext = system.executionContext
implicit val timeout: Timeout = Timeout(3.seconds)

val greeter: Future[ActorRef[HelloWorld.Greet]] =
  system.ask(SpawnProtocol.Spawn(behavior = HelloWorld(), name = "greeter", props = Props.empty, _))

val greetedBehavior = Behaviors.receive[HelloWorld.Greeted] { (context, message) =>
  context.log.info2("Greeting for {} from {}", message.whom, message.from)
  Behaviors.stopped
}

val greetedReplyTo: Future[ActorRef[HelloWorld.Greeted]] =
  system.ask(SpawnProtocol.Spawn(greetedBehavior, name = "", props = Props.empty, _))

for (greeterRef <- greeter; replyToRef <- greetedReplyTo) {
  greeterRef ! HelloWorld.Greet("Akka", replyToRef)
}
```

可以在 Actor 层次结构的其他位置使用`SpawnProtocol`，而不是必须在根守护者 Actor 中。

[Actor发现](actor-discovery.md)中描述了一种查找正在运行的actor的方法。

## 停止 Actors

Actor 可以通过返回`Behaviors.stopped`作为下一个行为来停止自己。

通过使用父 Actor 的`ActorContext`的`stop`方法，可以在子 Actor 完成当前消息的处理后强制停止它。只有当它是子 Actor 才能这样做。

所有的子 Actor 在父级停止时也会被停止。

当一个 Actor 停止时，它将接收到`PostStop`信号以便于它清除资源。

下面是一个示例：

```scala
import akka.actor.typed.Behavior
import akka.actor.typed.scaladsl.Behaviors
import akka.actor.typed.{ ActorSystem, PostStop }


object MasterControlProgram {
  sealed trait Command
  final case class SpawnJob(name: String) extends Command
  case object GracefulShutdown extends Command

  def apply(): Behavior[Command] = {
    Behaviors
      .receive[Command] { (context, message) =>
        message match {
          case SpawnJob(jobName) =>
            context.log.info("Spawning job {}!", jobName)
            context.spawn(Job(jobName), name = jobName)
            Behaviors.same
          case GracefulShutdown =>
            context.log.info("Initiating graceful shutdown...")
            // Here it can perform graceful stop (possibly asynchronous) and when completed
            // return `Behaviors.stopped` here or after receiving another message.
            Behaviors.stopped
        }
      }
      .receiveSignal {
        case (context, PostStop) =>
          context.log.info("Master Control Program stopped")
          Behaviors.same
      }
  }
}

object Job {
  sealed trait Command

  def apply(name: String): Behavior[Command] = {
    Behaviors.receiveSignal[Command] {
      case (context, PostStop) =>
        context.log.info("Worker {} stopped", name)
        Behaviors.same
    }
  }
}
```

## 关注Actor

为了在另一个 actor 终止（永久停止，而不是临时故障和重启）时得到通知，一个 actor 可以使用`watch`关注另一个actor。它将在被关注的actor终止（参见[停止actor](actor-lifecycle.md#stopping-actors)）时接收到[`Terminated`](Terminated.md)信号。

```scala
object MasterControlProgram {
  sealed trait Command
  final case class SpawnJob(name: String) extends Command

  def apply(): Behavior[Command] = {
    Behaviors
      .receive[Command] { (context, message) =>
        message match {
          case SpawnJob(jobName) =>
            context.log.info("Spawning job {}!", jobName)
            val job = context.spawn(Job(jobName), name = jobName)
            context.watch(job)
            Behaviors.same
        }
      }
      .receiveSignal {
        case (context, Terminated(ref)) =>
          context.log.info("Job stopped: {}", ref.path.name)
          Behaviors.same
      }
  }
}
```

`watch`is的替代方法`watchWith`，它允许指定自定义消息而不是`Terminated`。这通常比使用`watch`和`Terminated`信号更可取，因为可以在消息中包含附加信息，以便稍后在接收时使用。

与上面类似的示例，但使用`watchWith`以便在任务完成时回复最初的请求者。

```scala
object MasterControlProgram {
  sealed trait Command
  final case class SpawnJob(name: String, replyToWhenDone: ActorRef[JobDone]) extends Command
  final case class JobDone(name: String)
  private final case class JobTerminated(name: String, replyToWhenDone: ActorRef[JobDone]) extends Command

  def apply(): Behavior[Command] = {
    Behaviors.receive { (context, message) =>
      message match {
        case SpawnJob(jobName, replyToWhenDone) =>
          context.log.info("Spawning job {}!", jobName)
          val job = context.spawn(Job(jobName), name = jobName)
          context.watchWith(job, JobTerminated(jobName, replyToWhenDone))
          Behaviors.same
        case JobTerminated(jobName, replyToWhenDone) =>
          context.log.info("Job stopped: {}", jobName)
          replyToWhenDone ! JobDone(jobName)
          Behaviors.same
      }
    }
  }
}
```

请注意`replyToWhenDone` 是如何包含在`watchWith`消息中，然后在接收`JobTerminated`消息时被使用的。

被关注的 Actor 可以是任意 `ActorRef`，它不必象上面的例子中必须是子actor。

应当注意，终止消息的生成与注册和终止发生的顺序无关。特别是，即使被关注的actor在注册时已经被终止，关注的actor也会收到终止消息。

多次注册不一定会导致生成多条消息，但不能保证只收到一条这样的消息：如果被关注的 actor 的终止消息生成并加入队列之后，直到它被处理之前，另一个注册到达，那么第二条消息也将被排队，因为对一个已经终止的actor的关注会导致立即生成终止的消息。

也可以使用 `context.unwatch(target)`来取消对其他actor存活状态的关注。即使终止的消息已经在邮箱中排队，这也有效；调用`unwatch`之后，该actor 的终止消息将不再被处理。

当被关注的 actor 被从[Cluster](cluster.md) 中删除不再做为一个节点存在时，也会发送终止消息。

----

[交互模式 ](interaction-patterns.md)

----------
**英文原文链接**：[Actor lifecycle](https://doc.akka.io/docs/akka/current/typed/actor-lifecycle.html).