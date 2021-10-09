# 交互模式

您正在查看新 actor API 的文档，要查看经典 Akka 文档，请参阅[Classic Actors](https://doc-akka-io.translate.goog/docs/akka/current/actors.html)。

## 依赖

要使用 Akka Actor Typed，您必须在项目中添加以下依赖项：

```scala
val AkkaVersion = "2.6.15"
libraryDependencies += "com.typesafe.akka" %% "akka-actor-typed" % AkkaVersion
```

## 介绍

与一个 Akka actor 的交互是通过 `ActorRef[T]`来进行的，其中`T`是该 Actor 能接受的消息的类型，它也被称为“协议”。这确保只能将正确类型的消息发送给 Actor，并且除了 Actor 本身之外没有其他人可以访问 Actor 实例内部。

与 Actor 的消息交换遵循一些常见模式，让我们逐一介绍。 

## 发出即遗忘

与 actor 交互的基本方式是通过“tell”，它非常常见，以至于它有一个特殊的符号方法名称：`actorRef ! message` . 可以在任何线程中安全地使用 tell 发送消息。

Tell 是异步的，这意味着该方法会立即返回。该语句返回后，无法保证收件人已经处理了该消息。这也意味着无法知道消息是否已收到、处理是成功了还是失败了。

**例子：**

![忘记密码.png](https://doc.akka.io/docs/akka/current/typed/images/fire-forget.png)

以上 Actor 的行为和协议：

```scala
object Printer {

  case class PrintMe(message: String)

  def apply(): Behavior[PrintMe] =
    Behaviors.receive {
      case (context, PrintMe(message)) =>
        context.log.info(message)
        Behaviors.same
    }
}
```

发出即遗忘模式看起来是这样的：

```scala
val system = ActorSystem(Printer(), "fire-and-forget-sample")

// note how the system is also the top level actor ref
val printer: ActorRef[Printer.PrintMe] = system

// these are all fire and forget
printer ! Printer.PrintMe("message 1")
printer ! Printer.PrintMe("not message 2")
```

**它被使用在以下情况中：**

- 消息已被处理的确保不重要
- 没有办法对不成功的交付或处理采取行动
- 我们希望被创建的消息数量最少以获得更高的吞吐量（发送响应需要创建两倍数量的消息）

**问题：**

- 如果消息的流入量高于actor的处理能力，则收件箱将被填满，并且在最坏的情况下会导致 JVM `OutOfMemoryError` 崩溃。
- 如果消息丢失，发件人将不知道

## 请求-响应

Actor 之间的许多交互需要从接收者发回一个或多个响应消息。响应消息可以是查询的结果、某种形式的对消息已被接收和处理的确认或请求者所订阅的事件。

在 Akka 中，响应的接收者必须被编码为消息本身中的一个字段，然后接收者可以使用它来发送（告诉）响应。

**例子：**

![请求响应.png](https://doc.akka.io/docs/akka/current/typed/images/request-response.png)

请使用以下协议：

```scala
case class Request(query: String, replyTo: ActorRef[Response])
case class Response(result: String)
```

发送者将使用其自己的`ActorRef[Response]`，它可以通过访问`ActorContext.self`到，然后赋给`replyTo`

```scala
cookieFabric ! CookieFabric.Request("give me cookies", context.self)
```

在接收端，`ActorRef[Response]`可被用于发送一个或多个返回响应：

```scala
def apply(): Behaviors.Receive[Request] =
  Behaviors.receiveMessage[Request] {
    case Request(query, replyTo) =>
      // ... process query ...
      replyTo ! Response(s"Here are the cookies for [$query]!")
      Behaviors.same
  }
```

**该模式被使用在以下情况：**

- 向会发回一个或多个响应消息的 actor 执行订阅。

**问题：**

- Actor 很少将来自另一个Actor 的响应消息作为其协议的一部分（请参阅[适应性响应](#适应性响应)）
- 很难检测到消息请求未被传递或处理（请参阅[ask](#通过ask在两个Actor之间执行请求响应)）
- 除非协议已经包含提供上下文的方法，例如也在响应中包含请求 ID，否则不可能不通过新建一个单独的 actor 来将交互绑定到某个特定的上下文（请参阅[ask](#通过ask在两个Actor之间执行请求响应)或[为每个 session 生成 子actor](#为每个session生成子actor)）

## 适应性响应

大多数情况下，发送Actor不支持也不应该支持接收另一个Actor的响应消息。在这种情况下，我们需要提供一个具有发送者能够正确处理的响应消息的类型的`ActorRef`。

**例子：**

![自适应响应.png](https://doc.akka.io/docs/akka/current/typed/images/adapted-response.png)

```scala
object Backend {
  sealed trait Request
  final case class StartTranslationJob(taskId: Int, site: URI, replyTo: ActorRef[Response]) extends Request

  sealed trait Response
  final case class JobStarted(taskId: Int) extends Response
  final case class JobProgress(taskId: Int, progress: Double) extends Response
  final case class JobCompleted(taskId: Int, result: URI) extends Response
}

object Frontend {

  sealed trait Command
  final case class Translate(site: URI, replyTo: ActorRef[URI]) extends Command
  private final case class WrappedBackendResponse(response: Backend.Response) extends Command

  def apply(backend: ActorRef[Backend.Request]): Behavior[Command] =
    Behaviors.setup[Command] { context =>
      val backendResponseMapper: ActorRef[Backend.Response] =
        context.messageAdapter(rsp => WrappedBackendResponse(rsp))

      def active(inProgress: Map[Int, ActorRef[URI]], count: Int): Behavior[Command] = {
        Behaviors.receiveMessage[Command] {
          case Translate(site, replyTo) =>
            val taskId = count + 1
            backend ! Backend.StartTranslationJob(taskId, site, backendResponseMapper)
            active(inProgress.updated(taskId, replyTo), taskId)

          case wrapped: WrappedBackendResponse =>
            wrapped.response match {
              case Backend.JobStarted(taskId) =>
                context.log.info("Started {}", taskId)
                Behaviors.same
              case Backend.JobProgress(taskId, progress) =>
                context.log.info2("Progress {}: {}", taskId, progress)
                Behaviors.same
              case Backend.JobCompleted(taskId, result) =>
                context.log.info2("Completed {}: {}", taskId, result)
                inProgress(taskId) ! result
                active(inProgress - taskId, count)
            }
        }
      }

      active(inProgress = Map.empty, count = 0)
    }
}
```

您可以为不同的消息类别注册多个消息适配器。每个消息类只能有一个消息适配器，以确保如果重复注册适配器的数量不会无限增长。同时注册的适配器将替换同一消息类中的现有的适配器。

如果消息类与给定类匹配或者是其子类，则将使用消息适配器。注册的适配器按其注册顺序的相反顺序进行尝试，即最后注册的先注册。

消息适配器（和返回的`ActorRef`）与接收参与者具有相同的生命周期。建议在顶级`Behaviors.setup`或构造函数中注册适配器，`AbstractBehavior`但如果需要，可以稍后注册它们。

适配器函数在接收actor中运行并且可以安全地访问它的状态，但是如果它抛出异常，actor就会停止。

**在以下情况下有用：**

- 在不同的actor消息协议之间进行转换
- 订阅一个会发回许多响应消息的参与者

**问题：**

- 很难检测到消息请求未被传递或处理（请参阅[ask](https://doc-akka-io.translate.goog/docs/akka/current/typed/interaction-patterns.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#request-response-with-ask-between-two-actors)）
- 每种响应消息类型只能进行一种适配，如果注册了新的，则替换旧的，例如，如果不同的目标参与者使用相同的响应类型，则它们不能有不同的适配，除非在消息中编码了某种相关性
- 除非协议已经包含提供上下文的方法，例如也在响应中发送的请求 id，否则不可能在不引入新的、单独的参与者的情况下将交互绑定到某些特定的上下文

## 请求-响应在两个参与者之间询问

在交互其中存在1：请求，我们可以使用响应之间1映射`ask`在`ActorContext`与另一个参与者进行交互。

交互有两个步骤，首先我们需要构造传出消息，为此我们需要将一个作为接收者放入传出消息中。第二步是将成功或失败转换为作为发送参与者协议一部分的消息。另请参阅[通用响应包装器](https://doc-akka-io.translate.goog/docs/akka/current/typed/interaction-patterns.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#generic-response-wrapper)以获取成功或错误的回复。`ActorRef[Response]``Response`

**例子：**

![问演员.png](https://doc.akka.io/docs/akka/current/typed/images/ask-from-actor.png)

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab1)

响应适配函数在接收actor中运行并且可以安全地访问它的状态，但是如果它抛出异常，actor就会停止。

**在以下情况下有用：**

- 单响应查询
- 在继续之前，参与者需要知道消息已被处理
- 如果没有及时响应，允许参与者重新发送
- 跟踪未完成的请求，而不是用消息压倒收件人（“背压”）
- 上下文应该附加到交互，但协议不支持（请求 id，响应的查询是什么）

**问题：**

- 只能有一个响应`ask`（请参阅[每个会话子 Actor](https://doc-akka-io.translate.goog/docs/akka/current/typed/interaction-patterns.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#per-session-child-actor)）
- 当`ask`超时时，接收actor不知道并且可能仍然处理它直到完成，甚至在事后开始处理它
- 为超时找到一个合适的值，尤其是在接收 actor 中`ask`触发 chained `ask`s 时。你想要一个短暂的超时来响应并回复请求者，但同时你不希望有很多误报



## 来自 Actor 外部的请求-响应

有时您需要与来自actor系统外部的actor进行交互，这可以通过如上所述的`ask`即发即忘来完成，或者通过另一个版本返回a ，该版本要么成功响应，要么失败，如果有在指定的超时时间内没有响应。`Future[Response]``TimeoutException`

为此，我们使用`ask`（或符号`?`）隐式添加到`ActorRef`by`akka.actor.typed.scaladsl.AskPattern._`向演员发送消息并获得`Future[Response]`回复。`ask`采用隐式`Timeout`和`ActorSystem`参数。 

**例子：**

![问从外面.png](https://doc.akka.io/docs/akka/current/typed/images/ask-from-outside.png)

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab1)

请注意，消息协议中的验证错误也是明确的。该`GiveMeCookies`请求可以与回复`Cookies`或`InvalidRequest`。请求者必须决定如何处理`InvalidRequest`回复。有时它应该被视为失败，因此可以在请求者端映射回复。另请参阅[通用响应包装器](https://doc-akka-io.translate.goog/docs/akka/current/typed/interaction-patterns.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#generic-response-wrapper)以获取成功或错误的回复。`Future`

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab1)

**在以下情况下有用：**

- 从演员系统外部查询演员

**问题：**

- 返回的回调很容易意外关闭和不安全的可变状态，因为它们将在不同的线程上执行`Future`
- 只能有一个响应`ask`（请参阅[每个会话子 Actor](https://doc-akka-io.translate.goog/docs/akka/current/typed/interaction-patterns.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#per-session-child-actor)）
- 当`ask`超时时，接收actor不知道并且可能仍然处理它直到完成，甚至在事后开始处理它

## 通用响应包装器

在许多情况下，响应可以是成功的结果，也可以是错误（例如，命令无效的验证错误）。必须为每个请求类型定义两个响应类和一个共享超类型可能是重复的，尤其是在集群上下文中，您还必须确保消息可以序列化以通过网络发送。

为了帮助解决这个问题，Akka: 中包含了一个通用的状态响应类型，在任何可以使用的地方，还有第二种方法，假设响应是一个将打开成功响应并帮助处理验证错误的方法。Akka 为该类型包含预构建的序列化程序，因此在正常用例中，集群应用程序只需要为成功的结果提供序列化程序。[`StatusReply`](https://doc-akka-io.translate.goog/api/akka/2.6/akka/pattern/StatusReply.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)`ask``askWithStatus``StatusReply`

对于成功回复不包含实际值但更多是确认的情况，有一个预定义的 type 。`StatusReply.Ack``StatusReply[Done]`

错误最好作为描述错误的文本发送，但也可以使用异常来附加类型。

**示例演员对演员提问：**

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab1)

验证错误变成`Failure`了消息适配器的错误。在这种情况下，我们明确地将验证错误与其他询问失败分开处理。

**示例从外部询问：**

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab1)

请注意，消息协议中的验证错误也是显式的，但编码为包装器类型，使用以下方式构造：`StatusReply.Error(text)`

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab1)

## 忽略回复

在某些情况下，参与者对特定请求消息有响应，但您对响应不感兴趣。在这种情况下，您可以通过将请求响应转换为即发即忘。`system.ignoreRef`

`system.ignoreRef`，顾名思义，返回`ActorRef`忽略发送给它的任何消息的 。

使用与上述[请求响应](https://doc-akka-io.translate.goog/docs/akka/current/typed/interaction-patterns.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#request-response)相同的协议，如果发送方更愿意忽略它可以传递给 的回复，它可以通过访问。`system.ignoreRef``replyTo``ActorContext.system.ignoreRef`

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab1)

**在以下情况下有用：**

- 发送协议定义了回复的消息，但您对获得回复不感兴趣

**问题：**

返回的`ActorRef`消息会忽略发送给它的所有消息，因此应谨慎使用。

- 不经意间把它当作正常情况`ActorRef`来传递可能会导致演员与演员之间的互动中断。
- 在`ask`从 Actor System 外部执行 an 时使用它会导致由to返回的超时，因为它永远不会完成。`Future``ask`
- 最后，它是合法的`watch`，但由于它是一种特殊类型，它永远不会终止，因此您永远不会收到`Terminated`来自它的信号。

## 将 Future 结果发送给自己

使用从参与者返回 a 的 API 时，您通常希望在完成时使用参与者中的响应值。为此目的，提供了一种方法。`Future``Future``ActorContext``pipeToSelf`

**例子：**

![管道到self.png](https://doc.akka.io/docs/akka/current/typed/images/pipe-to-self.png)

演员`CustomerRepository`正在调用`CustomerDataAccess`返回 的方法。`Future`

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab1)

It could be tempting to just use `onComplete on the Future`, but that introduces the risk of accessing internal state of the actor  that is not thread-safe from an external thread. For example, the `numberOfPendingOperations` counter in above example can’t be accessed from such callback.  Therefore it is better to map the result to a message and perform  further processing when receiving that message.

**Useful when:**

- Accessing APIs that are returning `Future` from an actor, such as a database or an external service
- The actor needs to continue processing when the `Future` has completed
- Keep context from the original request and use that when the `Future` has completed, for example an `replyTo` actor reference

**Problems:**

- Boilerplate of adding wrapper messages for the results

## Per session child Actor

In some cases a complete response to a request can only be  created and sent back after collecting multiple answers from other  actors. For these kinds of interaction it can be good to delegate the  work to a per “session” child actor. The child could also contain  arbitrary logic to implement retrying, failing on timeout, tail  chopping, progress inspection etc.

Note that this is essentially how `ask` is implemented, if all you need is a single response with a timeout it is better to use `ask`.

The child is created with the context it needs to do the work, including an `ActorRef` that it can respond to. When the complete result is there the child responds with the result and stops itself.

As the protocol of the session actor is not a public API but  rather an implementation detail of the parent actor, it may not always  make sense to have an explicit protocol and adapt the messages of the  actors that the session actor interacts with. For this use case it is  possible to express that the actor can receive any message (`Any`).

**Example:**

![每个会话-child.png](https://doc.akka.io/docs/akka/current/typed/images/per-session-child.png)

- [          Scala         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab0)

  ​          ``

- [          Java         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab1)

In an actual session child you would likely want to include some form of timeout as well (see [scheduling messages to self](https://doc-akka-io.translate.goog/docs/akka/current/typed/interaction-patterns.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#scheduling-messages-to-self)).

**Useful when:**

- A single incoming request should result in multiple  interactions with other actors before a result can be built, for example aggregation of several results
- You need to handle acknowledgement and retry messages for at-least-once delivery

**Problems:**

- Children have life cycles that must be managed to not create a resource leak, it can be easy to miss a scenario where the session  actor is not stopped
- It increases complexity, since each such child can execute concurrently with other children and the parent

## General purpose response aggregator

This is similar to above [Per session child Actor](https://doc-akka-io.translate.goog/docs/akka/current/typed/interaction-patterns.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#per-session-child-actor) pattern. Sometimes you might end up repeating the same way of aggregating replies and want to extract that to a reusable actor.

There are many variations of this pattern and that is the  reason this is provided as a documentation example rather than a built  in `Behavior` in Akka. It is intended to be adjusted to your specific needs.

**Example:**

![聚合器.png](https://doc.akka.io/docs/akka/current/typed/images/aggregator.png)

This example is an aggregator of expected number of replies. Requests for quotes are sent with the given `sendRequests` function to the two hotel actors, which both speak different protocols. When both expected replies have been collected they are aggregated with the given `aggregateReplies` function and sent back to the `replyTo`. If replies don’t arrive within the `timeout` the replies so far are aggregated and sent back to the `replyTo`.

- [          Scala         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab0)

  ​          ``

- [          Java         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab1)

The implementation of the `Aggregator`:

- [          Scala         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab0)

  ​          ``

- [          Java         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab1)

**Useful when:**

- Aggregating replies are performed in the same way at multiple places and should be extracted to a more general purpose actor.
- A single incoming request should result in multiple  interactions with other actors before a result can be built, for example aggregation of several results
- You need to handle acknowledgement and retry messages for at-least-once delivery

**Problems:**

- Message protocols with generic types are difficult since the generic types are erased in runtime
- Children have life cycles that must be managed to not create a resource leak, it can be easy to miss a scenario where the session  actor is not stopped
- It increases complexity, since each such child can execute concurrently with other children and the parent

## Latency tail chopping

This is a variation of above [General purpose response aggregator](https://doc-akka-io.translate.goog/docs/akka/current/typed/interaction-patterns.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#general-purpose-response-aggregator) pattern.

The goal of this algorithm is to decrease tail latencies  (“chop off the tail latency”) in situations where multiple destination  actors can perform the same piece of work, and where an actor may  occasionally respond more slowly than expected. In this case, sending  the same work request (also known as a “backup request”) to another  actor results in decreased response time - because it’s less probable  that multiple actors are under heavy load simultaneously. This technique is explained in depth in Jeff Dean’s presentation on [Achieving Rapid Response Times in Large Online Services](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://static.googleusercontent.com/media/research.google.com/en//people/jeff/Berkeley-Latency-Mar2012.pdf).

There are many variations of this pattern and that is the  reason this is provided as a documentation example rather than a built  in `Behavior` in Akka. It is intended to be adjusted to your specific needs.

**Example:**

![斩尾.png](https://doc.akka.io/docs/akka/current/typed/images/tail-chopping.png)

- [          Scala         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab0)

  ​          ``

- [          Java         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab1)

**Useful when:**

- Reducing higher latency percentiles and variations of latency are important
- The “work” can be done more than once with the same result, e.g. a request to retrieve information

**Problems:**

- Increased load since more messages are sent and “work” is performed more than once
- Can’t be used when the “work” is not idempotent and must only be performed once
- Message protocols with generic types are difficult since the generic types are erased in runtime
- Children have life cycles that must be managed to not create a resource leak, it can be easy to miss a scenario where the session  actor is not stopped



## Scheduling messages to self

The following example demonstrates how to use timers to schedule messages to an actor. 

**Example:**

![计时器.png](https://doc.akka.io/docs/akka/current/typed/images/timer.png)

的`Buncher`演员缓冲器传入消息的突发，并提供它们作为超时或者当成批消息的数目超过最大大小后的批次。

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab1)

这里有几点值得注意：

- 要访问您开始使用的计时器`Behaviors.withTimers`，它将`TimerScheduler`向函数传递一个实例。这可以用于任何类型的`Behavior`，包括`receive`、`receiveMessage`，但也可以用于`setup`任何其他行为。
- 每个计时器都有一个键，如果启动了具有相同键的新计时器，则前一个计时器将被取消。保证不会收到来自前一个计时器的消息，即使它在新计时器启动时已经在邮箱中排队。
- 支持周期性和单消息定时器。
- 在`TimerScheduler`本身可变的，因为它执行和管理注册计划任务的副作用。
- 该`TimerScheduler`势必拥有它，当演员停止时自动取消演员的生命周期。
- `Behaviors.withTimers`也可以在里面使用`Behaviors.supervise`，当actor重新启动时它会自动正确地取消启动的计时器，这样新的化身就不会收到来自前一个化身的预定消息。

### 定期安排

重复消息的调度可以有两个不同的特征：

- 固定延迟 - 发送后续消息之间的延迟将始终（至少）是给定的`delay`。使用`startTimerWithFixedDelay`.
- 固定速率 - 随着时间的推移执行频率将满足给定的`interval`. 使用`startTimerAtFixedRate`.

如果您不确定要使用哪一个，您应该选择`startTimerWithFixedDelay`。

使用**固定延迟时**，如果由于某种原因调度的延迟时间超过指定**的时间**，则不会补偿消息之间的延迟。发送后续消息之间的延迟将始终（至少）是给定的`delay`。从长远来看，消息的频率一般会略低于指定的倒数`delay`。

固定延迟执行适用于需要“平稳性”的重复性活动。换句话说，它适用于短期内比长期保持频率准确更重要的活动。

使用**固定速率时**，如果先前的消息延迟太长，它将补偿后续任务的延迟。在这种情况下，实际发送间隔将与传递给`scheduleAtFixedRate`方法的间隔不同。

如果任务延迟时间超过`interval`，则后续消息将在前一个消息之后立即发送。这也会导致在长时间的垃圾收集暂停或其他原因导致 JVM 暂停后，所有“错过”的任务将在进程再次唤醒时执行。例如，`scheduleAtFixedRate`间隔为 1 秒，进程暂停 30 秒将导致 30 条消息快速连续发送以赶上。从长远来看，执行频率将恰好是指定`interval`.

Fixed-rate execution is appropriate for recurring activities  that are sensitive to absolute time or where the total time to perform a fixed number of executions is important, such as a countdown timer that ticks once every second for ten seconds.

​         Warning        

`scheduleAtFixedRate` can result in bursts of  scheduled messages after long garbage collection pauses, which may in  worst case cause undesired load on the system. `scheduleWithFixedDelay` is often preferred.

## Responding to a sharded actor

When [Akka Cluster](https://doc-akka-io.translate.goog/docs/akka/current/typed/cluster.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem) is used to [shard actors](https://doc-akka-io.translate.goog/docs/akka/current/typed/cluster-sharding.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem) you need to take into account that an actor may move or get passivated.

期待回复的正常模式是在消息中包含一个，通常是一个消息适配器。这可以用于分片演员，但如果发送并且分片演员被移动或钝化，那么回复将发送到死信。[`ActorRef`](https://doc-akka-io.translate.goog/api/akka/2.6/akka/actor/typed/ActorRef.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)`ctx.self`

另一种方法是`entityId`在消息中发送 并通过分片发送回复。

**例子：**

![分片响应.png](https://doc.akka.io/docs/akka/current/typed/images/sharded-response.png)

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html#tab1)

一个缺点是不能使用消息适配器，因此响应必须在被响应的参与者的协议中。此外，`EntityTypeKey`如果它不是静态已知的，则可以包含在消息中。

作为“替代方案的替代方案”，可以包含在消息中。的透明包装在一个消息，并通过分片发送。如果目标分片实体已被钝化，它将被交付给该实体的新化身；如果目标分片实体已移动到不同的集群节点，它将被路由到该新节点。如果使用这种方法，请注意，此时[需要自定义序列化程序](https://doc-akka-io.translate.goog/docs/akka/current/typed/cluster-sharding.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#a-note-about-entityref-and-serialization)。[`EntityRef`](https://doc-akka-io.translate.goog/api/akka/2.6/akka/cluster/sharding/typed/scaladsl/EntityRef.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)`EntityRef``ShardingEnvelope`

与直接在消息中包含`entityId`和一样`EntityTypeKey`，`EntityRef`s 不支持消息适配：响应必须在被响应实体的协议中。

在某些情况下，使用 a 定义消息可能很有用，a是and的常见超类型。这时候序列化a需要自定义序列化器。[`RecipientRef`](https://doc-akka-io.translate.goog/api/akka/2.6/akka/actor/typed/RecipientRef.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)`ActorRef``EntityRef``RecipientRef`

----

[容错](fault-tolerance.md)

----

https://doc.akka.io/docs/akka/current/typed/interaction-patterns.html