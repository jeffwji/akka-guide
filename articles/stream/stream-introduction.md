# 介绍

## 动机

我们今天从 Internet  消费的服务中包括许多流数据的服务实例，包括从下载和上传到服务或点对点数据传输等。将数据视为元素流而不是其整体是非常有用的视角，因为它与计算机发送和接收它们的方式相匹配（例如通过  TCP），并且通常也是必要的，因为数据集经常变得太大而无法当作整体来处理。我们将计算或分析分布在大型集群上，并将其称为“大数据”，处理它们的整个原理是将它们以数据流的形式喂给不同的 CPU。

Actor也可以以同样的视角来处理流：他们发送和接收一系列消息，以便将知识（或数据）从一个地方传输到另一个地方。我们发现实施所有适当的措施以实现Actor之间的稳定流传输是乏味且容易出错的，因为除了发送和接收之外，我们还需要注意不要在此过程中发生任何缓冲区或邮箱溢出。另一个陷阱是 Actor 消息可能会丢失并且在这种情况下必须重新传输。否则会导致接收侧出现漏洞。

出于这些原因，我们决定将这些问题的解决方案捆绑在一起以形成 Akka Streams API。目的是提供一种直观且安全的方式来制定流处理的设置，以便我们可以有效地执行它们并限制资源使用——不再出现  OutOfMemoryErrors。为了实现这一点，我们需要能够限制流使用的缓冲区，如果消费者跟不上，它们需要能够减慢生产者的速度。此功能称为背压，是[Reactive Streams](https://www.reactive-streams.org/)的核心思想，Akka 是其中的创始成员之一。对你来说，这意味着传播和对背压作出反应的难题已经被纳入 Akka Streams 的设计中，所以你不必担心一件事；这也意味着 Akka Streams 可以与所有其他 Reactive Streams 实现无缝互操作（Reactive Streams 接口定义了互操作性 SPI，而 Akka Streams 等实现提供了一个很好的用户 API）。

### 与反应式流的关系

Akka Streams API 与 Reactive Streams 接口完全分离。Akka Streams 专注于数据流转换的制定，Reactive Streams 的范围是定义如何跨异步边界移动数据而不会出现丢失、缓冲或资源耗尽的通用机制。

这两者之间的关系是 Akka Streams API 面向最终用户，Akka Streams 在内部实现中使用 Reactive Streams 接口在不同运算符之间传递数据。因此，您不会发现 Reactive Streams 接口和 Akka Streams API 之间有任何相似之处。这符合 Reactive Streams 项目的预期，其主要目的是定义接口，以便不同的流实现可以互操作；Reactive Streams 的目的不是描述最终用户 API。

## 如何阅读这些文档

流处理是与 Actor 模型或 Future 组合不同的范式，因此可能需要仔细研究这个主题，直到您熟悉这些工具和技术。该文档旨在提供帮助，为了获得最佳结果，我们建议通过以下方法循序渐进：

- 阅读[快速入门指南](stream-quickstart.md)以了解流的外观以及它们的功能。
- 自上而下的学习者此时可能想要仔细阅读[Akka Streams 背后的设计原则](stream-design.md)。
- 自下而上的学习者可能会感觉[Streams Cookbook](stream-cookbook.md)更自在。
- 有关内置处理Operator的完整概述，您可以查看[Operator 索引](operators/index.md)
- 其他部分可以按顺序阅读，也可以在前面的步骤中根据需要阅读，每个部分都深入研究特定主题。

----

[快速入门指南](stream-quickstart.md)

----

https://doc.akka.io/docs/akka/current/stream/stream-quickstart.html