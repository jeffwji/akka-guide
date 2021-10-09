# 集群使用

本文档介绍了如何使用 Akka Cluster 和 Cluster API。The [有状态或无状态应用: To Akka Cluster or not](https://akka.io/blog/news/2020/06/01/akka-cluster-motivation) 视频是了解使用 Akka Cluster 动机的一个很好的起点。

有关特定文档主题，请参阅： 

- [何时何地使用 Akka 集群](choosing-cluster.md)
- [集群规范](cluster-concepts.md)
- [集群成员服务](cluster-membership.md)
- [高级别的集群工具](cluster.md#高级别的集群工具)
- [滚动更新](rolling-updates.md)
- [运营、管理、可观察性](operations.md)

您正在查看新 actor API 的文档，要查看 Akka Classic 文档，请参阅[Classic Cluster](https://doc.akka.io/docs/akka/current/cluster-usage.html)。

您必须启用[序列化](serialization.md)才能在集群中的 ActorSystems（节点）之间发送消息。在许多情况下，[使用 Jackson 进行序列化](serialization-jackson.md)是一个不错的选择，如果您没有其他偏好或限制，我们建议您这样做。

## 模块信息

要使用 Akka Cluster，在您的项目中添加以下依赖项：

```sbt
val AkkaVersion = "2.6.16"
libraryDependencies += "com.typesafe.akka" %% "akka-cluster-typed" % AkkaVersion
```

| Project Info: Akka Cluster (typed) |                                                              |
| ---------------------------------- | ------------------------------------------------------------ |
| Artifact                           | com.typesafe.akka  akka-cluster-typed  2.6.16  [Snapshots are available](https://doc.akka.io/docs/akka/current/typed/project/links.html#snapshots-repository) |
| JDK versions                       | Adopt OpenJDK 8Adopt OpenJDK 11                              |
| Scala versions                     | 2.12.14, 2.13.6                                              |
| JPMS module name                   | akka.cluster.typed                                           |
| License                            | [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0.html) |
| Readiness level                    | [Supported](https://developer.lightbend.com/docs/introduction/getting-help/support-terminology.html#supported), [Lightbend Subscription](https://www.lightbend.com/lightbend-subscription) provides support  Since 2.6.0, 2019-11-06 |
| Home page                          | https://akka.io/                                             |
| API documentation                  | [API (Scaladoc)](https://doc.akka.io/api/akka/2.6.16/akka/cluster/typed/index.html)  [API (Javadoc)](https://doc.akka.io/japi/akka/2.6.16/akka/cluster/typed/package-summary.html) |
| Forums                             | [Lightbend Discuss](https://discuss.akka.io)  [akka/akka Gitter channel](https://gitter.im/akka/akka) |
| Release notes                      | [akka.io blog](https://akka.io/blog/news-archive.html)       |
| Issues                             | [Github issues](https://github.com/akka/akka/issues)         |
| Sources                            | https://github.com/akka/akka                                 |

## Cluster API 扩展

Cluster 扩展为您提供了管理集群任务的能力，例如 [Joining](cluster-membership.md#user-actions)、[Leaving](cluster-membership.md#user-actions) 和 [Downing](cluster-membership.md#user-actions)，以及订阅集群成员事件的能力，例如[MemberUp](cluster-membership.md#membership-lifecycle)、[MemberRemoved](cluster-membership.md#membership-lifecycle) 和 [UnreachableMember](cluster-membership.md#membership-lifecycle)，这些功能里以事件 API 的形式公开。

`Cluster`扩展中提供了以下引用来实现这些功能：

- `manager`: 是一个 `ActorRef[akka.cluster.typed.ClusterCommand]` 类型的引用，它的 `ClusterCommand` 是一个命令，例如：`Join`，`Leave` 或`Down`。
- `subscriptions`: 是一个`ActorRef[akka.cluster.typed.ClusterStateSubscription]`类型的引用，它的`ClusterStateSubscription`是一个对某个集群事件，比如`MemberRemoved` 的`GetCurrentState` 或 `Subscribe` 或 `Unsubscribe`，进行注册的事件。
- `state`: 当前的 `CurrentClusterState`

以下所有示例均假设以下导入：

```scala
import akka.actor.typed._
import akka.actor.typed.scaladsl._
import akka.cluster.ClusterEvent._
import akka.cluster.MemberStatus
import akka.cluster.typed._
```

在节点上获取 cluster：

```scaka
val cluster = Cluster(system)
```

 笔记

```
当启动 ActorSystem 的时候，所有传递给 cluster 成员的 ActorSystem 名称必须相同。
```

### 加入和离开集群

如果不使用配置来指定[要加入的种子节点](cluster.md#joining)，则可以通过程序来使用`manager`来实现。

```scala
cluster.manager ! Join(cluster.selfMember.address)
```

[离开](cluster.md#leaving)集群和[停止](cluster.md#停机)节点是类似的：

```scala
cluster2.manager ! Leave(cluster2.selfMember.address)
```

### 集群订阅

集群的`subscriptions`函数可用于在集群状态发生变化时接收消息。例如，注册所有`MemberEvent`事件，然后使用 `manager` 注销一个节点时将导致节点产生 [成员生命周期](cluster-membership.md#membership-lifecycle) 中的某些事件。

以下例子订阅`subscriber: ActorRef[MemberEvent]` 事件

```scala
cluster.subscriptions ! Subscribe(subscriber, classOf[MemberEvent])
```

然后要求一个节点离开：

```scala
cluster.manager ! Leave(anotherMemberAddress)
// subscriber will receive events MemberLeft, MemberExited and MemberRemoved
```

### 集群状态

与订阅集群事件不同，有时可以使用 `Cluster(system).state` 来方便地只获得全部成员的状态。请注意，此状态不一定与发布到集群订阅的事件同步。

有关成员事件的更多信息，请参阅[集群成员](cluster-membership.md#成员事件)。更多的变更事件的类型，请参阅`akka.cluster.ClusterEvent.ClusterDomainEvent`扩展类的 API 文档以获取有关事件的详细信息。

## 集群成员 API

### Joining

种子节点是加入集群的初始联系点，可以通过不同的方式完成：

- [Cluster Bootstrap 时自动启动](cluster.md#使用-Cluster-Bootstrap-自动加入种子节点)
- [通过配置文件加入种子节点](cluster.md#通过配置文件加入种子节点)
- [以编程方式](cluster.md#以编程方式加入种子节点)

在完成加入过程之后，它们便以与其他节点完全相同的方式参与集群。

#### 使用 Cluster Bootstrap 自动加入种子节点

使用开源 Akka 管理项目的模块[Cluster Bootstrap](operations.md#集群引导)可以在加入过程中自动发现节点。有关更多详细信息，请参阅其文档。

#### 通过配置文件加入种子节点

当一个新节点启动时，它会向所有种子节点发送一条消息，然后向第一个回答的节点发送加入命令。如果没有种子节点回复（可能尚未启动），它会重试此过程，直到成功或停止。

您可以在[配置文件](cluster.md#configuration)（application.conf）中定义种子节点：

```
akka.cluster.seed-nodes = [
  "akka://ClusterSystem@host1:2552",
  "akka://ClusterSystem@host2:2552"]
```

这也可以使用以下语法在启动 JVM 时定义为 JVM 系统属性：

```
-Dakka.cluster.seed-nodes.0=akka://ClusterSystem@host1:2552
-Dakka.cluster.seed-nodes.1=akka://ClusterSystem@host2:2552
```

当一个新节点启动时，它会向所有已配置的`seed-nodes`发送一条消息，然后向第一个回答的节点发送加入命令。如果没有种子节点回复（可能尚未启动），它会重试此过程，直到成功或停止。

种子节点可以按任何顺序启动。没有必要所有种子节点都运行，但集群中配置为`seed-nodes`列表中的**第一元素**的节点必须被启动。如果没有，其他种子节点将不会被初始化，也没有其他节点可以加入集群。第一个种子节点的特殊性原因是为了避免从一开始就形成分离的孤岛。最好以最快的速度同时启动所有配置的种子节点（顺序无关紧要），否它将等待配置的`seed-node-timeout`时长后才能开始添加节点。（节点的添加必须等到必要数量的种子节点启动完毕，而这个过程最长可能达到`seed-node-timeout`，因此同时启动所有节点能够缩短第一个种子节点与其他节点的同步时间，加快启动速度。）

当两个以上的节点启动后，哪怕第一个种子节点被关闭也没有问题了。如果第一个种子节点重新启动，它将首先尝试加入现有集群中的其他种子节点。请注意，如果您同时停止所有种子节点并使用相同的`seed-nodes`配置重新启动它们，它们将自行加入并形成一个新集群，而不是加入现有集群的其余节点。这可能不是我们想要的，可以通过将多个节点列为冗余种子节点来避免，并且不要同时停止所有节点。

如果要在不同的机器上启动节点，则需要在`application.conf`指定机器的 ip 地址或主机名，而不是`127.0.0.1`

#### 以编程方式加入种子节点

以编程方式加入对于通过外部工具或 API 在启动时**动态发现**其他节点非常有用。

```scala
import akka.actor.Address
import akka.actor.AddressFromURIString
import akka.cluster.typed.JoinSeedNodes

val seedNodes: List[Address] =
  List("akka://ClusterSystem@127.0.0.1:2551", "akka://ClusterSystem@127.0.0.1:2552").map(AddressFromURIString.parse)
Cluster(system).manager ! JoinSeedNodes(seedNodes)
```

种子节点地址列表与配置的`seed-nodes`语义相同，底层实现过程也相同，参见[通过配置文件加入种子节点](cluster.md#通过配置文件加入种子节点)。

加入种子节点时，不应包含节点本身，除非是引导集群的第一个种子节点。当以程序加入时应将初始种子节点放在地址列表参数的首位。

#### 调整连接

节点在尝试联系种子节点失败后，会在配置属性中定义的`seed-node-timeout`时间到达之后自动重新联系种子节点。种子节点在新加失败后，等待配置的`retry-unsuccessful-join-after`时间到达后也将重新尝试联系所有种子节点，然后加入最先回答的节点。列表中的第一个节点如果无法在规定的`seed-node-timeout`时长之内联系上任何其他种子节点，它将加入它自己。

默认情况下，给定种子节点将无限次数地重试从新加入直到成功为止。如果感觉没必要的话，可以通过配置超时来中止该过程，中止时，它将运行[Coordinated Shutdown](coordinated-shutdown.md)，默认情况下将终止 ActorSystem。CoordinatedShutdown 也可以配置为退出 JVM。如果`seed-nodes`动态组装，定义此超时很有用，并且在尝试失败后应尝试使用新的种子节点重新启动。

```
akka.cluster.shutdown-after-unsuccessful-join-seed-nodes = 20s
akka.coordinated-shutdown.exit-jvm = on
```

如果您不配置种子节点或使用加入种子节点函数，则可以使用 [JMX](../additional/operations.md#jmx)或[HTTP](../additional/operations.md#http)手动加入集群。

您可以加入集群中的任何节点。它不必配置为种子节点。请注意，您只能加入现有的集群成员，引导节点必须加入自身，后续节点可以加入它们以组成集群。

一个 Actor system 只能加入一个集群一次，额外的尝试将被忽略。一旦 Actor system 成功加入集群，它必须重新启动才能再次加入同一个集群。重启后可以使用相同的主机名和端口。当它作为集群中现有成员的新化身出现并尝试加入时，现有成员将被删除并允许其新化身加入。

### 离开

有几种方法可以从集群中删除成员。

1. 建议以优雅地方式离开集群，向集群发出通知某个节点将离开。这在`ActorSystem`终止时由[联动关闭](coordinated-shutdown.md)执行，并且当系统向JVM发送 SIGTERM 停止信令时也会执行。
2. 也可以使用 [JMX](../additional/operations.md#jmx)或[HTTP](../additional/operations.md#http)执行优雅退出。
3. 当无法正常退出时，例如在 JVM 进程突然终止的情况下，该节点将被其他节点检测为无法访问并在标记为[停机](cluster.md#停机)后删除。

与突然终止和关闭相比，优雅离开在节点关闭期间可以更快地（将任务）切换到其他对等的节点。

集群节点发现自己正处在`Exiting`状态下时也将执行[联动关闭](coordinated-shutdown.md)，即从一个节点离开将触发离开节点上的关机过程。Akka Cluster 会自动添加优雅地离开集群时的各项任务，包括集群单例和集群分片的优雅关闭。例如，运行关闭过程也将触发优雅离开（如果尚未进行）。

通常这是自动执行的，但如果在此过程中出现网络故障，可能仍需要将节点的状态设置`Down`为以完成删除，请参阅[停机](cluster.md#停机)。

### 停机

在许多情况下，成员可以优雅地退出集群，如 [离开](cluster.md#离开) 中所述，但在某些情况下，需要明确的关闭决策才能将其删除。例如，在 JVM 进程突然终止、无法恢复的系统过载或无法修复的网络中断的情况下。在这种情况下，节点将被其他节点视为无法访问，它们必须在将其删除之前将其标注为`Down`。

当一个成员被故障检测器认为是`unreachable`时，Leader 将不允许其接受任务，比如将状态更改为“Up”。该节点必须首先再次变为`reachable`，或将状态更改为`Down`。将状态更改为`Down`可以自动或手动地执行。

我们建议您启用属于 Akka Cluster 模块的 [Split Brain Resolver](split-brain-resolver.md)。您可以通过配置启用它：

```
akka.cluster.downing-provider-class = "akka.cluster.sbr.SplitBrainResolverProvider"
```

您还应该考虑不同的[停机策略](split-brain-resolver.md#strategies)。

如果停机程序未配置，则必须使用[HTTP](operations.md#http)或[JMX](operations.md#jmx)手动执行停机。

请注意，在将一个 崩溃（unreachable）的[Cluster Singleton](cluster-singleton.)或[Cluster Sharding](cluster-sharding.md) 实体节点从集群中删除前之前，不会在另一个节点上启动新的节点。删除崩溃（无法访问）的节点将在停机决定做出之后。

停机也可以以编程的方式通过`Cluster(system).manager ! Down(address)`执行，但这通常在测试和实现`DowningProvider`时用到。

如果崩溃的节点重新启动并使用相同的主机名和端口再次加入集群，则该成员的先前化身必须将首先被关闭并删除。具有相同主机名和端口的新加入尝试用作前一个不再存在的证据。

如果一个节点仍在运行并且发现它自己处于`Down`状态，它将自行关闭。如果`run-coordinated-shutdown-when-down`被设置为`on`（缺省），[Coordinated Shutdown](coordinated-shutdown.md)将自动运行，但是节点不会尝试优雅地离开集群。

## 节点角色

并非集群的所有节点都需要执行相同的功能。例如，可能有一个运行 Web 前端的子集，一个运行数据访问层，一个用于数字处理。可以选择在每个节点上启动哪些Actor，例如通过集群感知路由器实现这种职责分配。

节点角色在配置中由`akka.cluster.roles`属性定义，并且通常在启动脚本中定义为系统属性或环境变量。

角色是你可以在`MemberEvent`中查阅的成员信息的一部分。自己节点的角色可从`selfMember`中查阅并可作为启动某些特定的 Actor 的条件：

```scala
val selfMember = Cluster(context.system).selfMember
if (selfMember.hasRole("backend")) {
  context.spawn(Backend(), "back")
} else if (selfMember.hasRole("frontend")) {
  context.spawn(Frontend(), "front")
}
```

## 故障检测器

集群中的节点通过发送心跳来检测节点是否无法从集群的其余部分访问，从而相互监视。请参见：

- [故障检测器](cluster-concepts.md#故障检测器)
- [Phi值应计故障检测器](failure-detector.md)的实现
- [使用故障检测器](cluster.md#使用故障检测器)

### 使用故障检测器

集群默认使用`akka.remote.PhiAccrualFailureDetector`故障检测器，或者您可以通过实现`akka.remote.FailureDetector`来提供您的实现并配置它：

```
akka.cluster.implementation-class = "com.example.CustomFailureDetector"
```

在[集群配置](cluster.md#配置)中您可以根据您的情况调整这些参数：

- 当*phi*值被认为是失败时：`akka.cluster.failure-detector.threshold`
- 突发异常的误差幅度： `akka.cluster.failure-detector.acceptable-heartbeat-pause`

## 如何测试

Akka 附带并使用了几种类型的测试策略：

- [测试](testing.md)
- [多节点测试](multi-node-testing.md)
- [多 JVM 测试](multi-jvm-testing.md)

## 配置

集群有几个配置属性。有关完整的配置说明、默认值和选项，请参阅[参考配置](configuration-reference.md)。

### 如何在达到集群设定时启动Actor

一个常见的用例是在集群初始化、成员加入并且集群达到一定规模后启动actor。

使用以下配置选项，您可以让 Leader 将“Joining”成员的成员状态更改为“Up”之前定义所需的成员数量。：

```
akka.cluster.min-nr-of-members = 3
```

以类似的方式，您可以让 Leader 在将成员的成员状态由 “Joining” 更改为“Up”之前定义特定角色所需的成员数量。：

```
akka.cluster.role {
  frontend.min-nr-of-members = 1
  backend.min-nr-of-members = 2
}
```

### 集群信息记录

您可以使用配置属性关闭`info`级别上的集群事件的日志输出：

```
akka.cluster.log-info = off
```

您可以通过配置项启用集群事件的详细输出，例如用于临时故障排除：

```
akka.cluster.log-info-verbose = on
```

### 集群调度器

Cluster 扩展是通过 actor 实现的。为了保护它们免受用户 actor 的干扰，它们默认运行在由 `akka.actor.internal-dispatcher`指定的内部 dispatcher 上，集群 actor 可以在该设置的内部使用 `akka,cluster.use-dispatcher`或在同一个调度程序上保持线程数来进一步隔离。

### 配置兼容性检查

创建集群就是部署两个或更多节点，并让它们像单个应用程序一样运行。因此，保持集群中的所有节点的配置彼此兼容非常重要。

配置兼容性检查功能确保集群中的所有节点都具有兼容的配置。每当新节点加入现有集群时，其配置设置的子集（仅需要检查的那些）将发送到集群中的节点进行验证。在集群端检查配置后，集群会发回自己的一组所需配置设置。然后加入节点将验证它是否符合集群配置。只有当双方的所有检查都通过时，加入节点才可以继续。

可以通过扩展`akka.cluster.JoinConfigCompatChecker`并在配置中包含新的自定义检查器来添加它们。每个检查器必须与一个唯一的键相关联：

```
akka.cluster.configuration-compatibility-check.checkers {
  my-custom-config = "com.company.MyCustomJoinConfigCompatChecker"
}
```

注意

```
配置兼容性检查默认启用，但可以通过设置禁用`akka.cluster.configuration-compatibility-check.enforce-on-join = off`。这在执行滚动更新时特别有用。显然，只有在无法选择完全关闭集群的情况下才应该这样做。具有不同配置设置的节点的集群可能会导致数据丢失或数据损坏。

此设置应仅在加入节点上禁用。始终在双方执行检查，并记录警告。在不兼容的情况下，加入节点有责任决定是否应该中断进程。

如果您使用 Akka 2.5.9 或之前版本（不支持此功能）对集群执行滚动更新，则不会执行检查，因为正在运行的集群无法验证加入节点发送的配置，也无法发回自己的配置。 
```

## 高级别的集群工具

### 集群单例

对于某些用例，确保只有一个特定类型 actor 在集群中的某处运行是方便或必要的。这可以通过订阅成员事件来实现，但有几个极端情况需要考虑。因此，此特定用例包含在 Cluster Singleton 中。

请参阅[集群单例](cluster-singleton.md)。

### 集群分片

将参与者分布在集群中的多个节点上，并通过 actor 的逻辑标识符与 actor 进行交互，而不必关心他们在集群中的物理位置。

请参阅[集群分片](cluster-sharding.md)。

### 分布式数据

当您需要在 Akka 集群中的节点之间共享数据时，分布式数据很有用。数据由具有键值存储的actor提供，就像API 一样访问。

请参阅[分布式数据](distributed-data.md)。

### 分布式发布订阅

在集群中Actor之间基于主题发布订阅消息，发送方不必知道目标actor正在哪个节点上运行。

请参阅[分布式发布订阅](distributed-pub-sub.md)。

### 集群感知路由器

例如使用轮询或一致散列等路由策略将消息分发给集群中不同节点上的参与者。

请参阅[组路由器](routers.md#group-router)。

### 跨多个数据中心的集群

Akka Cluster 可以跨多个数据中心、可用区或区域使用，这样一个 Cluster 就可以跨越多个数据中心，并且仍然可以容忍网络分区。

请参阅[集群多 DC](cluster-dc.md)。

### 可靠的交付

集群中参与者之间消息的可靠传递和流量控制。

查看[可靠交付](reliable-delivery.md)

## 示例项目

[集群示例项目](https://developer.lightbend.com/start/?group=akka&project=akka-samples-cluster-scala)是一个可以下载的示例项目，并附有如何运行的说明。

该项目包含说明不同集群功能的示例，例如订阅集群成员事件，以及使用集群感知路由器向在集群节点上运行的参与者发送消息。

---

[Cluster Specification ](https://doc.akka.io/docs/akka/current/typed/cluster-concepts.html)

---

