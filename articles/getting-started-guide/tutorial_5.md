# 第 5 部分: 查询设备组
## 简介
到目前为止，我们所看到的对话模式很简单，因为它们要求 Actor 保持很少或根本就没有状态。明确地：

- 设备 Actor 返回一个不需要状态更改的读取
- 记录温度，更新单个字段
- 设备组 Actor 通过添加或删除Map中的条目来维护组成员资格。

在本部分中，我们将使用一个更复杂的示例。由于房主会对整个家庭的温度感兴趣，我们的目标是能够查询一个组中的所有设备 Actor。让我们先研究一下这样的查询 API 应该如何工作。

## 处理可能的情况
我们面临的第一个问题是，一个组的成员是动态的。每个传感器设备都由一个可以随时停止的 Actor 表示。在查询开始时，我们可以询问所有现有设备 Actor 当前的温度。但是，在查询的生命周期中：

- 设备 Actor 可能会停止工作，无法用温度读数做出响应。
- 一个新的设备 Actor 可能会启动，并且不会包含在查询中，因为我们不知道它的存在。

这些问题可以用许多不同的方式来解决，但重要的是要确定所需的行为。以下工作对于我们的用例是很有用的：

- 当查询到达时，组 Actor 将获取现有设备 Actor 的快照（`snapshot`），并且只向这些 Actor 询问温度。
- 查询到达后启动的 Actor 将被忽略。
- 如果快照中的某个 Actor 在查询期间停止而没有应答，我们将向查询消息的发送者报告它停止的事实。

除了设备 Actor 动态地变化之外，一些 Actor 可能需要很长时间来响应。例如，它们可能被困在一个意外的无限循环中，或者由于一个 bug 而失败，并放弃我们的请求。我们不希望查询无限期地继续，因此在以下任何一种情况下，我们都会认为它已完成：

- 快照中的所有 Actor 要么已响应，要么确认已停止。
- 查询时长达到了预定的截止期限。

考虑到这些决定，再加上快照中的设备可能刚刚启动但尚未接收到要记录的温度，我们可以针对温度查询为每个设备 Actor 定义四种状态：

- 它有一个可用的温度：`Temperature`。
- 它已经响应，但还没有可用的温度：`TemperatureNotAvailable`。
- 它在响应之前已停止：`DeviceNotAvailable`。
- 它在最后期限之前没有响应：`DeviceTimedOut`。

在消息类型中总结这些，我们可以将以下内容添加到消息协议中（DeviceManager object）：

```scala
final case class RequestAllTemperatures(requestId: Long, groupId: String, replyTo: ActorRef[RespondAllTemperatures])
    extends DeviceGroupQuery.Command
    with DeviceGroup.Command
    with DeviceManager.Command

final case class RespondAllTemperatures(requestId: Long, temperatures: Map[String, TemperatureReading])

sealed trait TemperatureReading
final case class Temperature(value: Double) extends TemperatureReading
case object TemperatureNotAvailable extends TemperatureReading
case object DeviceNotAvailable extends TemperatureReading
case object DeviceTimedOut extends TemperatureReading
```
## 实现查询
实现查询的一种方法是向组设备 Actor 添加代码。然而，在实践中，这可能非常麻烦并且容易出错。请记住，当我们启动查询时，我们需要获取当前设备的快照并启动计时器，以便强制执行截止时间。同时，另一个查询可能同时到达。对于第二个查询，我们需要跟踪完全相同的信息，但与前一个查询隔离。这将要求我们在查询和设备 Actor 之间维护单独的映射。

相反，我们将实现一种更简单、更优雅的方法。我们将创建一个表示单个查询的 Actor，并代表组 Actor 执行完成查询所需的任务。到目前为止，我们已经创建了属于典型域对象（`classical domain`）的 Actor，但是现在，我们将创建一个表示流程或任务而不是实体的 Actor。我们将从让组设备 Actor 简单和能够更好地隔离检测查询的能力中受益。

### 定义查询 Actor
首先，我们需要设计查询 Actor 的生命周期。这包括识别其初始状态、将要采取的第一个操作以及清除（如果需要）。查询 Actor 需要以下信息：

- 要查询的活动设备 Actor 的快照和 ID。
- 启动查询的请求的 ID（以便我们可以在响应中包含它）。
- 发送查询的 Actor 的引用。我们会直接给这个 Actor 响应。
- 指示查询等待响应的期限。将其作为参数将简化测试。

### 设置查询超时
由于我们需要一种方法来表明我们愿意等待响应的时间，现在是时候引入一个我们还没有使用的新的 Akka 特性，即内置的调度器（`built-in scheduler`）功能了。使用`Behaviors.withTimers`和`startSingleTimer`安排将在给定延迟后发送的消息。

我们需要创建一个表示查询超时的消息。为此，我们创建了一个没有任何参数的简单消息`CollectionTimeout`。

在查询开始时，我们需要向每个设备actor询问当前温度。为了能够快速检测在收到`ReadTemperature`消息之前停止的设备，我们还将观察每个actor。这样，我们会收到`DeviceTerminated`那些在查询生命周期内停止的消息，所以我们不需要等到超时才将它们标记为不可用。

综上所述，`DeviceGroupQuery` Actor 的代码大致如下：

```scala
object DeviceGroupQuery {

  def apply(
      deviceIdToActor: Map[String, ActorRef[Device.Command]],
      requestId: Long,
      requester: ActorRef[DeviceManager.RespondAllTemperatures],
      timeout: FiniteDuration): Behavior[Command] = {
    Behaviors.setup { context =>
      Behaviors.withTimers { timers =>
        new DeviceGroupQuery(deviceIdToActor, requestId, requester, timeout, context, timers)
      }
    }
  }

  trait Command

  private case object CollectionTimeout extends Command

  final case class WrappedRespondTemperature(response: Device.RespondTemperature) extends Command

  private final case class DeviceTerminated(deviceId: String) extends Command
}

class DeviceGroupQuery(
    deviceIdToActor: Map[String, ActorRef[Device.Command]],
    requestId: Long,
    requester: ActorRef[DeviceManager.RespondAllTemperatures],
    timeout: FiniteDuration,
    context: ActorContext[DeviceGroupQuery.Command],
    timers: TimerScheduler[DeviceGroupQuery.Command])
    extends AbstractBehavior[DeviceGroupQuery.Command](context) {

  import DeviceGroupQuery._
  import DeviceManager.DeviceNotAvailable
  import DeviceManager.DeviceTimedOut
  import DeviceManager.RespondAllTemperatures
  import DeviceManager.Temperature
  import DeviceManager.TemperatureNotAvailable
  import DeviceManager.TemperatureReading

  timers.startSingleTimer(CollectionTimeout, CollectionTimeout, timeout)

  private val respondTemperatureAdapter = context.messageAdapter(WrappedRespondTemperature.apply)


  deviceIdToActor.foreach {
    case (deviceId, device) =>
      context.watchWith(device, DeviceTerminated(deviceId))
      device ! Device.ReadTemperature(0, respondTemperatureAdapter)
  }

}
```
请注意，我们必须将来自设备actor 的回复的`RespondTemperature`转换为`DeviceGroupQuery` actor理解的消息协议，即`DeviceGroupQueryMessage`。为此，我们使用`messageAdapter`将`RespondTemperature`包装到扩展自 `DeviceGroupQueryMessage` （笔误？实际是 `DeviceGroupQuery.Command`）的 `WrappedRespondTemperature`中去。

### 跟踪 Actor 状态

除了挂起的定时器之外，查询 Actor 还有一个状态方面，它跟踪一组 Actor：已回复、已停止或未回复。我们借助actor 中的一个可变 Map 变量来跟踪此状态。

对于我们的用例：

- 我们通过以下方式跟踪状态：          
  - 用一个`Map`保存已经收到的答复
  - 用一个 `Set`保存在等待中的actor

我们有三件事要做：

- 我们可以从其中一个设备接收`RespondTemperature`。
- 我们可以为同时被停止的设备 Actor 接收`Terminated`的消息。
- 我们可以达到截止时间（`deadline`）并收到一个`CollectionTimeout`消息。

为此，请将以下内容添加到您的`DeviceGroupQuery`源文件中：

```scala
private var repliesSoFar = Map.empty[String, TemperatureReading]
private var stillWaiting = deviceIdToActor.keySet

override def onMessage(msg: Command): Behavior[Command] =
  msg match {
    case WrappedRespondTemperature(response) => onRespondTemperature(response)
    case DeviceTerminated(deviceId)          => onDeviceTerminated(deviceId)
    case CollectionTimeout                   => onCollectionTimout()
  }

private def onRespondTemperature(response: Device.RespondTemperature): Behavior[Command] = {
  val reading = response.value match {
    case Some(value) => Temperature(value)
    case None        => TemperatureNotAvailable
  }

  val deviceId = response.deviceId
  repliesSoFar += (deviceId -> reading)
  stillWaiting -= deviceId

  respondWhenAllCollected()
}

private def onDeviceTerminated(deviceId: String): Behavior[Command] = {
  if (stillWaiting(deviceId)) {
    repliesSoFar += (deviceId -> DeviceNotAvailable)
    stillWaiting -= deviceId
  }
  respondWhenAllCollected()
}

private def onCollectionTimout(): Behavior[Command] = {
  repliesSoFar ++= stillWaiting.map(deviceId => deviceId -> DeviceTimedOut)
  stillWaiting = Set.empty
  respondWhenAllCollected()
}
```
我们通过依据`RespondTemperature`和`DeviceTerminated`消息来不断更新`repliesSoFar`和从`stillWaiting`中删除actor。为此，我们可以使用`DeviceTerminated`消息中已经存在的actor 设备标识符。对于我们的`RespondTemperature`消息，我们需要向`RespondTemperature`中添加 deviceId：

```scala
final case class RespondTemperature(requestId: Long, deviceId: String, value: Option[Double])
```

并修改：

```scala
case ReadTemperature(id, replyTo) =>
  replyTo ! RespondTemperature(id, deviceId, lastTemperatureReading)
  this
```

处理完每条消息后调用的方法`respondWhenAllCollected`，我们将很快讨论。

在超时的情况下，我们需要将所有尚未回复的 actor（Set 变量`stillWaiting`的成员）的状态设置为`DeviceTimedOut` 并添加到回复中去。

我们现在必须弄清楚`respondWhenAllCollected`在做什么。首先，我们需要在`repliesSoFar` Map中记录最新结果并从`stillWaiting`中将 actor 移除。然后是检查是否还剩余actor处在等待中。如果没有，我们将查询结果发送给原始请求者并停止查询参与者。否则，我们需要更新`repliesSoFar`和`stillWaiting`结构并等待更多消息。

有了所有这些认知，我们可以创建`respondWhenAllCollected`方法：

```scala
private def respondWhenAllCollected(): Behavior[Command] = {
  if (stillWaiting.isEmpty) {
    requester ! RespondAllTemperatures(requestId, repliesSoFar)
    Behaviors.stopped
  } else {
    this
  }
}
```

我们的查询actor现在完成了：

```scala
object DeviceGroupQuery {

  def apply(
      deviceIdToActor: Map[String, ActorRef[Device.Command]],
      requestId: Long,
      requester: ActorRef[DeviceManager.RespondAllTemperatures],
      timeout: FiniteDuration): Behavior[Command] = {
    Behaviors.setup { context =>
      Behaviors.withTimers { timers =>
        new DeviceGroupQuery(deviceIdToActor, requestId, requester, timeout, context, timers)
      }
    }
  }

  trait Command

  private case object CollectionTimeout extends Command

  final case class WrappedRespondTemperature(response: Device.RespondTemperature) extends Command

  private final case class DeviceTerminated(deviceId: String) extends Command
}

class DeviceGroupQuery(
    deviceIdToActor: Map[String, ActorRef[Device.Command]],
    requestId: Long,
    requester: ActorRef[DeviceManager.RespondAllTemperatures],
    timeout: FiniteDuration,
    context: ActorContext[DeviceGroupQuery.Command],
    timers: TimerScheduler[DeviceGroupQuery.Command])
    extends AbstractBehavior[DeviceGroupQuery.Command](context) {

  import DeviceGroupQuery._
  import DeviceManager.DeviceNotAvailable
  import DeviceManager.DeviceTimedOut
  import DeviceManager.RespondAllTemperatures
  import DeviceManager.Temperature
  import DeviceManager.TemperatureNotAvailable
  import DeviceManager.TemperatureReading

  timers.startSingleTimer(CollectionTimeout, CollectionTimeout, timeout)

  private val respondTemperatureAdapter = context.messageAdapter(WrappedRespondTemperature.apply)

  private var repliesSoFar = Map.empty[String, TemperatureReading]
  private var stillWaiting = deviceIdToActor.keySet


  deviceIdToActor.foreach {
    case (deviceId, device) =>
      context.watchWith(device, DeviceTerminated(deviceId))
      device ! Device.ReadTemperature(0, respondTemperatureAdapter)
  }

  override def onMessage(msg: Command): Behavior[Command] =
    msg match {
      case WrappedRespondTemperature(response) => onRespondTemperature(response)
      case DeviceTerminated(deviceId)          => onDeviceTerminated(deviceId)
      case CollectionTimeout                   => onCollectionTimout()
    }

  private def onRespondTemperature(response: Device.RespondTemperature): Behavior[Command] = {
    val reading = response.value match {
      case Some(value) => Temperature(value)
      case None        => TemperatureNotAvailable
    }

    val deviceId = response.deviceId
    repliesSoFar += (deviceId -> reading)
    stillWaiting -= deviceId

    respondWhenAllCollected()
  }

  private def onDeviceTerminated(deviceId: String): Behavior[Command] = {
    if (stillWaiting(deviceId)) {
      repliesSoFar += (deviceId -> DeviceNotAvailable)
      stillWaiting -= deviceId
    }
    respondWhenAllCollected()
  }

  private def onCollectionTimout(): Behavior[Command] = {
    repliesSoFar ++= stillWaiting.map(deviceId => deviceId -> DeviceTimedOut)
    stillWaiting = Set.empty
    respondWhenAllCollected()
  }

  private def respondWhenAllCollected(): Behavior[Command] = {
    if (stillWaiting.isEmpty) {
      requester ! RespondAllTemperatures(requestId, repliesSoFar)
      Behaviors.stopped
    } else {
      this
    }
  }
}
```

### 测试查询 Actor
现在，让我们验证查询 Actor 实现的正确性。我们需要单独测试各种场景，以确保一切都按预期工作。为了能够做到这一点，我们需要以某种方式模拟设备 Actor 来运行各种正常或故障场景。幸运的是，我们将合作者（`collaborators`）列表（实际上是一个`Map`）作为查询 Actor 的参数，这样我们就可以传入`TestProbe`引用。在我们的第一个测试中，我们在有两个设备的情况下进行测试，两个设备都报告了温度：

```scala
"return temperature value for working devices" in {
  val requester = createTestProbe[RespondAllTemperatures]()

  val device1 = createTestProbe[Command]()
  val device2 = createTestProbe[Command]()

  val deviceIdToActor = Map("device1" -> device1.ref, "device2" -> device2.ref)

  val queryActor =
    spawn(DeviceGroupQuery(deviceIdToActor, requestId = 1, requester = requester.ref, timeout = 3.seconds))

  device1.expectMessageType[Device.ReadTemperature]
  device2.expectMessageType[Device.ReadTemperature]

  queryActor ! WrappedRespondTemperature(Device.RespondTemperature(requestId = 0, "device1", Some(1.0)))
  queryActor ! WrappedRespondTemperature(Device.RespondTemperature(requestId = 0, "device2", Some(2.0)))

  requester.expectMessage(
    RespondAllTemperatures(
      requestId = 1,
      temperatures = Map("device1" -> Temperature(1.0), "device2" -> Temperature(2.0))))
}
```
这是一种乐观的例子，但我们知道有时设备不能提供温度测量。这种情况与前一种情况略有不同：

```scala
@Test
public void testReturnTemperatureNotAvailableForDevicesWithNoReadings() {
  TestKit requester = new TestKit(system);

  TestKit device1 = new TestKit(system);
  TestKit device2 = new TestKit(system);

  Map<ActorRef, String> actorToDeviceId = new HashMap<>();
  actorToDeviceId.put(device1.getRef(), "device1");
  actorToDeviceId.put(device2.getRef(), "device2");

  ActorRef queryActor = system.actorOf(DeviceGroupQuery.props(
          actorToDeviceId,
          1L,
          requester.getRef(),
          new FiniteDuration(3, TimeUnit.SECONDS)));

  assertEquals(0L, device1.expectMsgClass(Device.ReadTemperature.class).requestId);
  assertEquals(0L, device2.expectMsgClass(Device.ReadTemperature.class).requestId);

  queryActor.tell(new Device.RespondTemperature(0L, Optional.empty()), device1.getRef());
  queryActor.tell(new Device.RespondTemperature(0L, Optional.of(2.0)), device2.getRef());

  DeviceGroup.RespondAllTemperatures response = requester.expectMsgClass(DeviceGroup.RespondAllTemperatures.class);
  assertEquals(1L, response.requestId);

  Map<String, DeviceGroup.TemperatureReading> expectedTemperatures = new HashMap<>();
  expectedTemperatures.put("device1", DeviceGroup.TemperatureNotAvailable.INSTANCE);
  expectedTemperatures.put("device2", new DeviceGroup.Temperature(2.0));

  assertEquals(expectedTemperatures, response.temperatures);
}
```
我们也知道，有时设备 Actor 会在响应之前停止：

```scala
@Test
public void testReturnDeviceNotAvailableIfDeviceStopsBeforeAnswering() {
  TestKit requester = new TestKit(system);

  TestKit device1 = new TestKit(system);
  TestKit device2 = new TestKit(system);

  Map<ActorRef, String> actorToDeviceId = new HashMap<>();
  actorToDeviceId.put(device1.getRef(), "device1");
  actorToDeviceId.put(device2.getRef(), "device2");

  ActorRef queryActor = system.actorOf(DeviceGroupQuery.props(
          actorToDeviceId,
          1L,
          requester.getRef(),
          new FiniteDuration(3, TimeUnit.SECONDS)));

  assertEquals(0L, device1.expectMsgClass(Device.ReadTemperature.class).requestId);
  assertEquals(0L, device2.expectMsgClass(Device.ReadTemperature.class).requestId);

  queryActor.tell(new Device.RespondTemperature(0L, Optional.of(1.0)), device1.getRef());
  device2.getRef().tell(PoisonPill.getInstance(), ActorRef.noSender());

  DeviceGroup.RespondAllTemperatures response = requester.expectMsgClass(DeviceGroup.RespondAllTemperatures.class);
  assertEquals(1L, response.requestId);

  Map<String, DeviceGroup.TemperatureReading> expectedTemperatures = new HashMap<>();
  expectedTemperatures.put("device1", new DeviceGroup.Temperature(1.0));
  expectedTemperatures.put("device2", DeviceGroup.DeviceNotAvailable.INSTANCE);

  assertEquals(expectedTemperatures, response.temperatures);
}
```
如果你还记得，还有一个用例与设备 Actor 停止相关。我们可以从一个设备 Actor 得到一个正常的回复，但是随后接收到同一个 Actor 的一个`Terminated`消息。在这种情况下，我们希望保持第一次回复，而不是将设备标记为`DeviceNotAvailable`。我们也应该测试一下：

```scala
"return temperature reading even if device stops after answering" in {
  val requester = createTestProbe[RespondAllTemperatures]()

  val device1 = createTestProbe[Command]()
  val device2 = createTestProbe[Command]()

  val deviceIdToActor = Map("device1" -> device1.ref, "device2" -> device2.ref)

  val queryActor =
    spawn(DeviceGroupQuery(deviceIdToActor, requestId = 1, requester = requester.ref, timeout = 3.seconds))

  device1.expectMessageType[Device.ReadTemperature]
  device2.expectMessageType[Device.ReadTemperature]

  queryActor ! WrappedRespondTemperature(Device.RespondTemperature(requestId = 0, "device1", Some(1.0)))
  queryActor ! WrappedRespondTemperature(Device.RespondTemperature(requestId = 0, "device2", Some(2.0)))

  device2.stop()

  requester.expectMessage(
    RespondAllTemperatures(
      requestId = 1,
      temperatures = Map("device1" -> Temperature(1.0), "device2" -> Temperature(2.0))))
}
```
最后一种情况是，并非所有设备都能及时响应。为了保持我们的测试相对较快，我们将用较小的超时构造`DeviceGroupQuery` Actor：

```scala
"return DeviceTimedOut if device does not answer in time" in {
  val requester = createTestProbe[RespondAllTemperatures]()

  val device1 = createTestProbe[Command]()
  val device2 = createTestProbe[Command]()

  val deviceIdToActor = Map("device1" -> device1.ref, "device2" -> device2.ref)

  val queryActor =
    spawn(DeviceGroupQuery(deviceIdToActor, requestId = 1, requester = requester.ref, timeout = 200.millis))

  device1.expectMessageType[Device.ReadTemperature]
  device2.expectMessageType[Device.ReadTemperature]

  queryActor ! WrappedRespondTemperature(Device.RespondTemperature(requestId = 0, "device1", Some(1.0)))

  // no reply from device2

  requester.expectMessage(
    RespondAllTemperatures(
      requestId = 1,
      temperatures = Map("device1" -> Temperature(1.0), "device2" -> DeviceTimedOut)))
}
```
查询功能已经按预期工作了，现在是时候在`DeviceGroup` Actor 中添加这个新功能了。

## 向设备组添加查询功能

现在在设备组 Actor 中包含查询功能相当简单。我们在查询 Actor 本身中完成了所有繁重的工作，设备组 Actor 只需要使用正确的初始参数创建它，而不需要其他任何参数。

```scala
class DeviceGroup(context: ActorContext[DeviceGroup.Command], groupId: String)
    extends AbstractBehavior[DeviceGroup.Command](context) {
  import DeviceGroup._
  import DeviceManager.{
    DeviceRegistered,
    ReplyDeviceList,
    RequestAllTemperatures,
    RequestDeviceList,
    RequestTrackDevice
  }

  private var deviceIdToActor = Map.empty[String, ActorRef[Device.Command]]

  context.log.info("DeviceGroup {} started", groupId)

  override def onMessage(msg: Command): Behavior[Command] =
    msg match {
      // ... other cases omitted

      case RequestAllTemperatures(requestId, gId, replyTo) =>
        if (gId == groupId) {
          context.spawnAnonymous(
            DeviceGroupQuery(deviceIdToActor, requestId = requestId, requester = replyTo, 3.seconds))
          this
        } else
          Behaviors.unhandled
    }

  override def onSignal: PartialFunction[Signal, Behavior[Command]] = {
    case PostStop =>
      context.log.info("DeviceGroup {} stopped", groupId)
      this
  }
}
```
或许值得重述一下我们在本章开头所说的话。通过将只与查询本身相关的临时状态保留在单独的 Actor 中，我们使组 Actor 的实现非常简单。它将一切委托给子 Actor，因此不必保留与核心业务无关的状态。此外，多个查询现在可以彼此并行运行，事实上，可以根据需要运行任意多个查询。在我们的例子中，查询单个设备 Actor 是一种快速操作，但是如果不是这样，例如，因为需要通过网络联系远程传感器，这种设计将显著提高吞吐量。

我们通过测试所有的功能一起工作来结束这一章。此测试是前一个测试的变体，现在使用组查询功能：

```scala
"be able to collect temperatures from all active devices" in {
  val registeredProbe = createTestProbe[DeviceRegistered]()
  val groupActor = spawn(DeviceGroup("group"))

  groupActor ! RequestTrackDevice("group", "device1", registeredProbe.ref)
  val deviceActor1 = registeredProbe.receiveMessage().device

  groupActor ! RequestTrackDevice("group", "device2", registeredProbe.ref)
  val deviceActor2 = registeredProbe.receiveMessage().device

  groupActor ! RequestTrackDevice("group", "device3", registeredProbe.ref)
  registeredProbe.receiveMessage()

  // Check that the device actors are working
  val recordProbe = createTestProbe[TemperatureRecorded]()
  deviceActor1 ! RecordTemperature(requestId = 0, 1.0, recordProbe.ref)
  recordProbe.expectMessage(TemperatureRecorded(requestId = 0))
  deviceActor2 ! RecordTemperature(requestId = 1, 2.0, recordProbe.ref)
  recordProbe.expectMessage(TemperatureRecorded(requestId = 1))
  // No temperature for device3

  val allTempProbe = createTestProbe[RespondAllTemperatures]()
  groupActor ! RequestAllTemperatures(requestId = 0, groupId = "group", allTempProbe.ref)
  allTempProbe.expectMessage(
    RespondAllTemperatures(
      requestId = 0,
      temperatures =
        Map("device1" -> Temperature(1.0), "device2" -> Temperature(2.0), "device3" -> TemperatureNotAvailable)))
}
```
## 总结
在物联网（`IoT`）系统的背景下，本指南介绍了以下概念。如有必要，你可以通过以下链接进行查看：

- [Actor 的层级结构及其生命周期](tutorial_1.md)
- [灵活性设计消息的重要性](tutorial_3.md)
- [如何监视和停止 Actor](tutorial_4.md)

## 下一步是什么？
要继续你的 Akka 之旅，我们建议：

- 开始用 Akka 构建你自己的应用程序，如果你陷入困境的话，希望你能参与到我们的「[社区](https://akka.io/get-involved/)」中，寻求帮助。
- 如果你想了解更多的背景知识，请阅读[参考文件](actor-intro.md)的其余部分，并查看一些关于 Akka 的「[书籍和视频](https://doc.akka.io/docs/akka/current/additional/books.html)」。

要从本指南获得完整的应用程序，你可能需要提供 UI 或 API。为此，我们建议你查看以下技术，看看哪些适合你：

- [Microservices with Akka 教程](https://developer.lightbend.com/docs/akka-platform-guide/microservices-tutorial/)说明了如何使用 Akka Persistence 和 Akka Projections 实现事件源 CQRS 应用程序。
- 「[Akka HTTP](https://doc.akka.io/docs/akka-http/current/introduction.html)」是一个 HTTP 服务和客户端的库，使发布和使用 HTTP 端点（`endpoints`）成为可能。
- 「[Play Framework](https://www.playframework.com/)」是一个成熟的 Web 框架，它构建在 Akka HTTP 之上，它能与 Akka 很好地集成，可用于创建一个完整的现代 Web 用户界面。
- 「[Lagom](https://www.lagomframework.com/)」是一个基于 Akka 的独立的微服务框架，它编码了 Akka 和 Play 的许多最佳实践。

----------

[一般概念](../general-concepts/terminology.md) 

----------
**英文原文链接**：[Part 5: Querying Device Groups](https://doc.akka.io/docs/akka/current/guide/tutorial_5.html).