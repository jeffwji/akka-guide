# Actor介绍

您正在查看 Actor Typed API 的文档，要查看 Akka Classic 文档，请参阅[[Classic Actors](https://github.com/guobinhit/akka-guide)。

## 模块信息

要使用 Akka Actors，请在您的项目中添加以下依赖项：

```scala
val AkkaVersion = "2.6.15"
    libraryDependencies ++= Seq(
      "com.typesafe.akka" %% "akka-actor-typed" % AkkaVersion,
      "com.typesafe.akka" %% "akka-actor-testkit-typed" % AkkaVersion % Test
)
```

Akka 模块的 Java 和 Scala DSL 都捆绑在同一个 JAR 中。为了获得流畅的开发体验，在使用 Eclipse 或 IntelliJ 等 IDE 时，您可以禁用导入器在 Scala 中工作时自动建议导入`javadsl`，反之亦然。请参阅[IDE 提示](https://doc.akka.io/docs/akka/current/additional/ide.html)。

| Project Info: Akka Actors (typed) |                                                              |
| --------------------------------- | ------------------------------------------------------------ |
| Artifact                          | com.typesafe.akka<br />akka-actor-typed<br />2.6.15<br />[Snapshots are available](https://doc.akka.io/docs/akka/current/typed/project/links.html#snapshots-repository) |
| JDK versions                      | Adopt OpenJDK 8Adopt OpenJDK 11                              |
| Scala versions                    | 2.12.14, 2.13.5                                              |
| JPMS module name                  | akka.actor.typed                                             |
| License                           | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0.html) |
| Readiness level                   | [Supported](https://developer.lightbend.com/docs/introduction/getting-help/support-terminology.html#supported), [Lightbend Subscription](https://www.lightbend.com/lightbend-subscription) provides support  Since 2.6.0, 2019-11-06 |
| Home page                         | https://akka.io/                                             |
| API documentation                 | [API (Scaladoc)](https://doc.akka.io/api/akka/2.6.15/akka/actor/typed/index.html) <br /> [API (Javadoc)](https://doc.akka.io/japi/akka/2.6.15/akka/actor/typed/package-summary.html) |
| Forums                            | [Lightbend Discuss](https://discuss.akka.io)  <br />[akka/akka Gitter channel](https://gitter.im/akka/akka) |
| Release notes                     | [akka.io blog](https://akka.io/blog/news-archive.html)       |
| Issues                            | [Github issues](https://github.com/akka/akka/issues)         |
| Sources                           | https://github.com/akka/akka                                 |

## Akka actor

该[Actor模型](https://en.wikipedia.org/wiki/Actor_model)提供了一个编写并发和分布式系统更高的水平的抽象。它使开发人员不必显式处理锁定和线程管理，从而更容易编写正确的并发和并行系统。Actors 是由 Carl Hewitt 在 1973 年的论文中定义的，但后来被 Erlang 语言普及，例如在 Ericsson 使用它来构建高度并发和可靠的电信系统取得了巨大成功。Akka 的 Actors 的 API 借鉴了 Erlang 的一些语法。

## 第一个例子

如果您是 Akka 的新手，您可能希望从阅读[入门指南开始](../getting-started-guide/introduction-to-akka.md)，然后回到这里了解更多信息。我们还建议您观看Akka  actor的简短[介绍视频](https://akka.io/blog/news/2019/12/03/akka-typed-actor-intro-video)。

熟悉 Actor 的基础、外部和内部生态系统很有帮助，了解您可以根据需要利用和自定义哪些内容，请参阅[Actor 系统](../general-concepts/actor-systems.md)和[Actor 引用、路径和地址](../eneral-concepts/addressing.md)。

正如[Actor Systems](../general-concepts/actor-systems.md)中所讨论的，Actor 是在独立的计算单元之间发送消息，但它看起来像什么？

假设你已经在以下所有代码中引入了：

```scala
import akka.actor.typed.scaladsl.Behaviors
import akka.actor.typed.scaladsl.LoggerOps
import akka.actor.typed.{ ActorRef, ActorSystem, Behavior }
```

有了这些，我们就可以定义我们的第一个 Actor，它会互相打招呼！

![你好世界1.png](https://doc.akka.io/docs/akka/current/typed/images/hello-world1.png)

```scala
object HelloWorld {
  final case class Greet(whom: String, replyTo: ActorRef[Greeted])
  final case class Greeted(whom: String, from: ActorRef[Greet])

  def apply(): Behavior[Greet] = Behaviors.receive { (context, message) =>
    context.log.info("Hello {}!", message.whom)
    message.replyTo ! Greeted(message.whom, context.self)
    Behaviors.same
  }
}
```

这段代码定义了两种消息类型，一种用于让 Actor 向某人打招呼，另一种是 Actor 将用来确认它已经这样做了。该`Greet`类型不仅包含要问候的人的信息，还包含代表提供消息的发送者的`ActorRef`的信息，以便`HelloWorld`Actor 可以发回确认消息。

Actor的行为在`receive`行为工厂的帮助下被定义为`Greeter`。处理下一条消息会导致新行为可能与当前行为不同。通过返回保持新的不可变状态的新行为来更新状态。在这个例子中，我们不需要更新任何状态，因此我们返回`same`，这意味着下一个行为“与当前行为相同”。

此行为处理的消息类型声明为 `Greet` 类，这意味着该消息的参数是此类型。这就是为什么我们可以访问 `whom`和`replyTo`成员而无需使用模式匹配。通常，一个actor 处理多个特定的消息类型，所有这些类型都直接或间接地扩展自一个公共的`trait`

在最后一行，我们看到`HelloWorld` Actor 向另一个 Actor 发送消息，这是使用`!`操作符（发音为“bang”或“tell”）完成的。它是一个异步操作，不会阻塞调用者的线程。

由于`replyTo`地址被声明为ActorRef[Greeted] 类型，编译器将只允许我们发送这种类型的消息，其他用法将导致编译错误。

Actor 接受的消息类型以及所有回复类型定义了所说的此 Actor 的协议；在这个例子中，它是一个简单的请求-回复协议，但 Actor 可以在需要时为任意复杂的协议建模。该协议与它的行为的绑定被很好地包装在一起——`HelloWorld` 对象中。

正如Carl Hewitt 所说，一个 Actor 不构成 Actor（系统）——没有人交谈可能会很孤独。我们需要另一个与`Greeter`交互的 actor。让我们创建一个`HelloWorldBot`接收`Greeter`的回复，并再次额外发送一些问候消息并收集回复，直到达到给定的最大消息数。

![你好-world2.png](https://doc.akka.io/docs/akka/current/typed/images/hello-world2.png)

```scala
object HelloWorldBot {

  def apply(max: Int): Behavior[HelloWorld.Greeted] = {
    bot(0, max)
  }

  private def bot(greetingCounter: Int, max: Int): Behavior[HelloWorld.Greeted] =
    Behaviors.receive { (context, message) =>
      val n = greetingCounter + 1
      context.log.info2("Greeting {} for {}", n, message.whom)
      if (n == max) {
        Behaviors.stopped
      } else {
        message.from ! HelloWorld.Greet(message.whom, context.self)
        bot(n, max)
      }
    }
}
```

请注意此 Actor 如何通过更改每个返回的`Greeted`的行为而不是使用任何变量来管理计数器。由于Actor 实例一次处理一条消息，因此不需要使用，例如`synchronized`或`AtomicInteger`，来做并发保护。

第三个Actor产生`Greeter`和`HelloWorldBot`并开始它们之间的交互。

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

现在我们想试用这个 Actor，所以我们必须启动一个 ActorSystem 来托管它：

```scala
val system: ActorSystem[HelloWorldMain.SayHello] =
  ActorSystem(HelloWorldMain(), "hello")

system ! HelloWorldMain.SayHello("World")
system ! HelloWorldMain.SayHello("Akka")
```



我们从定义的`HelloWorldMain`的行为开始启动一个 Actor 系统，并发送两条`SayHello`消息，这将启动两个单独的`HelloWorldBot` actor 和单个的`Greeter` Actor 之间的交互。

一个应用程序的每个 JVM 通常包括一个单独的`ActorSystem 并运行多个 actor。

控制台输出可能如下所示：

```
[INFO] [03/13/2018 15:50:05.814] [hello-akka.actor.default-dispatcher-4] [akka://hello/user/greeter] Hello World!
[INFO] [03/13/2018 15:50:05.815] [hello-akka.actor.default-dispatcher-4] [akka://hello/user/greeter] Hello Akka!
[INFO] [03/13/2018 15:50:05.815] [hello-akka.actor.default-dispatcher-2] [akka://hello/user/World] Greeting 1 for World
[INFO] [03/13/2018 15:50:05.815] [hello-akka.actor.default-dispatcher-4] [akka://hello/user/Akka] Greeting 1 for Akka
[INFO] [03/13/2018 15:50:05.815] [hello-akka.actor.default-dispatcher-5] [akka://hello/user/greeter] Hello World!
[INFO] [03/13/2018 15:50:05.815] [hello-akka.actor.default-dispatcher-5] [akka://hello/user/greeter] Hello Akka!
[INFO] [03/13/2018 15:50:05.815] [hello-akka.actor.default-dispatcher-4] [akka://hello/user/World] Greeting 2 for World
[INFO] [03/13/2018 15:50:05.815] [hello-akka.actor.default-dispatcher-5] [akka://hello/user/greeter] Hello World!
[INFO] [03/13/2018 15:50:05.815] [hello-akka.actor.default-dispatcher-4] [akka://hello/user/Akka] Greeting 2 for Akka
[INFO] [03/13/2018 15:50:05.816] [hello-akka.actor.default-dispatcher-5] [akka://hello/user/greeter] Hello Akka!
[INFO] [03/13/2018 15:50:05.816] [hello-akka.actor.default-dispatcher-4] [akka://hello/user/World] Greeting 3 for World
[INFO] [03/13/2018 15:50:05.816] [hello-akka.actor.default-dispatcher-6] [akka://hello/user/Akka] Greeting 3 for Akka
```

您还需要添加[日志依赖项](logging.md)以在运行时查看该输出。

#### 这是完整的示例：

```scala
import akka.actor.typed.scaladsl.Behaviors
import akka.actor.typed.scaladsl.LoggerOps
import akka.actor.typed.{ ActorRef, ActorSystem, Behavior }

object HelloWorld {
  final case class Greet(whom: String, replyTo: ActorRef[Greeted])
  final case class Greeted(whom: String, from: ActorRef[Greet])

  def apply(): Behavior[Greet] = Behaviors.receive { (context, message) =>
    println(s"Hello ${message.whom}!")
    message.replyTo ! Greeted(message.whom, context.self)
    Behaviors.same
  }
}

object HelloWorldBot {

  def apply(max: Int): Behavior[HelloWorld.Greeted] = {
    bot(0, max)
  }

  private def bot(greetingCounter: Int, max: Int): Behavior[HelloWorld.Greeted] =
    Behaviors.receive { (context, message) =>
      val n = greetingCounter + 1
      println(s"Greeting $n for ${message.whom}")
      if (n == max) {
        Behaviors.stopped
      } else {
        message.from ! HelloWorld.Greet(message.whom, context.self)
        bot(n, max)
      }
    }
}

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

  def main(args: Array[String]): Unit = {
    val system: ActorSystem[HelloWorldMain.SayHello] =
      ActorSystem(HelloWorldMain(), "hello")

    system ! HelloWorldMain.SayHello("World")
    system ! HelloWorldMain.SayHello("Akka")
  }
}

// This is run by ScalaFiddle
HelloWorldMain.main(Array.empty)
```

## 一个更复杂的例子

下一个示例更加现实，并且它演示了一些重要的范式：

- 使用 `sealed trait` 和`case class`/`object` 来表达Actor 如何接收多个消息。
- 使用子 actor 处理会话
- 通过改变行为来处理状态
- 以类型安全的方式使用多个 Actor 来表达协议中不同的部分。

![聊天室.png](https://doc.akka.io/docs/akka/current/typed/images/chat-room.png)

### 函数式风格

首先我们将以函数式风格展示这个例子，然后以[面向对象的风格](https://doc-akka-io.translate.goog/docs/akka/current/typed/actors.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#object-oriented-style)展示同样的例子。您选择使用哪种风格取决于品味，两种风格可以混合使用，具体取决于哪种风格最适合特定Actor。[样式指南中](style-guide.md#函数式与面向对象的风格)提供了做出选择所需的注意事项。

考虑运行一个聊天室的 Actor：客户端 Actor 可以通过发送包含其用户名称的消息进行连接，然后他们可以发布消息。聊天室 Actor 会将所有发布的消息传播给所有当前连接的客户端 Actor。协议定义可能如下所示：

```scala
object ChatRoom {
  sealed trait RoomCommand
  final case class GetSession(screenName: String, replyTo: ActorRef[SessionEvent]) extends RoomCommand

  sealed trait SessionEvent
  final case class SessionGranted(handle: ActorRef[PostMessage]) extends SessionEvent
  final case class SessionDenied(reason: String) extends SessionEvent
  final case class MessagePosted(screenName: String, message: String) extends SessionEvent

  sealed trait SessionCommand
  final case class PostMessage(message: String) extends SessionCommand
  private final case class NotifyClient(message: MessagePosted) extends SessionCommand
}
```

最初，客户端 Actor 只能访问`ActorRef[GetSession]`以允许他们迈出第一步的 。一旦客户端建立了会话，它就会收到一条`SessionGranted`消息，其中包含解锁下一个协议步骤的消息，即发布消息。`PostMessage` 命令将被发送到该聊天室中该会话所指向的特定地址。会话的另一作用是客户端通过参数`replyTo`传达了自己的地址，以便可以接收发送给它的后续事件。

该流程展示了 Actor 可以表达比仅仅做为 Java 对象方法调用等价物更多的方面。声明的消息类型及其内容描述了一个完整的协议，该协议可以涉及多个 Actor 并且可以在多个步骤中传递。下面是聊天室协议的实现：

```scala
object ChatRoom {
  private final case class PublishSessionMessage(screenName: String, message: String) extends RoomCommand

  def apply(): Behavior[RoomCommand] =
    chatRoom(List.empty)

  private def chatRoom(sessions: List[ActorRef[SessionCommand]]): Behavior[RoomCommand] =
    Behaviors.receive { (context, message) =>
      message match {
        case GetSession(screenName, client) =>
          // create a child actor for further interaction with the client
          val ses = context.spawn(
            session(context.self, screenName, client),
            name = URLEncoder.encode(screenName, StandardCharsets.UTF_8.name))
          client ! SessionGranted(ses)
          chatRoom(ses :: sessions)
        case PublishSessionMessage(screenName, message) =>
          val notification = NotifyClient(MessagePosted(screenName, message))
          sessions.foreach(_ ! notification)
          Behaviors.same
      }
    }

  private def session(
      room: ActorRef[PublishSessionMessage],
      screenName: String,
      client: ActorRef[SessionEvent]): Behavior[SessionCommand] =
    Behaviors.receiveMessage {
      case PostMessage(message) =>
        // from client, publish to others via the room
        room ! PublishSessionMessage(screenName, message)
        Behaviors.same
      case NotifyClient(message) =>
        // published from the room
        client ! message
        Behaviors.same
    }
}
```

状态是通过改变行为而不是使用任何变量来管理的。

当一个新`GetSession`命令进来时，我们将该客户端添加到返回行为中的列表中。然后我们还需要创建`ActorRef`将用于发布消息的会话。在本例中，我们希望创建一个非常简单的 Actor，将`PostMessage`命令重新打包为一个`PublishSessionMessage`还包含屏幕名称的命令。

我们已经解释过了在这里声明的 `Behavior` 可以处理子类型`RoomCommand`.`GetSession`和会话 Actor的`PublishSessionMessage`命令，该命令将其包含的聊天室消息传播到所有连接的客户端。但是我们不想赋予任意客户端发送`PublishSessionMessage`命令的能力，我们将此权力保留在所创建的内部会话 actor 中——否则客户端可能会伪装成完全不同的名称（想象一下`GetSession` 协议将包含身份验证信息以进一步确保安全）。因此`PublishSessionMessage`具有`private`可见性，不能在`ChatRoom` object 之外创建。

如果我们不关心会话安全和用户名称之间的对应关系的话，那么我们可以更改协议，将`PostMessage`删除并且所有客户端都可以通过`ActorRef[PublishSessionMessage]`发送消息。在这种情况下，不需要会话Actor，我们可以在这种情况下使用`context.self` ，类型检查将会认为有效，因为`ActorRef[-T]`的类型参数是逆变的，这意味着我们可以在任何需要`ActorRef[PublishSessionMessage]`的地方使用`ActorRef[RoomCommand]`——这是合理的，因为后者比后者包含更宽泛的表达能力。相反则会出现问题，也就是说传递一个`ActorRef[PublishSessionMessage]`给需要`ActorRef[RoomCommand]` 的地方将导致类型错误。

**试运行**

为了查看这个聊天室的运行情况，我们需要编写一个客户端 Actor来使用它的：

```scala
object Gabbler {
  import ChatRoom._

  def apply(): Behavior[SessionEvent] =
    Behaviors.setup { context =>
      Behaviors.receiveMessage {
        case SessionGranted(handle) =>
          handle ! PostMessage("Hello World!")
          Behaviors.same
        case MessagePosted(screenName, message) =>
          context.log.info2("message has been posted by '{}': {}", screenName, message)
          Behaviors.stopped
      }
    }
}
```

根据这个Behavior，我们创建了一个 Actor，它接受一个聊天室会话，然后发布一条消息并等待它发布完成，然后终止。最后需要有改变Behavior的能力，我们需要从正常的运行Behavior过渡到终止Behavior。这就是为什么这里我们不返回`same`，如上所述，而是另一个特殊值`stopped`。

由于`SessionEvent`是一个密封的` trait`，因此如果我们忘记处理其中一个子类型，Scala 编译器将会警告我们；在这个例子中，它提醒我们除了`SessionGranted`我们还可能会收到一个`SessionDenied`事件。

现在要运行的话，我们必须同时启动聊天室和 gabbler，当然我们需要在 Actor 系统中执行此操作。由于只能有一个 `user` 监护人，我们可以从 gabbler 启动聊天室（但是我们不想这么做——因为这会使逻辑复杂化）或从聊天室启动 gabbler（这也是荒谬的），或者——唯一明智的选择——从一个第三方 Actor 开始：

```scala
object Main {
  def apply(): Behavior[NotUsed] =
    Behaviors.setup { context =>
      val chatRoom = context.spawn(ChatRoom(), "chatroom")
      val gabblerRef = context.spawn(Gabbler(), "gabbler")
      context.watch(gabblerRef)
      chatRoom ! ChatRoom.GetSession("ol’ Gabbler", gabblerRef)

      Behaviors.receiveSignal {
        case (_, Terminated(_)) =>
          Behaviors.stopped
      }
    }

  def main(args: Array[String]): Unit = {
    ActorSystem(Main(), "ChatRoomDemo")
  }

}
```

传统上，我们将这个Actor 称为 `Main`，它直接对应于传统 Java 应用程序中的`main`方法。这个 Actor 会自己执行它的工作，我们不需要从外部给它发送消息，所以我们声明它的类型是`NotUsed`。 Actors 不仅接收外部消息，还收到某些系统事件的通知，即所谓的信号是那些那些我们使用 `receive` 行为装饰器来实现的那些特定的消息。信号（`Signal`的子类型）或用户消息触发`onMessage`函数时，`onSignal`函数将被调用。

这个特定的 `Main` Actor 我们使用 `Behaviors.setup`来创建，它就像一个行为的工厂。行为实例的创建被推迟到actor 启动之后，而`Behaviors.receive`则相反，它在 actor 运行前立即创建行为实例。工厂函数`setup`将`ActorContext`作为参数传入，它可以例如用于生成子 actor。这个`Main` Actor 创建了聊天室和 gabbler 并启动了它们之间的会话，当 gabbler 终止时，我们将收到`Terminated`事件，因为我们用`context.watch`观察了它，这允许我们安全地关闭 Actor 系统：当`Main`Actor 终止时就没有什么遗留的事情需要做了。

因此当 Actor 系统和`Main` Actor 的`Behavior`一起被创建后，我们可以让`main`方法直接返回，`ActorSystem`将继续保持运行，而JVM将持续存活，直到根 actor终止。

### 面向对象的风格

上面的示例使用了函数式编程风格，您将一个函数传递给一个工厂，然后该工厂构造一个行为，对于有状态的 Actor，这意味着将不可变状态作为参数传递，并在需要对更改的状态执行操作时切换到新行为。另一种可选的实现方式是更加面向对象的风格的，这种风格采用具体类来定义 Actor，并将可变状态作为字段保存在其中。

您选择使用哪种风格取决于您的品味，两种风格可以混合使用，具体取决于哪种风格最适合特定Actor。[样式指南中](style-guide.md#函数式与面向对象式)提供了选择的注意事项。

#### AbstractBehavior API

定义一个基于类的Actor西药扩展自` akka.actor.typed.scaladsl.AbstractBehavior[T]`，其中 `T` 是能够接收的消息类型。

让我们用 `AbstractBehavior`重复上面[一个更复杂的例子中](#一个更复杂的例子)的聊天室示例。与actor交互的协议看起来将会是一样的：

```scala
object ChatRoom {
  sealed trait RoomCommand
  final case class GetSession(screenName: String, replyTo: ActorRef[SessionEvent]) extends RoomCommand

  sealed trait SessionEvent
  final case class SessionGranted(handle: ActorRef[PostMessage]) extends SessionEvent
  final case class SessionDenied(reason: String) extends SessionEvent
  final case class MessagePosted(screenName: String, message: String) extends SessionEvent

  sealed trait SessionCommand
  final case class PostMessage(message: String) extends SessionCommand
  private final case class NotifyClient(message: MessagePosted) extends SessionCommand
}
```

最初，客户端 Actor 只接受一个 `ActorRef[GetSession]`以允许他们迈出第一步的 。一旦一个客户端会话被建立，它就会收到一条`SessionGranted`消息，其中包含解锁下一个协议步骤的消息，即发布消息。`PostMessage` 命令将被发送到该聊天室中该会话所指向的特定地址。会话的另一作用是客户端通过参数`replyTo`传达了自己的地址，以便可以接收发送给它的后续事件。

该流程展示了 Actor 可以表达比仅仅做为 Java 对象方法调用等价物更多的方面。声明的消息类型及其内容描述了一个完整的协议，该协议可以涉及多个 Actor 并且可以在多个步骤中传递。下面是基于`AbstractBehavior`的聊天室协议的实现：

```scala
object ChatRoom {
  private final case class PublishSessionMessage(screenName: String, message: String) extends RoomCommand

  def apply(): Behavior[RoomCommand] =
    Behaviors.setup(context => new ChatRoomBehavior(context))

  class ChatRoomBehavior(context: ActorContext[RoomCommand]) extends AbstractBehavior[RoomCommand](context) {
    private var sessions: List[ActorRef[SessionCommand]] = List.empty

    override def onMessage(message: RoomCommand): Behavior[RoomCommand] = {
      message match {
        case GetSession(screenName, client) =>
          // create a child actor for further interaction with the client
          val ses = context.spawn(
            SessionBehavior(context.self, screenName, client),
            name = URLEncoder.encode(screenName, StandardCharsets.UTF_8.name))
          client ! SessionGranted(ses)
          sessions = ses :: sessions
          this
        case PublishSessionMessage(screenName, message) =>
          val notification = NotifyClient(MessagePosted(screenName, message))
          sessions.foreach(_ ! notification)
          this
      }
    }
  }

  object SessionBehavior {
    def apply(
        room: ActorRef[PublishSessionMessage],
        screenName: String,
        client: ActorRef[SessionEvent]): Behavior[SessionCommand] =
      Behaviors.setup(ctx => new SessionBehavior(ctx, room, screenName, client))
  }

  private class SessionBehavior(
      context: ActorContext[SessionCommand],
      room: ActorRef[PublishSessionMessage],
      screenName: String,
      client: ActorRef[SessionEvent])
      extends AbstractBehavior[SessionCommand](context) {

    override def onMessage(msg: SessionCommand): Behavior[SessionCommand] =
      msg match {
        case PostMessage(message) =>
          // from client, publish to others via the room
          room ! PublishSessionMessage(screenName, message)
          Behaviors.same
        case NotifyClient(message) =>
          // published from the room
          client ! message
          Behaviors.same
      }
  }
}
```

状态通过类中的字段进行管理，就像使用常规的面向对象类一样。由于状态是可变的，我们从不返回与消息逻辑不同的行为，但可以返回`AbstractBehavior`实例本身 ( `this`) 作为用于处理传入的下一条消息的行为。我们也可以返回`Behavior.same`以实现相同的目的。

还可以返回一个新的不同的`AbstractBehavior`，例如在有限状态机（FSM）中表示不同的状态，或者使用某个函数式行为工厂将面向对象与函数式风格结合起来，来实现同一个 Actor 的生命周期中的不同部分。

当一个新`GetSession` 命令被接收到时，我们将该客户端添加到当前会话列表中。然后我们还需要创建会话的`ActorRef`用于发布消息。在本例中，我们创建一个非常简单的 Actor，用于将`PostMessage`命令和用户名重新打包到一个`PublishSessionMessage`命令中。

要实现为会话生成孩子的逻辑，我们们需要访问 ActorContext。这是在创建行为时作为构造函数参数注入的，请注意我们如何在`apply`工厂方法中结合使用`Behaviors.setup`和`AbstractBehavior`来执行此操作的。

我们在这里所声明的Behavior可以处理已经解释过的`RoomCommand`.`GetSession`，和来自会话 Actor的`PublishSessionMessage`命令，该命令将其包含的聊天室消息传播到所有连接的客户端。但是我们不想赋予任意客户端发送`PublishSessionMessage`命令的能力，我们将此权力保留在所创建的内部会话 actor 中——否则客户端可能会伪装成完全不同的名称（想象一下`GetSession` 协议将包含身份验证信息以进一步确保安全）。因此`PublishSessionMessage`具有`private`可见性，不能在`ChatRoom` object 之外创建。

如果我们不关心会话安全和用户名称之间的对应关系的话，那么我们可以更改协议，将`PostMessage`删除并且所有客户端都可以通过`ActorRef[PublishSessionMessage]`发送消息。在这种情况下，不需要会话Actor，我们可以在这种情况下使用`context.self` ，类型检查将会认为有效，因为`ActorRef[-T]`的类型参数是逆变的，这意味着我们可以在任何需要`ActorRef[PublishSessionMessage]`的地方使用`ActorRef[RoomCommand]`——这是合理的，因为后者比后者包含更宽泛的表达能力。相反则会出现问题，也就是说传递一个`ActorRef[PublishSessionMessage]`给需要`ActorRef[RoomCommand]` 的地方将导致类型错误。

#### **试运行**

为了查看这个聊天室的运行情况，我们需要编写一个可以使用它的客户端 Actor ，对于这个无状态的 Actor，使用`AbstractBehavior` 没有多大意义，所以让我们重用上面示例中的函数式风格 gabbler：

```scala
object Gabbler {
  import ChatRoom._

  def apply(): Behavior[SessionEvent] =
    Behaviors.setup { context =>
      Behaviors.receiveMessage {
        case SessionDenied(reason) =>
          context.log.info("cannot start chat room session: {}", reason)
          Behaviors.stopped
        case SessionGranted(handle) =>
          handle ! PostMessage("Hello World!")
          Behaviors.same
        case MessagePosted(screenName, message) =>
          context.log.info2("message has been posted by '{}': {}", screenName, message)
          Behaviors.stopped
      }
    }
```

现在要运行的话，我们必须同时启动聊天室和 gabbler，当然我们需要在 Actor 系统中执行此操作。由于只能有一个 `user` 监护人，我们可以从 gabbler 启动聊天室（但是我们不想这么做——因为这会使逻辑复杂化）或从聊天室启动 gabbler（这也是荒谬的），或者——唯一明智的选择——从一个第三方 Actor 开始：

```scala
object Main {
  def apply(): Behavior[NotUsed] =
    Behaviors.setup { context =>
      val chatRoom = context.spawn(ChatRoom(), "chatroom")
      val gabblerRef = context.spawn(Gabbler(), "gabbler")
      context.watch(gabblerRef)
      chatRoom ! ChatRoom.GetSession("ol’ Gabbler", gabblerRef)

      Behaviors.receiveSignal {
        case (_, Terminated(_)) =>
          Behaviors.stopped
      }
    }

  def main(args: Array[String]): Unit = {
    ActorSystem(Main(), "ChatRoomDemo")
  }

}
```

传统上，我们将这个Actor 称为 `Main`，它直接对应于传统 Java 应用程序中的`main`方法。这个 Actor 会自己执行它的工作，我们不需要从外部给它发送消息，所以我们声明它的类型是`NotUsed`。 Actors 不仅接收外部消息，还收到某些系统事件的通知，即所谓的信号是那些那些我们使用 `receive` 行为装饰器来实现的那些特定的消息。信号（`Signal`的子类型）或用户消息触发`onMessage`函数时，`onSignal`函数将被调用。

这个特定的 `Main` Actor 我们使用 `Behaviors.setup`来创建，它就像一个行为的工厂。行为实例的创建被推迟到actor 启动之后，而`Behaviors.receive`则相反，它在 actor 运行前立即创建行为实例。工厂函数`setup`将`ActorContext`作为参数传入，它可以例如用于生成子 actor。这个`Main` Actor 创建了聊天室和 gabbler 并启动了它们之间的会话，当 gabbler 终止时，我们将收到`Terminated`事件，因为我们用`context.watch`观察了它，这允许我们安全地关闭 Actor 系统：当`Main`Actor 终止时就没有什么遗留的事情需要做了。

因此当 Actor 系统和`Main` Actor 的`Behavior`一起被创建后，我们可以让`main`方法直接返回，`ActorSystem`将继续保持运行，而JVM将持续存活，直到根 actor终止。

----

[Actor 生命周期](actor-lifecycle.md)

----

[原文链接](https://doc.akka.io/docs/akka/current/typed/actors.html)