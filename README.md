# Akka typed 中文指南(Scala)

该文档介绍 akka typed Scala 语言的开发与使用，为节约翻译的时间，该项目借助了（基于 Java 语言的 Akka classic 中文文档） https://github.com/guobinhit/akka-guide 项目的部分翻译，并在此基础上根据 Akka 官方的 Scala 语言教程重新翻译、修订而成。特此声明与感谢！


- [Gitter Chat](https://gitter.im/akka/akka?source=orgpage)，Akka 在线交流平台；
- [Akka Forums](https://discuss.lightbend.com/c/akka/)，Akka 论坛；
- [Akka in GitHub](https://github.com/akka/akka)，Akka 开源项目仓库；
- [Akka Official Website](https://akka.io/)，Akka 官网；
- [Akka Java API](https://doc.akka.io/japi/akka/2.6/overview-summary.html)，Akka 应用程序编程接口。


## 快速入门指南 

- [快速入门 Akka classic Java 指南](articles/quickstart-akka-java.md)
- [快速入门 Akka typed Scala 指南](articles/quickstart-akka-scala.md)

## 目录

- [安全公告](articles/security-announcements.md)
- [入门指南](articles/getting-started-guide//index.md)
  - [Akka 简介](articles/getting-started-guide/introduction-to-akka.md) 
  - [为什么现代系统需要新的编程模型](articles/getting-started-guide/actors-motivation.md) 
  - [Actor 模型如何满足现代分布式系统的需求](articles/getting-started-guide/actor-intro.md)
  - [Akka 库和模块概述](articles/getting-started-guide/modules.md) 
  - [Akka 应用程序示例简介](articles/getting-started-guide/tutorial.md)
  - [第 1 部分: Actor 的体系结构](articles/getting-started-guide/tutorial_1.md)
  - [第 2 部分: 创建第一个 Actor](articles/getting-started-guide/tutorial_2.md)
  - [第 3 部分: 使用设备 Actors](articles/getting-started-guide/tutorial_3.md)
  - [第 4 部分: 使用设备组](articles/getting-started-guide/tutorial_4.md)
  - [第 5 部分: 查询设备组](articles/getting-started-guide/tutorial_5.md)
- [一般概念](articles/general-concepts/index.md)
  - [术语及概念](articles/general-concepts/terminology.md)
  - [Actor 系统](articles/general-concepts/actor-systems.md)
  - [什么是 Actor？](articles/general-concepts/actors.md)
  - [监督和监控](articles/general-concepts/supervision.md)
  - [Actor的引用、路径和地址](articles/general-concepts/addressing.md)
  - [位置透明](articles/general-concepts/remoting.md)
  - [Akka 和 Java 内存模型](articles/general-concepts/jmm.md)
  - [消息传递可靠性](articles/general-concepts/message-delivery-reliability.md)
  - [配置](articles/general-concepts/configuration.md)
- [Actors](articles/typed/index.md)
  - [Actor 介绍](articles/typed/actors.md)
  - [Actor 生命周期](articles/typed/actor-lifecycle.md)
  - [交互模式](articles/typed/interaction-patterns.md)
  - [容错](articles/typed/fault-tolerance.md)
  - [Actor发现](articles/typed/actor-discovery.md)
  - [路由](articles/typed/routers.md)
  - [Stash](articles/typed/stash.md)
  - [将 Behaviors 做为有限状态机](articles/typed/fsm.md)
  - [协同关闭](articles/typed/coordinated-shutdown.md)
  - [调度器](articles/typed/dispatchers.md)
  - [邮箱](articles/typed/mailboxes.md)
  - [测试](articles/typed/testing.md)
  - [共存](articles/typed/coexisting.md)
  - [编程风格指南](articles/typed/style-guide.md)
  - [Learning Akka Typed from Classic](articles/typed/from-classic.md)
- [集群](https://doc.akka.io/docs/akka/current/index-cluster.html)
  - [集群规范](articles/clustering/cluster-specification.md) 
  - [集群的使用方法](articles/clustering/cluster-usage.md) 
  - [集群感知路由器](articles/clustering/cluster-routing.md) 
  - [集群单例](articles/clustering/cluster-singleton.md) 
  - [集群中的分布式发布订阅](articles/clustering/distributed-pub-sub.md) 
  - [集群客户端](articles/clustering/cluster-client.md) 
  - [集群分片](articles/clustering/cluster-sharding.md)
  - [集群度量扩展](articles/clustering/cluster-metrics.md) 
  - [分布式数据](articles/clustering/distributed-data.md) 
  - [跨多个数据中心集群](articles/clustering/cluster-dc.md) 
  - [多虚拟机测试](articles/clustering/multi-jvm-testing.md) 
  - [多节点测试](articles/clustering/multi-node-testing.md) 
- [流](articles/stream/index.md)
  - [介绍](articles/stream/stream-introduction.md)
  - [Streams Quickstart Guide](articles/stream/stream-quickstart.md)
  - [Akka Streams 背后的设计原则](articles/stream/stream-design.md)
- [网络](https://doc.akka.io/docs/akka/current/index-network.html)
  - [远程处理](https://doc.akka.io/docs/akka/current/remoting.html) 
  - [远程处理（代号动脉）](https://doc.akka.io/docs/akka/current/remoting-artery.html)
  - [序列化](https://doc.akka.io/docs/akka/current/serialization.html) 
  - [I/O](https://doc.akka.io/docs/akka/current/io.html) 
  - [使用 TCP](https://doc.akka.io/docs/akka/current/io-tcp.html) 
  - [使用 UDP](https://doc.akka.io/docs/akka/current/io-udp.html) 
  - [DNS 扩展](https://doc.akka.io/docs/akka/current/io-dns.html) 
  - [Camel](https://doc.akka.io/docs/akka/current/camel.html)  
- [发现](articles/discovery-index.md)
- [协作](articles/coordination.md)
- [Futures 和 Agents](https://doc.akka.io/docs/akka/current/index-futures.html)
  - [Futures](https://doc.akka.io/docs/akka/current/futures.html) 
  - [Agents](articles/index-futures/agents.md) 
- [工具](https://doc.akka.io/docs/akka/current/index-utilities.html)
  - [事件总线](articles/index-utilities/event-bus.md) 
  - [日志记录](articles/index-utilities/logging.md) 
  - [调度程序](articles/index-utilities/scheduler.md) 
  - [持续时间](articles/index-utilities/duration.md) 
  - [断路器](articles/index-utilities/circuitbreaker.md) 
  - [Java 8 兼容性](articles/index-utilities/java8-compat.md)
  - [Akka 扩展](articles/index-utilities/extending-akka.md) 
- [其他 Akka 模块](https://doc.akka.io/docs/akka/current/common/other-modules.html)
  - [Akka HTTP](https://doc.akka.io/docs/akka-http/current/?language=java) 
  - [Alpakka](https://doc.akka.io/docs/alpakka/current/) 
  - [Alpakka Kafka Connector](http://doc.akka.io/docs/akka-stream-kafka/current/home.html) 
  - [Akka 持久化的 Cassandra 插件](https://github.com/akka/akka-persistence-cassandra) 
  - [Akka 持久化的 Couchbase 插件](https://doc.akka.io/docs/akka-persistence-couchbase/current/) 
  - [Akka 管理](http://developer.lightbend.com/docs/akka-management/current/) 
  - [Akka gRPC](https://doc.akka.io/docs/akka-grpc/current/) 
  - [社区项目](https://doc.akka.io/docs/akka/current/common/other-modules.html) 
  - [Lightbend 赞助的相关项目](https://doc.akka.io/docs/akka/current/common/other-modules.html) 
    - [Play 框架](https://www.playframework.com) 
    - [Lagom](https://www.lagomframework.com) 
- [如何：常见模式](articles/howto.md)
- [项目信息](https://doc.akka.io/docs/akka/current/project/index.html)
  - [迁移指南](articles/project/migration-guides.md) 
  - [滚动更新](articles/project/rolling-update.md)
  - [问题追踪](articles/project/issue-tracking.md)
  - [许可证](articles/project/licenses.md)
  - [项目](articles/project/links.md)
- [附加信息](https://doc.akka.io/docs/akka/current/additional/index.html)
  - [二进制兼容规则](articles/additional/binary-compatibility-rules.md)
  - [模块标记为“可能改变”](articles/additional/may-change.md)
  - [如何部署 Akka?](articles/additional/deploy.md)
  - [常见问题](articles/additional/faq.md)
  - [IDE 提示](articles/additional/ide.md)
  - [书籍和视频](articles/additional/books.md)
  - [OSGi 中的 Akka](articles/additional/osgi.md)



----------

**English Original Editon**: [Akka Documentation](https://doc.akka.io/docs/akka/current/index.html)

