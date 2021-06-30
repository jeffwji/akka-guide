# 第 3 部分: 使用设备 Actors
## 简介
在前面的主题中，我们解释了如何在大范围内查看 Actor 系统，也就是组件应该如何来表示，如何在层次结构中安排 Actor。在这一部分中，我们将通过实现设备 Actor 来在小范围内观察 Actor。

如果我们使用对象，我们通常将 API 设计为接口，由实际实现来填充抽象方法集合。在 Actor 的世界里，协议取代了接口。虽然在编程语言中无法将一般协议形式化，但是我们可以组合它们最基本的元素、消息。因此，我们将从识别我们要发送给设备 Actor 的消息开始。

通常，消息分为类别或模式。通过识别这些模式，你将发现在它们之间进行选择和实现变得更加容易。第一个示例演示“请求-响应”消息模式。

## 识别设备的消息
设备 Actor 的任务很简单：

- 收集温度测量值
- 当被询问时，报告上次测量的温度

然而，设备可能在没有立即进行温度测量的情况下启动。因此，我们需要考虑温度不存在的情况。这还允许我们在不存在写入部分的时候测试 Actor 的查询部分，因为设备 Actor 可以报告空结果。

从设备 Actor 获取当前温度的协议很简单。Actor：

- 等待当前温度的请求。
- 对请求作出响应，并答复：
  - 包含当前温度，或者
  - 指示温度尚不可用。

我们需要两条消息，一条用于请求，一条用于回复。我们的第一次尝试可能如下：

```scala
package com.example

  import akka.actor.typed.ActorRef

  object Device {
    sealed trait Command
    final case class ReadTemperature(replyTo: ActorRef[RespondTemperature]) extends Command
    final case class RespondTemperature(value: Option[Double])
  }
```
请注意，该`ReadTemperature`消息包含设备actor在回复请求时将使用的 。`ActorRef[RespondTemperature]`

这两条消息似乎涵盖了所需的功能。但是，我们选择的方法必须考虑到应用程序的分布式性质。虽然本地 JVM 上的 Actor 通信的基本机制与远程 Actor 通信的基本机制相同，但我们需要记住以下几点：

- 因为网络链路带宽和消息大小等因素的存在，本地和远程消息在传递延迟方面会有明显的差异。
- 可靠性是一个问题，因为远程消息发送需要更多的步骤，这意味着更多的步骤可能出错。
- 本地发送将在同一个 JVM 中传递对消息的引用，而对发送的底层对象没有任何限制，而远程传输将限制消息的大小。

此外，虽然在同一个 JVM 内发送明显更加可靠，但是如果一个 Actor 在处理消息时由于编程错误而失败，则效果与处理消息时由于远程主机崩溃而导致远程网络请求失败的效果相同。尽管在这两种情况下，服务会在一段时间后恢复（Actor 由其监督者重新启动，主机由操作员或监控系统重新启动），但在崩溃期间，单个请求会丢失。因此，你的 Actor 代码发送的每一条信息都可能丢失，这是一个安全的、悲观的赌注。

但是如果进一步理解协议灵活性的需求，它将有助于考虑 Akka 消息订阅和消息传递的安全保证。Akka 为消息发送提供以下行为：

- 至多发送一次消息，即消息发送没有保证；
- 按“发送方、接收方”对来维护消息顺序。

以下各节更详细地讨论了此行为：

- [消息传递](#消息传递)
- [消息序列](#消息序列)

### 消息传递
消息子系统提供的传递语义通常分为以下类别：

- 最多一次传递：`At-most-once delivery`，每一条消息都是传递零次或一次；在更因果关系的术语中，这意味着消息可能会丢失，但永远不会重复。
- 至少一次传递：`At-least-once delivery`，可能多次尝试传递每条消息，直到至少一条成功；同样，在更具因果关系的术语中，这意味着消息可能重复，但永远不会丢失。
- 恰好一次传递：`Exactly-once delivery`，每条消息只给收件人传递一次；消息既不能丢失，也不能重复。

第一种“最多一次传递”是 Akka 使用的方式，它是最廉价也是性能最好的方式。它具有最小的实现开销，因为它可以以一种“发完即忘（`fire-and-forget`）”的方式完成，而不需要将状态保持在发送端或传输机制中。第二个，“至少一次传递”，需要重试以抵消传输损失。这增加了在发送端保持状态和在接收端具有确认机制的开销。“恰好一次传递”最为昂贵，并且会导致最差的性能：除了“至少一次传递”所增加的开销之外，它还要求将状态保留在接收端，以便过滤重复传递的内容。

在 Actor 系统中，我们需要确切含义——即在哪一点上，系统认为消息传递完成：

 1. 消息何时在网络上发送？
 2. 目标 Actor 的主机何时接收消息？
 3. 消息何时被放入目标 Actor 的邮箱？
 4. 消息目标 Actor 何时开始处理消息？
 5. 目标 Actor 何时成功处理完消息？

大多数声称保证传递的框架和协议实际上提供了类似于第 4 点和第 5 点的内容。虽然这听起来很合理，但它真的有用吗？要理解其含义，请考虑一个简单、实用的示例：当用户尝试下单时，我们希望只有当订单数据被实际保存到数据库的磁盘上后，才说订单已成功处理。

如果我们依赖消息的成功处理，那么一旦订单被提交给负责验证、处理它并负责与数据库通讯的 API时，Actor 就会报告成功。不幸的是，在调用 API 之后，可能会立即发生以下任何情况：

- 主机可能崩溃。
- 反序列化可能失败。
- 验证可能失败。
- 数据库可能不可用。
- 可能发生编程错误。

这说明**交付**的**保证**并没有转化为**域级别的保证**。我们希望在订单被实际完全处理和持久化后报告成功。**唯一能够报告成功的实体是应用程序本身，因为只有它对所需的域保证最了解**。没有一个通用的框架能够弄清楚特定领域的细节，以及在该领域中什么被认为是成功的。

在这个特定的例子中，我们只希望在数据库成功写入之后就发出成功的信号，在这里数据库确认订单现在已安全存储。基于这些原因，Akka将保证的责任提升到应用程序本身，即你必须自己使用 Akka 提供的工具来实现这些保证。这使你能够完全控制你想要提供的保证。现在，让我们考虑一下 Akka 提供的消息序列，它可以很容易地解释应用程序逻辑。

### 消息序列
在 Akka 中 ，对于一对给定的 Actor，直接从第一个 Actor 发送到第二个 Actor 的消息不会被无序接收。该词直接强调这个保证仅在与`tell`运算符直接发送到最终目的地时适用，而在使用中介时不适用。

如果：

- Actor A1 向 A2 发送消息`M1`、`M2`和`M3`。
- Actor A3 向 A2 发送消息`M4`、`M5`和`M6`。

这意味着，对于 Akka 信息：

- 如果`M1`已交付，则必须在`M2`和`M3`之前交付。
- 如果`M2`已交付，则必须在`M3`之前交付。
- 如果`M4`已交付，则必须在`M5`和`M6`之前交付。
- 如果`M5`已交付，则必须在`M6`之前交付。
- A2 可以看到 A1 的消息与 A3 的消息交织在一起。
- 由于没有保证的传递，任何消息都可能丢失，即不能到达 A2。

这些保证实现了一个良好的平衡：让一个 Actor 发送的消息有序到达，便于构建易于推理的系统，而另一方面，允许不同 Actor 发送的消息交错到达，则为 Actor 系统的有效实现提供了足够的自由度。

有关消息传递保证的详细信息，请参阅「[消息传递可靠性](../general-concepts/message-delivery-reliability.md)」。

## 增加设备消息的灵活性

我们的第一个查询协议是正确的，但没有考虑分布式应用程序的执行。如果我们想在 actor 中实现重发（因为请求超时），或者如果我们想查询多个 Actor，我们需要能够关联请求和响应。因此，我们在消息中再添加一个字段，这样请求者就可以提供一个 ID（我们将在稍后的步骤中将此代码添加到我们的应用程序中）：

```java
sealed trait Command
final case class ReadTemperature(requestId: Long, replyTo: ActorRef[RespondTemperature]) extends Command
final case class RespondTemperature(requestId: Long, value: Option[Double])
```
## 实现设备 Actor 及其读取协议

正如我们在`Hello World`示例中了解到的，每个 Actor 都定义了它接受的消息类型。我们的设备 Actor 有责任为给定查询的响应使用相同的 ID 参数，这将使它看起来像下面这样。

```scala
import akka.actor.typed.Behavior
import akka.actor.typed.scaladsl.AbstractBehavior
import akka.actor.typed.scaladsl.ActorContext
import akka.actor.typed.scaladsl.Behaviors
import akka.actor.typed.scaladsl.LoggerOps

object Device {
  def apply(groupId: String, deviceId: String): Behavior[Command] =
    Behaviors.setup(context => new Device(context, groupId, deviceId))

  sealed trait Command
  final case class ReadTemperature(requestId: Long, replyTo: ActorRef[RespondTemperature]) extends Command
  final case class RespondTemperature(requestId: Long, value: Option[Double])
}

class Device(context: ActorContext[Device.Command], groupId: String, deviceId: String)
    extends AbstractBehavior[Device.Command](context) {
  import Device._

  var lastTemperatureReading: Option[Double] = None

  context.log.info2("Device actor {}-{} started", groupId, deviceId)

  override def onMessage(msg: Command): Behavior[Command] = {
    msg match {
      case ReadTemperature(id, replyTo) =>
        replyTo ! RespondTemperature(id, lastTemperatureReading)
        this
    }
  }

  override def onSignal: PartialFunction[Signal, Behavior[Command]] = {
    case PostStop =>
      context.log.info2("Device actor {}-{} stopped", groupId, deviceId)
      this
  }

}
```
在上述代码中需要注意：

- 在伴随对象的`apply`方法中定义了如何为设备actor构建Behavior。参数包括设备的 ID 和它所属的组，我们稍后会用到。

  我们之前介绍的消息在伴随对象中定义。

  在`Device`类中，`lastTemperatureReading` 的值最初设置为 None，actor 将在被查询时将它报告给对方。

## 测试 Actor

基于上面的actor，我们可以编写一个测试案例。在`com.example`项目测试树的包中，将以下代码添加到`DeviceSpec.scala`文件中。（我们使用 ScalaTest，但任何其他测试框架都可以与 Akka Testkit 一起使用）。

您可以通过`test`在 sbt 提示符下运行来运行此测试。

```java
import akka.actor.testkit.typed.scaladsl.ScalaTestWithActorTestKit
import org.scalatest.wordspec.AnyWordSpecLike

class DeviceSpec extends ScalaTestWithActorTestKit with AnyWordSpecLike {
  import Device._

  "Device actor" must {

    "reply with empty reading if no temperature is known" in {
      val probe = createTestProbe[RespondTemperature]()
      val deviceActor = spawn(Device("group", "device"))

      deviceActor ! Device.ReadTemperature(requestId = 42, probe.ref)
      val response = probe.receiveMessage()
      response.requestId should ===(42)
      response.value should ===(None)
    }
}
```

现在，当 Actor 收到来自传感器的信息时，它需要一种方法来改变温度的状态。

## 添加写入协议

写入协议（`write protocol`）的目的是在 Actor 收到包含温度的消息时更新`currentTemperature`字段。同样，很容易将写协议定义为一个非常简单的消息，如下所示：

```java
sealed trait Command
final case class RecordTemperature(value: Double) extends Command
```
但是，这种方法没有考虑到记录温度消息的发送者永远无法确定消息是否被处理。我们已经看到，Akka 不保证这些消息的传递，而是让应用程序提供成功通知。在我们的情况下，一旦我们更新了上次的温度记录，我们想向发件人发送确认例如回复一条`TemperatureRecorded`消息。就像在温度查询和响应的情况下一样，最好包含一个 ID 字段以提供最大的灵活性。

```scala
final case class RecordTemperature(requestId: Long, value: Double, replyTo: ActorRef[TemperatureRecorded])
    extends Command
final case class TemperatureRecorded(requestId: Long)
```

## 具有读写消息的 Actor

将读写协议放在一起，设备 Actor 如下所示：

```java
import akka.actor.typed.ActorRef
import akka.actor.typed.Behavior
import akka.actor.typed.PostStop
import akka.actor.typed.Signal
import akka.actor.typed.scaladsl.AbstractBehavior
import akka.actor.typed.scaladsl.ActorContext
import akka.actor.typed.scaladsl.Behaviors
import akka.actor.typed.scaladsl.LoggerOps

object Device {
  def apply(groupId: String, deviceId: String): Behavior[Command] =
    Behaviors.setup(context => new Device(context, groupId, deviceId))

  sealed trait Command

  final case class ReadTemperature(requestId: Long, replyTo: ActorRef[RespondTemperature]) extends Command
  final case class RespondTemperature(requestId: Long, value: Option[Double])

  final case class RecordTemperature(requestId: Long, value: Double, replyTo: ActorRef[TemperatureRecorded])
      extends Command
  final case class TemperatureRecorded(requestId: Long)
}

class Device(context: ActorContext[Device.Command], groupId: String, deviceId: String)
    extends AbstractBehavior[Device.Command](context) {
  import Device._

  var lastTemperatureReading: Option[Double] = None

  context.log.info2("Device actor {}-{} started", groupId, deviceId)

  override def onMessage(msg: Command): Behavior[Command] = {
    msg match {
      case RecordTemperature(id, value, replyTo) =>
        context.log.info2("Recorded temperature reading {} with {}", value, id)
        lastTemperatureReading = Some(value)
        replyTo ! TemperatureRecorded(id)
        this

      case ReadTemperature(id, replyTo) =>
        replyTo ! RespondTemperature(id, lastTemperatureReading)
        this
    }
  }

  override def onSignal: PartialFunction[Signal, Behavior[Command]] = {
    case PostStop =>
      context.log.info2("Device actor {}-{} stopped", groupId, deviceId)
      this
  }

}
```
我们现在还应该编写一个新的测试用例，同时使用读/查询和写/记录功能：

```java
"reply with latest temperature reading" in {
  val recordProbe = createTestProbe[TemperatureRecorded]()
  val readProbe = createTestProbe[RespondTemperature]()
  val deviceActor = spawn(Device("group", "device"))

  deviceActor ! Device.RecordTemperature(requestId = 1, 24.0, recordProbe.ref)
  recordProbe.expectMessage(Device.TemperatureRecorded(requestId = 1))

  deviceActor ! Device.ReadTemperature(requestId = 2, readProbe.ref)
  val response1 = readProbe.receiveMessage()
  response1.requestId should ===(2)
  response1.value should ===(Some(24.0))

  deviceActor ! Device.RecordTemperature(requestId = 3, 55.0, recordProbe.ref)
  recordProbe.expectMessage(Device.TemperatureRecorded(requestId = 3))

  deviceActor ! Device.ReadTemperature(requestId = 4, readProbe.ref)
  val response2 = readProbe.receiveMessage()
  response2.requestId should ===(4)
  response2.value should ===(Some(55.0))
}
```

## 下一步是什么？

到目前为止，我们已经开始设计我们的总体架构，并且我们编写了第一个直接对应于域的 Actor。我们现在必须创建负责维护设备组和设备 Actor 本身的组件。

----------

[第 4 部分：使用设备组](tutorial_4)



**英文原文链接**：[Part 3: Working with Device Actors](https://doc.akka.io/docs/akka/current/guide/tutorial_3.html).

----------
———— ☆☆☆ —— [返回 -> Akka 中文指南 <- 目录](https://github.com/guobinhit/akka-guide/blob/master/README.md) —— ☆☆☆ ————