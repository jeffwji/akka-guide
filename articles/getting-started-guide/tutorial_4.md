# 第 4 部分: 使用设备组
## 简介
让我们仔细看看用例所需的主要功能。在用于监测家庭温度的完整物联网系统中，将设备传感器连接到系统的步骤可能如下：

 1. 家庭中的传感器设备通过某种协议进行连接。
 2. 管理网络连接的组件接受连接。
 3. 传感器提供其组和设备 ID，以便在系统的设备管理器组件中注册。
 4. 设备管理器组件通过查找或创建负责保持传感器状态的 Actor 来处理注册。
 5. Actor 以确认（`acknowledgement`）回应，暴露其`ActorRef`。
 6. 网络组件现在使用`ActorRef`在传感器和设备 Actor 之间进行通信，而不需要经过设备管理器。

步骤 1 和 2 发生在教程系统的边界之外。在本章中，我们将开始处理步骤 3 - 6，并创建传感器在系统中注册和与 Actor 通信的方法。但首先，我们有另一个体系结构决策——我们应该使用多少个层次的 Actor 来表示设备组和设备传感器？

Akka 程序员面临的主要设计挑战之一是为 Actor 选择最佳的粒度。在实践中，根据 Actor 之间交互的特点，通常有几种有效的方法来组织系统。例如，在我们的用例中，可能有一个 Actor 维护所有的组和设备——或许可以使用哈希表（`hash maps`）。对于每个跟踪同一个家中所有设备状态的组来说，有一个 Actor 也是合理的。

以下指导原则可以帮助我们选择最合适的 Actor 层次结构：

- 一般来说，更倾向于更大的粒度。引入比需要更多的细粒度 Actor 会导致比它解决的问题更多的问题。
- 当系统需要时添加更细的粒度：
  - 更高的并发性。
  - 有许多状态的 Actor 之间的复杂交互。在下一章中，我们将看到一个很好的例子。
  - 足够多的状态，划分为较小的 Actor 是有意义地。
  - 多重无关责任。使用不同的 Actor 可以使单个 Actor 失败并恢复，而对其他的 Actor 影响很小。

## 设备管理器层次结构

考虑到上一节中概述的原则，我们将设备管理器组件建模为具有三个级别的 Actor 树：

- 顶级监督者 Actor 表示设备的系统组件。它也是查找和创建设备组和设备 Actor 的入口点。
- 在下一个级别，每个组 Actor 都监督设备 Actor 使用同一个组 ID（例如指代同一个家庭）。它们还提供服务，例如查询组中所有可用设备的温度读数。
- 设备 Actor 管理与实际设备传感器的所有交互，例如存储温度读数。

![device-manager](../../images/getting-started-guide/tutorial_4/device-manager.png)

我们选择这三层架构的原因如下：

- 划分组为单独的 Actor：
  - 隔离组中发生的故障。如果一个 Actor 管理所有设备组，则一个组中导致重启的错误将消除其他非故障组的状态。
  - 简化了查询属于一个组的所有设备的问题。每个组 Actor 只包含与其组相关的状态。
  - 提高系统的并行性。因为每个组都有一个专用的 Actor，所以它们可以并发运行，我们可以并发查询多个组。
- 将传感器建模为单个设备 Actor：
  - 将一个设备 Actor 的故障与组中的其他设备隔离开来。
  - 增加收集温度读数的平行度。来自不同传感器的网络连接直接与各自的设备 Actor 通信，从而减少了竞争点。

定义了设备体系结构后，我们就可以开始研究注册传感器的协议了。

## 注册协议

作为第一步，我们需要设计协议来注册一个设备，以及创建负责它的组和设备 Actor。此协议将由`DeviceManager`组件本身提供，因为它是唯一已知且预先可用的 Actor：设备组和设备 Actor 是按需创建的。

更详细地看一下注册，我们可以概述必要的功能：

- 当`DeviceManager`接收到具有组和设备 ID 的请求时：
  - 如果管理器已经有了设备组的 Actor，那么它会将请求转发给它。
  - 否则，它会创建一个新的设备组 Actor，然后转发请求。
- `DeviceGroup` Actor 接收为给定设备注册 Actor 的请求：
  - 如果组已经有设备的 Actor，则组 Actor 将请求转发给设备 Actor。
  - 否则，设备组 Actor 首先创建设备 Actor，然后转发请求。
- 传感器现在可以向设备actor的`ActorRef`直接向其发送消息。

我们将用来传递注册请求及其确认的消息有一个简单的定义：

```scala
final case class RequestTrackDevice(groupId: String, deviceId: String, replyTo: ActorRef[DeviceRegistered])
    extends DeviceManager.Command
    with DeviceGroup.Command

final case class DeviceRegistered(device: ActorRef[Device.Command])
```
在这种情况下，我们在消息中没有包含请求 ID 字段。由于注册只发生一次，当组件将系统连接到某个网络协议时，ID 并不重要。但是，包含请求 ID 通常是一种最佳实践。

现在，我们将从头开始实现该协议。在实践中，自上向下和自下而上的方法都很有效，但是在我们的例子中，我们实使用自下而上的方法，因为它允许我们立即为新特性编写测试，而不需要模拟出稍后需要构建的部分。

## 向设备 Actor 添加注册支持

group actor 在注册时有一些工作要做，包括：

- 处理现有device actor的注册请求或创建新actor。
- 跟踪组中存在哪些device actor，并在停止时将其从组中删除。

### 处理注册请求

设备组 actor 必须或者向请求返回一个已经存在的子actor 的`ActorRef`，或者应该创建一个新的。要通过设备 ID 查找子actor，我们将使用`Map`.

将以下内容添加到您的源文件中：

```scala
object DeviceGroup {
  def apply(groupId: String): Behavior[Command] =
    Behaviors.setup(context => new DeviceGroup(context, groupId))

  trait Command

  private final case class DeviceTerminated(device: ActorRef[Device.Command], groupId: String, deviceId: String)
      extends Command

}

class DeviceGroup(context: ActorContext[DeviceGroup.Command], groupId: String)
    extends AbstractBehavior[DeviceGroup.Command](context) {
  import DeviceGroup._
  import DeviceManager.{ DeviceRegistered, ReplyDeviceList, RequestDeviceList, RequestTrackDevice }

  private var deviceIdToActor = Map.empty[String, ActorRef[Device.Command]]

  context.log.info("DeviceGroup {} started", groupId)

  override def onMessage(msg: Command): Behavior[Command] =
    msg match {
      case trackMsg @ RequestTrackDevice(`groupId`, deviceId, replyTo) =>
        deviceIdToActor.get(deviceId) match {
          case Some(deviceActor) =>
            replyTo ! DeviceRegistered(deviceActor)
          case None =>
            context.log.info("Creating device actor for {}", trackMsg.deviceId)
            val deviceActor = context.spawn(Device(groupId, deviceId), s"device-$deviceId")
            deviceIdToActor += deviceId -> deviceActor
            replyTo ! DeviceRegistered(deviceActor)
        }
        this

      case RequestTrackDevice(gId, _, _) =>
        context.log.warn2("Ignoring TrackDevice request for {}. This actor is responsible for {}.", gId, groupId)
        this
    }

  override def onSignal: PartialFunction[Signal, Behavior[Command]] = {
    case PostStop =>
      context.log.info("DeviceGroup {} stopped", groupId)
      this
  }
}
```
正如我们对设备所做的那样，我们测试了这个新功能。我们还测试了为两个不同 ID 返回的actor实际上是不同的，我们还尝试记录每个设备的温度读数以查看actor是否有响应。

```scala
"be able to register a device actor" in {
  val probe = createTestProbe[DeviceRegistered]()
  val groupActor = spawn(DeviceGroup("group"))

  groupActor ! RequestTrackDevice("group", "device1", probe.ref)
  val registered1 = probe.receiveMessage()
  val deviceActor1 = registered1.device

  // another deviceId
  groupActor ! RequestTrackDevice("group", "device2", probe.ref)
  val registered2 = probe.receiveMessage()
  val deviceActor2 = registered2.device
  deviceActor1 should !==(deviceActor2)

  // Check that the device actors are working
  val recordProbe = createTestProbe[TemperatureRecorded]()
  deviceActor1 ! RecordTemperature(requestId = 0, 1.0, recordProbe.ref)
  recordProbe.expectMessage(TemperatureRecorded(requestId = 0))
  deviceActor2 ! Device.RecordTemperature(requestId = 1, 2.0, recordProbe.ref)
  recordProbe.expectMessage(Device.TemperatureRecorded(requestId = 1))
}

"ignore requests for wrong groupId" in {
  val probe = createTestProbe[DeviceRegistered]()
  val groupActor = spawn(DeviceGroup("group"))

  groupActor ! RequestTrackDevice("wrongGroup", "device1", probe.ref)
  probe.expectNoMessage(500.milliseconds)
}
```
如果注册请求的设备actor已经存在，我们希望使用现有actor而不是新actor。我们还没有测试过这个，所以我们需要解决这个问题：

```scala
"return same actor for same deviceId" in {
  val probe = createTestProbe[DeviceRegistered]()
  val groupActor = spawn(DeviceGroup("group"))

  groupActor ! RequestTrackDevice("group", "device1", probe.ref)
  val registered1 = probe.receiveMessage()

  // registering same again should be idempotent
  groupActor ! RequestTrackDevice("group", "device1", probe.ref)
  val registered2 = probe.receiveMessage()

  registered1.device should ===(registered2.device)
}
```

### 跟踪组中的device actor

到目前为止，我们已经实现了在组中注册设备 actor 的逻辑。然而，设备来来去去，所以当一个设备停止的时候我们需要一种方法来从 `Map[String, ActorRef[DeviceMessage]]` 中移除它。正如我们之前讨论的那样，监督处理错误——而不是优雅的停止。因此，当其中一个设备 actor 停止时，我们需要通知父级（来处理）。

Akka 提供了一个*Death Watch*功能，允许一个actor*观看*另一个actor并在另一个actor停止时得到通知。与监督不同，观看不限于亲子关系，任何actor都可以观看任何其他actor，只要它知道`ActorRef`。在被监视的actor 停止后，watcher 会收到一个`Terminated(actorRef)`信号，其中还包含对被监视的actor 的引用。观察者可以显式地处理此消息，也可以抛出`DeathPactException`失败。如果在被监视的actor被停止后不能再执行自己的职责，后者是有用的。在我们的例子中，组应该在任意一台设备停止后仍然工作，因此我们需要处理`Terminated(actorRef)`信号。

我们的设备组actor需要包含以下功能：

1. 在创建新设备 actor 时开始观察它们。
2. 当通知表明它已停止时，从 `Map[String, ActorRef[DeviceMessage]]` 中将设备删除。

不幸的是，该`Terminated`信号仅包含子 actor 的 `ActorRef`。我们需要子 actor 的 ID 才能将其从映射中删除。替代`Terminated`信号的方法是定义一个自定义消息，当被监视的 actor 停止时将发送该消息。我们将在这里使用它，因为它使我们可以在该消息中携带设备 ID。

添加识别角色的功能代码：

```scala
class DeviceGroup(context: ActorContext[DeviceGroup.Command], groupId: String)
    extends AbstractBehavior[DeviceGroup.Command](context) {
  import DeviceGroup._
  import DeviceManager.{ DeviceRegistered, ReplyDeviceList, RequestDeviceList, RequestTrackDevice }

  private var deviceIdToActor = Map.empty[String, ActorRef[Device.Command]]

  context.log.info("DeviceGroup {} started", groupId)

  override def onMessage(msg: Command): Behavior[Command] =
    msg match {
      case trackMsg @ RequestTrackDevice(`groupId`, deviceId, replyTo) =>
        deviceIdToActor.get(deviceId) match {
          case Some(deviceActor) =>
            replyTo ! DeviceRegistered(deviceActor)
          case None =>
            context.log.info("Creating device actor for {}", trackMsg.deviceId)
            val deviceActor = context.spawn(Device(groupId, deviceId), s"device-$deviceId")
            context.watchWith(deviceActor, DeviceTerminated(deviceActor, groupId, deviceId))
            deviceIdToActor += deviceId -> deviceActor
            replyTo ! DeviceRegistered(deviceActor)
        }
        this

      case RequestTrackDevice(gId, _, _) =>
        context.log.warn2("Ignoring TrackDevice request for {}. This actor is responsible for {}.", gId, groupId)
        this

      case DeviceTerminated(_, _, deviceId) =>
        context.log.info("Device actor for {} has been terminated", deviceId)
        deviceIdToActor -= deviceId
        this

    }

  override def onSignal: PartialFunction[Signal, Behavior[Command]] = {
    case PostStop =>
      context.log.info("DeviceGroup {} stopped", groupId)
      this
  }
}
```

到目前为止，我们还无法获得群组设备actor跟踪的设备列表，因此，我们还无法测试我们的新功能。为了使其可测试，我们添加了一个新的查询功能（消息`RequestDeviceList`），列出当前活动的设备 ID：

```scala
final case class RequestDeviceList(requestId: Long, groupId: String, replyTo: ActorRef[ReplyDeviceList])
    extends DeviceManager.Command
    with DeviceGroup.Command

final case class ReplyDeviceList(requestId: Long, ids: Set[String])
```

```scala
object DeviceGroup {
  def apply(groupId: String): Behavior[Command] =
    Behaviors.setup(context => new DeviceGroup(context, groupId))

  trait Command

  private final case class DeviceTerminated(device: ActorRef[Device.Command], groupId: String, deviceId: String)
      extends Command

}

class DeviceGroup(context: ActorContext[DeviceGroup.Command], groupId: String)
    extends AbstractBehavior[DeviceGroup.Command](context) {
  import DeviceGroup._
  import DeviceManager.{ DeviceRegistered, ReplyDeviceList, RequestDeviceList, RequestTrackDevice }

  private var deviceIdToActor = Map.empty[String, ActorRef[Device.Command]]

  context.log.info("DeviceGroup {} started", groupId)

  override def onMessage(msg: Command): Behavior[Command] =
    msg match {
      case trackMsg @ RequestTrackDevice(`groupId`, deviceId, replyTo) =>
        deviceIdToActor.get(deviceId) match {
          case Some(deviceActor) =>
            replyTo ! DeviceRegistered(deviceActor)
          case None =>
            context.log.info("Creating device actor for {}", trackMsg.deviceId)
            val deviceActor = context.spawn(Device(groupId, deviceId), s"device-$deviceId")
            context.watchWith(deviceActor, DeviceTerminated(deviceActor, groupId, deviceId))
            deviceIdToActor += deviceId -> deviceActor
            replyTo ! DeviceRegistered(deviceActor)
        }
        this

      case RequestTrackDevice(gId, _, _) =>
        context.log.warn2("Ignoring TrackDevice request for {}. This actor is responsible for {}.", gId, groupId)
        this

      case RequestDeviceList(requestId, gId, replyTo) =>
        if (gId == groupId) {
          replyTo ! ReplyDeviceList(requestId, deviceIdToActor.keySet)
          this
        } else
          Behaviors.unhandled

      case DeviceTerminated(_, _, deviceId) =>
        context.log.info("Device actor for {} has been terminated", deviceId)
        deviceIdToActor -= deviceId
        this

    }

  override def onSignal: PartialFunction[Signal, Behavior[Command]] = {
    case PostStop =>
      context.log.info("DeviceGroup {} stopped", groupId)
      this
  }
}
```

我们已经几乎准备好测试设备的移除了。但是在此之前，我们还需要添加以下功能：

- 在我们的测试用例中从外部停止设备actor，我们必须向它发送一条消息。我们添加一条`Passivate`消息，指示 actor 停止。
- 一旦device actor停止，就会收到通知。我们可以为此目的使用*DeathWatch* 设施。

```scala
case object Passivate extends Command
```

```scala
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

  case object Passivate extends Command
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

      case Passivate =>
        Behaviors.stopped
    }
  }

  override def onSignal: PartialFunction[Signal, Behavior[Command]] = {
    case PostStop =>
      context.log.info2("Device actor {}-{} stopped", groupId, deviceId)
      this
  }

}
```

现在我们再添加两个测试用例。首先，我们测试一旦我们添加了一些设备，我们就可以返回正确的 ID 列表。第二个测试用例确保在device actor停止后正确删除设备 ID。该`TestProbe`有一个`expectTerminated`方法，我们可以很容易地使用断言判断该设备的actor已经终止。

```scala
"be able to list active devices" in {
  val registeredProbe = createTestProbe[DeviceRegistered]()
  val groupActor = spawn(DeviceGroup("group"))

  groupActor ! RequestTrackDevice("group", "device1", registeredProbe.ref)
  registeredProbe.receiveMessage()

  groupActor ! RequestTrackDevice("group", "device2", registeredProbe.ref)
  registeredProbe.receiveMessage()

  val deviceListProbe = createTestProbe[ReplyDeviceList]()
  groupActor ! RequestDeviceList(requestId = 0, groupId = "group", deviceListProbe.ref)
  deviceListProbe.expectMessage(ReplyDeviceList(requestId = 0, Set("device1", "device2")))
}

"be able to list active devices after one shuts down" in {
  val registeredProbe = createTestProbe[DeviceRegistered]()
  val groupActor = spawn(DeviceGroup("group"))

  groupActor ! RequestTrackDevice("group", "device1", registeredProbe.ref)
  val registered1 = registeredProbe.receiveMessage()
  val toShutDown = registered1.device

  groupActor ! RequestTrackDevice("group", "device2", registeredProbe.ref)
  registeredProbe.receiveMessage()

  val deviceListProbe = createTestProbe[ReplyDeviceList]()
  groupActor ! RequestDeviceList(requestId = 0, groupId = "group", deviceListProbe.ref)
  deviceListProbe.expectMessage(ReplyDeviceList(requestId = 0, Set("device1", "device2")))

  toShutDown ! Passivate
  registeredProbe.expectTerminated(toShutDown, registeredProbe.remainingOrDefault)

  // using awaitAssert to retry because it might take longer for the groupActor
  // to see the Terminated, that order is undefined
  registeredProbe.awaitAssert {
    groupActor ! RequestDeviceList(requestId = 1, groupId = "group", deviceListProbe.ref)
    deviceListProbe.expectMessage(ReplyDeviceList(requestId = 1, Set("device2")))
  }
}
```

## 创建设备管理器角色

进入我们层次结构的下一个级别，我们需要在`DeviceManager`源文件中为我们的设备管理组件创建入口点。该actor与设备组actor非常相似，但它创建的是设备组actor而不是设备actor：

```scala
object DeviceManager {
  def apply(): Behavior[Command] =
    Behaviors.setup(context => new DeviceManager(context))


  sealed trait Command

  final case class RequestTrackDevice(groupId: String, deviceId: String, replyTo: ActorRef[DeviceRegistered])
      extends DeviceManager.Command
      with DeviceGroup.Command

  final case class DeviceRegistered(device: ActorRef[Device.Command])

  final case class RequestDeviceList(requestId: Long, groupId: String, replyTo: ActorRef[ReplyDeviceList])
      extends DeviceManager.Command
      with DeviceGroup.Command

  final case class ReplyDeviceList(requestId: Long, ids: Set[String])

  private final case class DeviceGroupTerminated(groupId: String) extends DeviceManager.Command
}

class DeviceManager(context: ActorContext[DeviceManager.Command])
    extends AbstractBehavior[DeviceManager.Command](context) {
  import DeviceManager._

  var groupIdToActor = Map.empty[String, ActorRef[DeviceGroup.Command]]

  context.log.info("DeviceManager started")

  override def onMessage(msg: Command): Behavior[Command] =
    msg match {
      case trackMsg @ RequestTrackDevice(groupId, _, replyTo) =>
        groupIdToActor.get(groupId) match {
          case Some(ref) =>
            ref ! trackMsg
          case None =>
            context.log.info("Creating device group actor for {}", groupId)
            val groupActor = context.spawn(DeviceGroup(groupId), "group-" + groupId)
            context.watchWith(groupActor, DeviceGroupTerminated(groupId))
            groupActor ! trackMsg
            groupIdToActor += groupId -> groupActor
        }
        this

      case req @ RequestDeviceList(requestId, groupId, replyTo) =>
        groupIdToActor.get(groupId) match {
          case Some(ref) =>
            ref ! req
          case None =>
            replyTo ! ReplyDeviceList(requestId, Set.empty)
        }
        this

      case DeviceGroupTerminated(groupId) =>
        context.log.info("Device group actor for {} has been terminated", groupId)
        groupIdToActor -= groupId
        this
    }

  override def onSignal: PartialFunction[Signal, Behavior[Command]] = {
    case PostStop =>
      context.log.info("DeviceManager stopped")
      this
  }

}
```

我们将设备管理器的测试留给你作为练习，因为它与我们为设备组 Actor 编写的测试非常相似。

## 下一步是什么？
我们现在有了一个用于注册和跟踪设备以及记录测量值的分层组件。我们已经了解了如何实现不同类型的对话模式，例如：

- 请求响应`Request-respond`（用于温度记录）。
- 按需生成`Create-on-demand`（用于设备注册）。
- 创建监视终止`Create-watch-terminate`（用于将组和设备 Actor 创建为子级）。

在下一章中，我们将介绍组查询功能，这将建立一种新的分散收集（`scatter-gather`）对话模式。特别地，我们将实现允许用户查询属于一个组的所有设备的状态的功能。

----------

[第 5 部分：查询设备组](tutorial_5.md)

----------
**英文原文链接**：[Part 4: Working with Device Groups](https://doc.akka.io/docs/akka/current/guide/tutorial_4.html).