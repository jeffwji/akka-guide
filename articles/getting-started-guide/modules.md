# Akka 库和模块概述
在深入研究 Actors 编程的一些最佳实践之前，预览最常用的 Akka 库会很有帮助。这将帮助你开始考虑你要在系统中使用的功能。所有核心的 Akka 功能都可以作为开源软件（`OSS`）提供。Lightbend 支持 Akka 开发，但也可以给你提供「[商业服务](https://www.lightbend.com/lightbend-platform-subscription)」，如培训、咨询、支持和「[企业套件](https://www.lightbend.com/lightbend-platform
)」，这是一套用于管理 Akka 系统的综合工具。

Akka OSS 包含以下功能，稍后将在本页介绍这些功能：

- [Actor library](##Actor-library)
- [Remoting](##Remoting)
- [Cluster](##Cluster)
- [Cluster Sharding](##Cluster-Sharding)
- [Cluster Singleton](##Cluster-Singleton)
- [Persistence](##Persistence)
- [Distributed Data](##Distributed-Data)
- [Streams](##Streams)
- [Alpakka](##Alpakka)
- [HTTP](##HTTP)
- [gRPC](##gRPC)
- [其它  Akka 模块](https://doc.akka.io/docs/akka/current/common/other-modules.html)

通过 [Lightbend Platform Subscription](https://www.lightbend.com/lightbend-subscription?_ga=2.1540465.1982410223.1624840999-1885200556.1624035854), 您可以使用 [Akka 增强功能](https://doc.akka.io/docs/akka-enhancements/current/)，其中包括:

[Akka 弹性增强功能](https://doc.akka.io/docs/akka-enhancements/current/akka-resilience-enhancements.html)：

- [Configuration Checker](https://developer.lightbend.com/docs/akka-commercial-addons/current/config-checker.html) — 检查潜在的配置问题并记录建议。

- [Diagnostics Recorder](https://developer.lightbend.com/docs/akka-commercial-addons/current/diagnostics-recorder.html) — 以一种便于在开发和生产过程中排除故障的格式捕获配置和系统信息。

- [Thread Starvation Detector](https://developer.lightbend.com/docs/akka-commercial-addons/current/starvation-detector.html) —监控 Akka 系统调度程序并在它无响应时记录警告。

- [Fast Failover](https://developer.lightbend.com/docs/akka-commercial-addons/current/fast-failover.html) — 群集分片（`Cluster Sharding`）的快速故障转移（`Fast failover`）。

[Akka 持久化增强](https://doc.akka.io/docs/akka-enhancements/current/akka-persistence-enhancements.html):

- [GDPR for Akka Persistence](https://developer.lightbend.com/docs/akka-commercial-addons/current/gdpr/index.html) — 数据分解可用于忘记事件中的信息。

此页面并未列出所有可用模块，但概述了主要功能，让你了解在 Akka 上构建系统时可以达到的复杂程度。

## Actor library

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
Akka 的核心库是`akka-actor-typed`，但是 Actor 在整个 Akka 库中使用，它提供了一个一致的、集成的模型，使您无需单独解决并发或分布式系统设计中出现的挑战。从鸟瞰图（`birds-eye`）来看，actor 是一种编程范式，它将封装（OOP 的支柱之一）发挥到了极致。与对象不同，Actor 不仅封装了他们的状态，而且还封装了他们的执行。与 Actor 的通信不是通过方法调用，而是通过传递消息。虽然这种差异看起来很小，但实际上它允许我们在并发性和远程通信方面打破 OOP 的限制。如果这个描述感觉太高而无法完全掌握，请不用担心，下一章我们将详细解释 Actor。现在，重要的一点是，这是一个在基础级别处理并发性和分布的模型，而不是将这些特性引入 OOP 的临时补丁尝试。

Actor 解决的挑战包括：

- 如何构建和设计高性能并发应用程序。
- 如何在多线程环境中处理错误。
- 如何保护自己的项目免受并发性的陷阱。

## Remoting
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
    <artifactId>akka-remote_${scala.binary.version}</artifactId>
  </dependency>
</dependencies>

<!-- Gradle -->
def versions = [
  ScalaBinary: "2.13"
]
dependencies {
  implementation platform("com.typesafe.akka:akka-bom_${versions.ScalaBinary}:2.6.15")
  implementation "com.typesafe.akka:akka-remote_${versions.ScalaBinary}"
}

<!-- sbt -->
val AkkaVersion = "2.6.15"
libraryDependencies += "com.typesafe.akka" %% "akka-remote" % AkkaVersion
```
远程处理（`Remoting`）使存活（`live`）在不同计算机上的 Actor 能够无缝地交换消息。虽然被作为 JAR 工件（`artifact`）分发，但是Remoting更像一个模块，而不是一个库。你主要通过配置来启用它，它只有几个 API。感谢actor模型，远程和本地消息发送看起来完全相同。在本地系统上使用的模式可以直接转换到远程系统。你很少需要直接使用远程处理，但它提供了构建集群子系统（`Cluster subsystem`）的基础。

远程处理解决的挑战包括：

- 如何处理远程主机上的 Actor 系统。
- 如何在远程 Actor 系统上处理单个 Actor。
- 如何将消息转换为线路上的字节。
- 如何管理主机之间的低级（`low-level`）网络连接（和重新连接），如何透明地检测崩溃的 Actor 系统和主机。
- 如何透明的在同一网络连接上从一组不相关的 Actor 那里复用通信。

## Cluster
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
    <artifactId>akka-cluster-typed_${scala.binary.version}</artifactId>
  </dependency>
</dependencies>

<!-- Gradle -->
def versions = [
  ScalaBinary: "2.13"
]
dependencies {
  implementation platform("com.typesafe.akka:akka-bom_${versions.ScalaBinary}:2.6.15")
  implementation "com.typesafe.akka:akka-cluster-typed_${versions.ScalaBinary}"
}

<!-- sbt -->
val AkkaVersion = "2.6.15"
libraryDependencies += "com.typesafe.akka" %% "akka-cluster-typed" % AkkaVersion
```
如果你有一组协作解决某些业务问题的 Actor 系统，那么你可能希望以规范的方式管理这些系统集（`set of systems`）。当远程处理解决了与远程系统组件寻址和通信的问题时，集群（`Clustering`）使你能够将它们组织成一个由成员协议绑定在一起的“元系统（`meta-system`）”。在大多数情况下，你希望使用集群模块（`Cluster module`）而不是直接使用远程（`Remoting`）。集群在 Remoting 之上提供了一组额外的服务，这是大多数实际应用程序所需要的。

集群模块解决的挑战包括：

- 如何维护一组 Actor 系统（一个集群），这些系统可以相互通信，并将彼此视为集群的一部分。
- 如何将新系统安全地引入到已经存在的成员集中。
- 如何可靠地检测暂时无法访问的系统。
- 如何删除失败的主机/系统（或缩小系统规模），以便集群中所有剩余的成员保持一致性。
- 如何在当前成员集中分布计算。
- 如何将集群成员指定到某个角色，换句话说，提供某些特定的服务而不是其他服务。

## Cluster Sharding
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
    <artifactId>akka-cluster-sharding-typed_${scala.binary.version}</artifactId>
  </dependency>
</dependencies>

<!-- Gradle -->
def versions = [
  ScalaBinary: "2.13"
]
dependencies {
  implementation platform("com.typesafe.akka:akka-bom_${versions.ScalaBinary}:2.6.15")
  implementation "com.typesafe.akka:akka-cluster-sharding-typed_${versions.ScalaBinary}"
}

<!-- sbt -->
val AkkaVersion = "2.6.15"
libraryDependencies += "com.typesafe.akka" %% "akka-cluster-sharding-typed" % AkkaVersion
```
分片（`Sharding`）有助于解决在 Akka 集群成员之间分配一组 Actor 的问题。分片是一种模式，它主要与持久性（`Persistence`）一起使用，以将一大组需要持久化的实体（由 Actors 支持）平衡到集群的各个成员，并在成员崩溃或离开时将它们迁移到其他节点。

分片解决的挑战包括：

- 如何在一组系统上建模和扩展大规模有状态的实体。
- 如何确保集群中的实体正确分布，以便在机器之间保持平衡负载。
- 如何确保从崩溃的系统迁移实体而不丢失状态。
- 如何确保一个实体不同时存在于多个系统上，从而保持一致性。

## Cluster Singleton
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
    <artifactId>akka-cluster-singleton_${scala.binary.version}</artifactId>
  </dependency>
</dependencies>

<!-- Gradle -->
def versions = [
  ScalaBinary: "2.13"
]
dependencies {
  implementation platform("com.typesafe.akka:akka-bom_${versions.ScalaBinary}:2.6.15")
  implementation "com.typesafe.akka:akka-cluster-singleton_${versions.ScalaBinary}"
}

<!-- sbt -->
val AkkaVersion = "2.6.15"
libraryDependencies += "com.typesafe.akka" %% "akka-cluster-singleton" % AkkaVersion
```
分布式系统中的一个常见（事实上，有点太常见）用例是让一个实体负责一个给定的任务，该任务在集群的其他成员之间共享，并且在主机系统发生故障时进行迁移。尽管这无疑会给整个集群带来一个限制扩展的常见瓶颈，但在某些情况下，使用这种模式是不可避免的。集群单例（`Cluster singleton`）允许集群选择一个 Actor 系统，该系统将承载一个特定的 Actor，而其他系统始终可以独立地访问该 Actor 承担的服务。

单例模块可用于解决这些挑战：

- 如何确保整个集群中只有一个服务实例在运行。
- 如何确保服务处于启动状态，即使承载它的系统在缩小规模的过程中崩溃或关闭。
- 如何从集群的任何成员访问这个实例，假设它可以随着时间迁移到其他系统。

## Persistence

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
    <artifactId>akka-persistence-typed_${scala.binary.version}</artifactId>
  </dependency>
</dependencies>

<!-- Gradle -->
def versions = [
  ScalaBinary: "2.13"
]
dependencies {
  implementation platform("com.typesafe.akka:akka-bom_${versions.ScalaBinary}:2.6.15")
  implementation "com.typesafe.akka:akka-persistence-typed_${versions.ScalaBinary}"
}

<!-- sbt -->
val AkkaVersion = "2.6.15"
libraryDependencies += "com.typesafe.akka" %% "akka-persistence-typed" % AkkaVersion
```

就像 OOP 中的对象一样，Actor 将其状态保存在易失性内存中。一旦系统正常关闭或突然崩溃，内存中的所有数据都将丢失。持久性（`Persistence`）提供了使 Actor 能够持久化导致其当前状态的事件的模式。启动时，可以重播事件以恢复由 Actor 承载的实体的状态。可以查询事件流并将其输入到其他处理管道（例如外部大数据集群）或备用视图（如报表）中。

持久性解决了以下挑战：

- 如何在系统重新启动或崩溃时恢复实体/参与者的状态。
- 如何实现「[CQRS](https://docs.microsoft.com/en-us/previous-versions/msp-n-p/jj591573(v=pandp.10))」系统。
- 如何在网络错误和系统崩溃时确保消息的可靠传递。
- 如何自查导致实体进入当前状态的域事件。
- 如何在项目继续发展的同时利用应用程序中的「[事件源](https://martinfowler.com/eaaDev/EventSourcing.html)」来支持长期运行的流程。

### Projections

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
    <artifactId>akka-projection-core_${scala.binary.version}</artifactId>
  </dependency>
</dependencies>

<!-- Gradle -->
def versions = [
  ScalaBinary: "2.13"
]
dependencies {
  implementation platform("com.typesafe.akka:akka-bom_${versions.ScalaBinary}:2.6.15")
  implementation "com.typesafe.akka:akka-projection-core_${versions.ScalaBinary}"
}

<!-- sbt -->
val AkkaVersion = "2.6.15"
libraryDependencies += "com.typesafe.akka" %% "akka-projection-core" % AkkaVersion
```

Projections 提供了一个简单的 API，用于消费事件流以投射到各种下游选项中。核心依赖仅提供 API，其他提供者依赖需要实现各自不同的源和接收器的实现。

Projections 解决以下挑战：

- 在事件流上构建分支或聚合视图。
- 将事件流传播到另一个下游媒介，例如 Kafka 的 topic。
- 在[事件溯源](https://martinfowler.com/eaaDev/EventSourcing.html)和[CQRS 系统](https://docs.microsoft.com/en-us/previous-versions/msp-n-p/jj591573%28v=pandp.10%29)的上下文中构建读取端投影的简单方法。

## Distributed Data
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
    <artifactId>akka-cluster-typed_${scala.binary.version}</artifactId>
  </dependency>
</dependencies>

<!-- Gradle -->
def versions = [
  ScalaBinary: "2.13"
]
dependencies {
  implementation platform("com.typesafe.akka:akka-bom_${versions.ScalaBinary}:2.6.15")
  implementation "com.typesafe.akka:akka-cluster-typed_${versions.ScalaBinary}"
}

<!-- sbt -->
val AkkaVersion = "2.6.15"
libraryDependencies += "com.typesafe.akka" %% "akka-cluster-typed" % AkkaVersion
```
在最终一致性可以接受的情况下，可以在 Akka 集群中的节点之间共享数据，甚至在集群分区面前也可以接受读和写。这可以通过使用「[无冲突的复制数据类型（CRDT）](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type)」来实现，其中不同节点上的写入可以并发进行，并随后以可预测的方式进行合并。分布式数据（`Distributed Data`）模块提供了共享数据和许多有用数据类型的基础结构。

分布式数据旨在解决以下挑战：

- 即使面对集群分区，如何接受写入。
- 如何在保证低延迟本地读写访问的同时共享数据。

## Streams
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
    <artifactId>akka-stream-typed_${scala.binary.version}</artifactId>
  </dependency>
</dependencies>

<!-- Gradle -->
def versions = [
  ScalaBinary: "2.13"
]
dependencies {
  implementation platform("com.typesafe.akka:akka-bom_${versions.ScalaBinary}:2.6.15")
  implementation "com.typesafe.akka:akka-stream-typed_${versions.ScalaBinary}"
}

<!-- sbt -->
val AkkaVersion = "2.6.15"
libraryDependencies += "com.typesafe.akka" %% "akka-stream-typed" % AkkaVersion
```
Actor 是并发性的基本模型，但是有一些共同的模式，它们的使用要求用户反复实现相同的模式。非常常见的情况是，Actor 的链或图需要处理潜在的大型或无限的连续事件流，并适当地协调资源使用，以便更快的处理阶段不会压倒链或图中较慢的阶段。流（`Streams`）在 Actor 之上提供了更高级别的抽象，从而简化了编写此类处理网络、处理后台的所有细节，并提供了一个安全、类型化、可组合的编程模型。流也是响应式流（`Reactive Streams`）标准的实现，它支持与该标准的所有第三方实现集成。

流解决了以下挑战：

- 如何以高性能处理事件流或大型数据集，如何利用并发性并保持资源使用的严格性。
- 如何将可重用的事件/数据处理片段组装成灵活的管道。
- 如何以灵活的、高性能的方式将异步服务相互连接。
- 如何提供或使用与第三方库接口的响应式流兼容接口。

## Alpakka

[Alpakka](https://doc.akka.io/docs/alpakka/current/)是一个独立于 Akka 的模块。

Alpakka 是基于 Streams API 构建的模块集合，可为云和基础设施领域中的各种常见技术提供反应式流(` Reactive Stream`) 连接器实现。
有关 API 和可用实现模块的更多详细信息，请参阅[Alpakka 概述页面](https://doc.akka.io/docs/alpakka/current/overview.html)。

Alpakka 有助于解决以下挑战：

- 将各种基础设施或持久性组件连接到基于流的工作流程。
- 以符合 Reactive Streams API 的方式连接到遗留系统。

## HTTP

「[Akka HTTP](https://doc.akka.io/docs/akka-http/current/)」是 Akka 的一个独立模块。

实际上，远程、内部或外部提供 API 的标准是「[HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)」。Akka 提供了一个库，通过提供一组工具来创建 HTTP 服务（并为其提供服务），以及一个可用于其他服务的客户端来使用此类 HTTP 服务。这些工具特别适合通过利用 Akka 流的底层模型来流入和流出大量数据或实时事件。

HTTP 解决的一些挑战：

- 如何通过 HTTP API 以高性能方式向外部公开系统或集群的服务。
- 如何使用 HTTP 将大型数据集流入和流出系统。
- 如何使用 HTTP 将活动事件（`live events`）流入和流出系统。

## gRPC

[Akka gRPC](https://doc-akka-io.translate.goog/docs/akka-grpc/current/index.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax)是一个独立于 Akka 的模块。

该库提供了与[HTTP](#http)和[Streams](#streams)模块很好地集成的 gRPC 实现。它能够从 protobuf 服务中定义并生成客户端和服务器端工件，然后可以使用 Akka HTTP 公开这些工件，并使用 Streams 进行处理。

Akka gRPC 解决的一些挑战：

- 获得使用 gRPC 和 protobuf 公开服务的所有好处：
- 模式优先合约。
- 支持模式进化。
- 高效的二进制协议
- 对流第一级的支持。
- 广泛的互操作性
- 使用 HTTP2 实现连接的复用。

## 模块使用示例

Akka 模块无缝集成在一起。例如，想想网站用户访问的一大组有状态的业务对象，例如文档或购物车。如果你使用分片（`Sharding`）和持久性（`Persistence`）将它们建模为分片实体（`sharded entities`），那么它们将在集群中得到平衡，你可以按需扩展。即使某些系统崩溃，它们也可以在广告活动高峰期间或节假日提供服务。你还可以使用持久性查询（`Persistence Query`）获取域事件的实时流，并使用流（`Streams`）将它们传输到流式快速数据引擎（`Fast Data engine`）中。然后，将该引擎的输出作为流（`Streams`），使用`Akka Streams`操作符对其进行操作，并将其公开为由集群托管的一组负载平衡的 HTTP 服务器提供服务的 Web 套接字连接，以支持实时业务分析工具。

我们希望这次预览能引起你的兴趣！下一个主题将介绍我们在本指南的教程部分中构建的示例应用程序。

----------

[示例介绍](tutorial.md)

**英文原文链接**：[Overview of Akka libraries and modules](https://doc.akka.io/docs/akka/current/guide/modules.html).

----------
———— ☆☆☆ —— [返回 -> Akka 中文指南 <- 目录](https://github.com/guobinhit/akka-guide/blob/master/README.md) —— ☆☆☆ ————