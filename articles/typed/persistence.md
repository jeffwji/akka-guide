# 事件溯源

您正在查看新 actor API 的文档，要查看 Akka Classic 文档，请参阅[Classic Akka Persistence](https://doc-akka-io.translate.goog/docs/akka/current/persistence.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)。

## 模块信息

要使用 Akka Persistence，请将模块添加到您的项目中：

- [          sbt         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​         `val AkkaVersion = "2.6.15" libraryDependencies ++= Seq(  "com.typesafe.akka" %% "akka-persistence-typed" % AkkaVersion,  "com.typesafe.akka" %% "akka-persistence-testkit" % AkkaVersion % Test )`        

- [          马文         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

- [          摇篮         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab2)

您还必须选择日志插件和可选的快照存储插件，请参阅[持久性插件](https://doc-akka-io.translate.goog/docs/akka/current/persistence-plugins.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)。

[[+\] 显示项目信息](https://doc.akka.io/docs/akka/current/typed/persistence.html#)

## 介绍

Akka Persistence 使有状态的 Actor 能够持久化它们的状态，以便在 Actor 重新启动时（例如在 JVM 崩溃后、由主管或手动停止启动或在集群内迁移时）可以恢复状态。 Akka Persistence 背后的关键概念是只存储由actor 持久化的*事件*，而不是actor 的实际状态（尽管actor  状态快照支持也可用）。事件通过附加到存储（没有任何变化）来持久化，这允许非常高的事务率和高效的复制。通过将存储的事件重播给 Actor  来恢复有状态的 Actor，允许它重建其状态。这可以是更改的完整历史记录，也可以是从快照中的检查点开始，这可以显着减少恢复时间。

该[事件与采购2.6阿卡视频](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://akka.io/blog/news/2020/01/07/akka-event-sourcing-video)是学习活动采购，与一起一个良好的起点[与阿卡教程微服务](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://developer.lightbend.com/docs/akka-platform-guide/microservices-tutorial/)，说明如何实现与阿卡持久性和阿卡预测事件源CQRS应用。

​         笔记        

通用数据保护条例 (GDPR) 要求必须根据用户的要求删除个人信息。删除或修改带有个人信息的事件会很困难。数据粉碎可用于忘记信息，而不是删除或修改信息。这是通过使用给定数据主体 ID（人）的密钥加密数据并在忘记该数据主体时删除该密钥来实现的。Lightbend 的[Akka Persistence](https://doc-akka-io.translate.goog/docs/akka-enhancements/current/gdpr/index.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)的[GDPR](https://doc-akka-io.translate.goog/docs/akka-enhancements/current/gdpr/index.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)提供了工具来促进构建具有 GDPR 能力的系统。

### 事件溯源的概念

请参阅MSDN[上的事件溯源简介](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://docs.microsoft.com/en-us/previous-versions/msp-n-p/jj591559%28v%3Dpandp.10%29)。

另一篇关于“在事件中思考”的优秀文章是Randy Shoup 的[Events As First-Class Citizens](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://hackernoon.com/events-as-first-class-citizens-8633e8479493)。如果您开始开发基于事件的应用程序，这是一本简短的推荐读物。

接下来是 Akka 通过事件源 actor 的实现。 

事件源参与者（也称为持久参与者）接收（非持久）命令，如果它可以应用于当前状态，则首先验证该命令。例如，这里的验证可以意味着任何事情，从简单检查命令消息的字段到与多个外部服务的对话。如果验证成功，则从命令生成事件，表示命令的效果。然后这些事件被持久化，并在成功持久化后用于改变参与者的状态。当需要恢复事件源actor时，只重放我们知道它们可以成功应用的持久事件。换句话说，与命令相反，事件在重播给持久参与者时不会失败。事件源参与者还可以处理不改变应用程序状态的命令，例如查询命令。

## 示例和核心 API

让我们从一个简单的例子开始。a 的最低要求是：[`EventSourcedBehavior`](https://doc-akka-io.translate.goog/api/akka/2.6/akka/persistence/typed/scaladsl/EventSourcedBehavior.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

要注意的第一件重要的事情是将`Behavior`持久性actor 的类型输入为 的类型，`Command`因为这是持久性actor 应该接收的消息类型。在 Akka 中，这现在由类型系统强制执行。

组成一个的组件`EventSourcedBehavior`是：

- `persistenceId` 是持久参与者的稳定唯一标识符。
- `emptyState`定义`State`第一次创建实体的时间，例如计数器将以 0 作为状态开始。
- `commandHandler` 定义如何通过产生效果来处理命令，例如持久化事件、停止持久化actor。
- `eventHandler` 当事件被持久化时，返回给定当前状态的新状态。

接下来，我们将详细讨论每一个。

### 持久性 ID

该是在后端事件日志和快照存储持久演员稳定的唯一标识符。[`PersistenceId`](https://doc-akka-io.translate.goog/api/akka/2.6/akka/persistence/typed/PersistenceId.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)

[集群分片](https://doc-akka-io.translate.goog/docs/akka/current/typed/cluster-sharding.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)通常与 一起使用，`EventSourcedBehavior`以确保每个`PersistenceId`( `entityId`)只有一个活动实体。

该`entityId`集群拆分是实体的业务域标识。的`entityId`可能不是唯一的，足以用作`PersistenceId`本身。例如，两种不同类型的实体可能具有相同的`entityId`. 要创建唯一`PersistenceId`的 ，`entityId`应该以实体类型的稳定名称作为前缀，该名称通常与`EntityTypeKey.name`集群分片中使用的 相同。有工厂方法可以帮助从and构造这样的方法。`PersistenceId.apply``PersistenceId``entityTypeHint``entityId`

连接`entityTypeHint`and时的默认分隔符`entityId`是`|`，但支持自定义分隔符。

​         笔记        

该`|`分离器还用于Lagom的`scaladsl.PersistentEntity`，但没有分隔在Lagom的使用`javadsl.PersistentEntity`。为了与 Lagom 兼容，`javadsl.PersistentEntity`您应该将其`""`用作分隔符。

所述[集群拆分文档中的持久性例子](https://doc-akka-io.translate.goog/docs/akka/current/typed/cluster-sharding.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#persistence-example)示出了如何构造`PersistenceId`从`entityTypeKey`与`entityId`通过所提供的`EntityContext`。

可以使用 来创建自定义标识符`PersistenceId.ofUniqueId`。

### 命令处理程序

命令处理程序是一个有 2 个参数的函数，当前`State`和传入的`Command`。

命令处理程序返回一个`Effect`指令，该指令定义要持久化的事件（如果有）。效果是使用工厂创建的。 `Effect`

两个最常用的效果是： 

- `persist` 将原子地持久化一个或多个事件，即如果出现错误，则存储所有事件或不存储任何事件
- `none` 没有事件将被持久化，例如只读命令

更多效果在[效果和](https://doc-akka-io.translate.goog/docs/akka/current/typed/persistence.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#effects-and-side-effects)副作用中进行了解释。

除了返回`Effect`命令`EventSourcedBehavior`的主函数之外，s 还可以链接在成功持久化后执行的副作用，这是通过`thenRun`函数实现的，例如.`Effect.persist(..).thenRun`

### 事件处理程序

当一个事件被成功持久化时，新状态是通过将事件应用到当前状态来创建的`eventHandler`。在多个持久事件的情况下，`eventHandler`每个事件都按照传递给 的相同顺序调用。`Effect.persist(..)`

状态通常定义为不可变类，然后事件处理程序返回状态的新实例。您可以选择对状态使用可变类，然后事件处理程序可能会更新状态实例并返回相同的实例。支持不可变和可变状态。

当实体启动以从存储的事件中恢复其状态时，也使用相同的事件处理程序。

事件处理程序必须只更新状态并且永远不会执行副作用，因为这些也会在持久性 Actor 的恢复期间执行。副作用应该在持久化事件后或从after [Recovery](https://doc-akka-io.translate.goog/docs/akka/current/typed/persistence.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#recovery)之后`thenRun`从[命令处理程序中](https://doc-akka-io.translate.goog/docs/akka/current/typed/persistence.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#command-handler)执行。`RecoveryCompleted`

### 完成示例

让我们填写示例的详细信息。

命令和事件：

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

State 是一个包含 5 个最新项目的列表：

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

命令处理程序将`Add`有效负载保留在一个`Added`事件中：

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

事件处理程序将项目附加到状态并保留 5 个项目。这在成功将事件持久化到数据库中后调用：

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

这些用于创建一个`EventSourcedBehavior`： 

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

## 效果和副作用

命令处理程序返回一个`Effect`指令，该指令定义要持久化的事件（如果有）。效果是使用工厂创建的，可以是以下之一： `Effect`

- `persist` 将原子地持久化一个或多个事件，即如果出现错误，则存储所有事件或不存储任何事件
- `none` 没有事件将被持久化，例如只读命令
- `unhandled` 该命令在当前状态下未处理（不支持）
- `stop` 阻止这个演员
- `stash` 当前命令被隐藏
- `unstashAll` 处理隐藏的命令 `Effect.stash`
- `reply` 向给定的人发送回复消息 `ActorRef`

请注意，每个传入命令只能选择其中之一。不可能既坚持又说没有/未处理。

除了返回`Effect`命令`EventSourcedBehavior`的主函数之外，s 还可以链接在成功持久化后执行的副作用，这是通过`thenRun`函数实现的，例如.`Effect.persist(..).thenRun`

在下面的示例中，状态被发送到`subscriber`ActorRef。请注意，应用事件后的新状态作为`thenRun`函数的参数传递。在多个事件已经被持久化的情况下，传递给的状态`thenRun`是所有事件都处理完后的更新状态。

所有`thenRun`注册的回调都在成功执行持久语句后顺序执行（或立即执行，在`none`和 的情况下`unhandled`）。

此外`thenRun`，也可以成功后进行以下动作坚持：

- `thenStop` 演员将被阻止
- `thenUnstashAll` 处理隐藏的命令 `Effect.stash`
- `thenReply` 向给定的人发送回复消息 `ActorRef`

效果示例：

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

大多数情况下，这将使用上述`thenRun`方法完成`Effect`。您可以将常见的副作用分解为函数并重用于多个命令。例如：

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

### 副作用排序和保证

任何副作用都最多执行一次，如果持久化失败，则不会执行。

当 actor 重新启动或停止后再次启动时，不会运行副作用。您可以在接收`RecoveryCompleted`信号时检查状态并执行当时尚未确认的副作用。这可能会导致多次执行副作用。

副作用是顺序执行的，不可能并行执行副作用，除非它们调用并发运行的东西（例如向另一个参与者发送消息）。

可以在持久化事件之前执行副作用，但这可能导致执行副作用但如果持久化失败则不会存储事件。

### 原子写

通过将`persist`效果与事件列表一起使用，可以原子地存储多个事件。这意味着传递给该方法的所有事件都会被存储，或者如果出现错误，则不存储任何事件。

因此，持久actor 的恢复永远不会部分完成，而只有单个`persist`效果持久化的事件子集。

一些期刊可能不支持多个事件的原子写入，然后他们会拒绝`persist`多个事件。这会`EventSourcedBehavior`通过一个`EventRejectedException`（通常是一个`UnsupportedOperationException`）通知给一个，并且可以通过一个[supervisor](https://doc-akka-io.translate.goog/docs/akka/current/typed/fault-tolerance.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)来处理。

## 集群分片和 EventSourcedBehavior

在需要的持久参与者数量高于一个节点的内存或弹性很重要的用例中，如果节点崩溃，持久参与者会在新节点上快速启动并可以恢复操作[集群分片](https://doc-akka-io.translate.goog/docs/akka/current/typed/cluster-sharding.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)非常适合将持久参与者分布在集群上并通过 id 寻址它们。

所述`EventSourcedBehavior`然后可以运行与任何普通的演员中所述[行动者的文档](https://doc-akka-io.translate.goog/docs/akka/current/typed/actors.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)，但由于阿卡持久性是基于单写原理持久演员通常与集群拆分一起使用。对于特定的`persistenceId`情况，一次应该只有一个持久的 actor 实例处于活动状态。如果多个实例同时保留事件，则这些事件将交错，并且在重放时可能无法正确解释。Cluster Sharding 确保每个 id 只有一个活动实体。所述[集群拆分例子](https://doc-akka-io.translate.goog/docs/akka/current/typed/cluster-sharding.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#persistence-example)示出了该常见的组合。

## 访问 ActorContext

如果需要使用，例如生成子actor，可以通过用 包装构造来获得：[`EventSourcedBehavior`](https://doc-akka-io.translate.goog/api/akka/2.6/akka/persistence/typed/scaladsl/EventSourcedBehavior.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)[`ActorContext`](https://doc-akka-io.translate.goog/api/akka/2.6/akka/actor/typed/scaladsl/ActorContext.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)`Behaviors.setup`

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

## 改变行为

处理完一条消息后，actor 能够返回`Behavior`用于下一条消息的 。

正如你在上面的例子中看到的那样，持久性演员不支持。相反，状态由 返回`eventHandler`。无法返回新行为的原因是行为是参与者状态的一部分，并且在恢复期间也必须仔细重建。如果它被支持，则意味着在重放事件时必须恢复行为，并且在使用快照时无论如何也要在状态中进行编码。这很容易出错，因此在 Akka Persistence 中是不允许的。

对于基本角色，您可以使用相同的命令处理程序集，而不管实体处于什么状态，如上例所示。对于更复杂的参与者，能够改变行为是有用的，因为可以根据参与者所处的状态定义用于处理命令的不同功能。这在实现类似实体的有限状态机 (FSM) 时很有用。

下一个示例演示如何基于当前`State`. 它显示了一个代表博客文章状态的演员。在帖子开始之前，它可以处理的唯一命令是`AddPost`. 一旦启动，就可以使用 查找`GetPost`、修改`ChangeBody`或发布它`Publish`。

状态被捕获：

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

命令，其中只有一个子集有效，具体取决于状态：

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

 通过首先查看状态然后查看命令来决定处理每个命令的命令处理程序。它通常变成两个级别的模式匹配，首先是状态，然后是命令。委托给方法是一种很好的做法，因为单行案例可以很好地概述消息分发。

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

事件处理程序：

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

最后，行为是从以下`EventSourcedBehavior.apply`创建的：

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

这可以通过定义如所示的状态类的事件和命令处理程序的一个或两个步骤进一步采取[的事件处理程序的状态](https://doc-akka-io.translate.goog/docs/akka/current/typed/persistence-style.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#event-handlers-in-the-state)和[在状态命令处理程序](https://doc-akka-io.translate.goog/docs/akka/current/typed/persistence-style.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#command-handlers-in-the-state)。

还有一个示例说明了[可选的初始状态](https://doc-akka-io.translate.goog/docs/akka/current/typed/persistence-style.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#optional-initial-state)。

## 回复

该[请求-响应交互模式](https://doc-akka-io.translate.goog/docs/akka/current/typed/interaction-patterns.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#request-response)是持久的演员很常见的，因为你通常想知道，如果命令被拒绝，由于验证错误，并接受当你想要一个确认的事件已成功储存时。

因此，您通常包含一个. 如果命令可以返回成功响应或验证错误，则可以使用通用响应类型。如果成功回复不包含值但更多的是确认，则可以使用预定义的类型。`ActorRef[ReplyMessageType]``StatusReply[ReplyType]]` `StatusReply.Ack``StatusReply[Done]`

在验证错误或持久化事件之后，使用`thenRun`副作用，可以将回复消息发送到`ActorRef`.

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

由于这是一个如此常见的模式，因此具有用于此目的的回复效果。它有一个很好的属性，它可以用来强制在实现`EventSourcedBehavior`. 如果定义为 with ，则返回的效果不是 a 会出现编译错误，可以用, , , 或来创建。`EventSourcedBehavior.withEnforcedReplies``ReplyEffect``Effect.reply``Effect.noReply``Effect.thenReply``Effect.thenNoReply`

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

命令必须有一个字段，然后可用于发送回复。`ActorRef[ReplyMessageType]`

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

在`ReplyEffect`与创建，，，或。`Effect.reply``Effect.noReply``Effect.thenReply``Effect.thenNoReply`



- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

即使不使用这些效果也会发送回复消息，但如果忽略回复决定，则不会有编译错误。`EventSourcedBehavior.withEnforcedReplies`

请注意，这`noReply`是一种有意识地决定不应该为特定命令发送回复或稍后发送回复的方式，可能在与其他参与者或服务进行一些异步交互之后。

## 序列化

与参与者消息相同的[序列化](https://doc-akka-io.translate.goog/docs/akka/current/serialization.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)机制也用于持久参与者。在为事件选择序列化解决方案时，您还应该考虑在应用程序发展时必须可以读取旧事件。可以在[模式演变中](https://doc-akka-io.translate.goog/docs/akka/current/persistence-schema-evolution.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)找到相应的策略。

您需要为您的命令（消息）、事件和状态（快照）启用[序列化](https://doc-akka-io.translate.goog/docs/akka/current/serialization.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)。在许多情况下，[使用 Jackson 进行序列化](https://doc-akka-io.translate.goog/docs/akka/current/serialization-jackson.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)是一个不错的选择，如果您没有其他偏好，我们建议您这样做。

## 恢复

通过重播日志事件，在启动和重新启动时自动恢复事件源actor。在恢复期间发送给参与者的新消息不会干扰重播事件。`EventSourcedBehavior`在恢复阶段完成后，它们被藏起来并被接收。

可以同时进行的并发恢复数量限制为不会使系统和后端数据存储过载。当超过限制时，actor 将等待其他恢复完成。这是通过以下方式配置的：

```
akka.persistence.max-concurrent-recoveries = 50
```

该[事件处理程序](https://doc-akka-io.translate.goog/docs/akka/current/typed/persistence.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#event-handler)用于重放日志事件时更新状态。

强烈不鼓励在事件处理程序中执行副作用，因此在恢复完成后应执行副作用作为对处理程序中`RecoveryCompleted`信号的反应`receiveSignal` 

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

该`RecoveryCompleted`包含当前`State`。

演员总是会收到一个`RecoveryCompleted`信号，即使没有活动在杂志和快照存储为空，或者如果它是一个以前未使用的一个新的持久的演员`PersistenceId`。

[快照](https://doc-akka-io.translate.goog/docs/akka/current/typed/persistence-snapshot.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)可用于优化恢复时间。

### 重播过滤器

可能存在事件流被破坏并且多个作者（即多个持久参与者实例）使用相同序列号记录不同消息的情况。在这种情况下，您可以配置在恢复时如何过滤来自多个作者的重播消息。

在您的配置中，在`akka.persistence.journal.xxx.replay-filter`部分（`xxx`您的日志插件 ID在哪里）下，您可以`mode`从以下值之一中选择重播过滤器：

- 废旧修复
- 失败
- 警告
- 离开

例如，如果您为 leveldb 插件配置重播过滤器，它看起来像这样：

```
# The replay filter can detect a corrupt event stream by inspecting
# sequence numbers and writerUuid when replaying events.
akka.persistence.journal.leveldb.replay-filter {
  # What the filter should do when detecting invalid events.
  # Supported values:
  # `repair-by-discard-old` : discard events from old writers,
  #                           warning is logged
  # `fail` : fail the replay, error is logged
  # `warn` : log warning but emit events untouched
  # `off` : disable this feature completely
  mode = repair-by-discard-old
}
```

### 禁用恢复

您还可以完全禁用事件和快照的恢复：

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

请参考[快照](https://doc-akka-io.translate.goog/docs/akka/current/typed/persistence-snapshot.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#snapshots)，如果你需要禁用仅快照恢复，或者您需要选择特定的快照。

在任何情况下，最高序列号将始终被恢复，因此您可以在不破坏事件日志的情况下保留新事件。

## 标记

持久性允许您在不使用 的情况下使用事件标签[`EventAdapter`](https://doc-akka-io.translate.goog/docs/akka/current/persistence.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#event-adapters)：

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

## 事件适配器

事件适配器可以以编程方式添加到您的`EventSourcedBehavior`s 中，可以将您的`Event`类型转换为另一种类型，然后传递给日志。

定义一个事件适配器是通过扩展一个 EventAdapter 来完成的：

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

然后将其安装在`EventSourcedBehavior`：

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

## 包装 EventSourcedBehavior

创建 时`EventSourcedBehavior`，可以包装`EventSourcedBehavior`其他行为，例如`Behaviors.setup`为了访问`ActorContext`对象。例如，为了调试目的，在拍摄快照时访问参与者日志。

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

## 日志失败

默认情况下，`EventSourcedBehavior`如果从日志中抛出异常，则将停止。可以用 any 覆盖它`BackoffSupervisorStrategy`。不能为此使用正常的监督包装，因为它对`resume`日志失败的行为无效，因为不知道事件是否持续存在。

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

如果从日志中恢复 Actor 的状态出现问题，`RecoveryFailed`则会向`receiveSignal`处理程序 发出一个信号，并且 Actor 将停止（或通过退避重新启动）。

### 期刊拒绝

期刊可以拒绝事件。与失败的区别在于日志必须在尝试持久化事件之前决定拒绝它，例如因为序列化异常。如果一个事件被拒绝，它肯定不会出现在日志中。这会`EventSourcedBehavior`通过一个信号发送给一个via an`EventRejectedException`并且可以由一个[supervisor](https://doc-akka-io.translate.goog/docs/akka/current/typed/fault-tolerance.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)处理。并非所有日志实现都使用拒绝并将此类问题也视为日志失败。

## 藏

当使用`persist`或持久化事件时`persistAll`，可以保证`EventSourcedBehavior`在确认事件被持久化并运行其他副作用之前不会接收进一步的命令。传入的消息会自动隐藏，直到`persist`完成。

命令也在恢复期间被隐藏，不会干扰重播事件。恢复完成后将接收命令。

上面描述的隐藏是自动处理的，但也有可能在接收到命令时隐藏命令以将它们的处理推迟到以后。一个示例可能是在处理其他命令之前等待某些外部条件或交互完成。这是通过返回一个`stash`effect 并稍后使用来完成的`thenUnstashAll`。

让我们用一个任务管理器的例子来说明如何使用隐藏效果。它处理三个命令；`StartTask`，`NextStep`和`EndTask`。这些命令与给定的命令相关联，`taskId`管理器一次处理一个`taskId`。任务在接收`StartTask`时启动，在接收`NextStep`命令时继续，直到接收到最终`EndTask`。与`taskId`正在进行的命令不同的命令通过隐藏它们来推迟。当`EndTask`被处理的新任务可以启动和藏匿的命令进行处理。

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/typed/persistence.html#tab1)

你应该小心，不要向持久参与者发送超过它可以跟上的消息，否则存储缓冲区将填满，当达到其最大容量时，命令将被丢弃。容量可以配置为：

```
akka.persistence.typed.stash-capacity = 10000
```

请注意，隐藏的命令保存在内存缓冲区中，因此如果发生崩溃，它们将不会被处理。

- 如果参与者（实体）被集群分片钝化或重新平衡，则将丢弃隐藏的命令。
- 如果 actor 在处理命令或持久化后的副作用时由于抛出异常而重新启动（或停止），则将丢弃隐藏的命令。
- 如果在存储事件时发生故障，但仅当`onPersistFailure`定义了退避监控器策略时，存储的命令会被保留和处理。

允许在取消隐藏时隐藏消息。那些新添加的命令不会被`unstashAll`正在进行的效果处理，必须由另一个`unstashAll`.

## 向外扩展

在需要的持久参与者数量高于一个节点的内存或弹性很重要的用例中，如果节点崩溃，持久参与者会在新节点上快速启动并可以恢复操作[集群分片](https://doc-akka-io.translate.goog/docs/akka/current/typed/cluster-sharding.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)非常适合将持久参与者分布在集群上并通过 id 寻址它们。

Akka Persistence 基于单写入器原则。对于特定的情况，一次应该`PersistenceId`只有一个`EventSourcedBehavior`实例处于活动状态。如果多个实例同时保留事件，则这些事件将交错，并且在重放时可能无法正确解释。Cluster Sharding 确保`EventSourcedBehavior`数据中心内的每个 id只有一个活动实体 ( )。[复制事件溯源](https://doc-akka-io.translate.goog/docs/akka/current/typed/replicated-eventsourcing.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)支持跨数据中心的主动-主动持久实体。

## 配置

持久化模块有几个配置属性，请参考[参考配置](https://doc-akka-io.translate.goog/docs/akka/current/general/configuration-reference.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#config-akka-persistence)。

该[轴颈和快照存储插件](https://doc-akka-io.translate.goog/docs/akka/current/persistence-plugins.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)具有特定的配置，请参见所选择的插件的参考文档。

## 示例项目

 [持久化示例项目](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://developer.lightbend.com/start/?group%3Dakka%26project%3Dakka-samples-persistence-scala)是一个可以下载的示例项目，并附有如何运行的说明。该项目包含一个购物车示例，说明如何使用 Akka Persistence。

在[使用 Akka 的微服务教程中](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://developer.lightbend.com/docs/akka-platform-guide/microservices-tutorial/)进一步扩展了购物车示例。在该示例中，事件被标记为由处理器使用以从事件构建其他表示，或将事件发布到其他服务。

 [多 DC 持久性示例项目](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://developer.lightbend.com/start/?group%3Dakka%26project%3Dakka-samples-persistence-dc-scala)说明了如何使用支持跨数据中心的主动-主动持久性实体的[复制事件溯源](https://doc-akka-io.translate.goog/docs/akka/current/typed/replicated-eventsourcing.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)。

​          [ 坚持](https://doc-akka-io.translate.goog/docs/akka/current/typed/index-persistence.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)         

​          [复制事件溯源 ](https://doc-akka-io.translate.goog/docs/akka/current/typed/replicated-eventsourcing.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)         