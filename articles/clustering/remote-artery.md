# Artery远程处理

```
Note

远程处理是不同节点上的 Actor 在内部相互通信的机制。

在构建 Akka 应用程序时，您通常不会直接使用 Remoting 概念，而是使用更高级的Akka Cluster实用程序或与技术无关的协议，例如HTTP、gRPC等。
```

如果从经典远程迁移，请查看[Artery提供的新功能](#Artery提供的新功能)

## 依赖

要使用 Artery Remoting，您必须在项目中添加以下依赖项：

```xml
<-- sbt --/>
val AkkaVersion = "2.6.15"
libraryDependencies += "com.typesafe.akka" %% "akka-remote" % AkkaVersion
```

另一种选择是将 Artery 与 Aeron 一起使用，请参阅[选择交通工具](#选择交通工具)。如果使用`aeron-udp`传输，需要显式添加 Aeron 依赖项：

```
libraryDependencies ++= Seq(
  "io.aeron" % "aeron-driver" % "1.32.0",
  "io.aeron" % "aeron-client" % "1.32.0"
)
```

## 配置

要在您的 Akka 项目中启用远程功能，您至少应该将以下配置添加到您的`application.conf`文件中：

```json
akka {
  actor {
    # provider=remote is possible, but prefer cluster
    provider = cluster 
  }
  remote {
    artery {
      transport = tcp # See Selecting a transport below
      canonical.hostname = "127.0.0.1"
      canonical.port = 25520
    }
  }
}
```

正如您在上面的示例中看到的，您需要添加四件事才能开始：

- 将 provider 改为本地。我们建议使用[Akka Cluster](https://doc.akka.io/docs/akka/current/cluster-usage.html)而不是直接使用远程处理。
- 启用 Artery 以将其用作远程处理实现
- 添加主机名 - 要在其上运行 actor 系统的机器；该主机名正是传递给远程系统以识别该系统并因此在需要时用于连接回该系统的名称，因此将其设置为可访问的 IP 地址或可解析的名称，以便您想通过网络进行通信。
- 添加端口号——actor 系统应该监听的端口，设置为 0 使其自动选择。

```
笔记

即使 actor 系统具有不同的名称，同一台机器上的每个actor 系统的端口号也需要是唯一的。这是因为每个 actor 系统都有自己的网络子系统，用于监听连接和处理消息，以免干扰其他actor 系统。
```

上面的示例仅说明了为启用远程处理而必须添加的最少属性。[远程配置](#远程配置)中描述了所有设置。

## 介绍

我们推荐[Akka Cluster](https://doc.akka.io/docs/akka/current/cluster-usage.html)而不是直接使用远程处理。但是由于远程处理是允许集群的底层模块，因此了解有关它的详细信息仍然很有用。

```
笔记  

本页介绍代号为 Artery 的远程处理子系统，它取代了[经典的远程处理实现](https://doc-akka-io.translate.goog/docs/akka/current/remoting.html) 。
```

远程处理使不同主机或 JVM 上的 Actor 系统能够相互通信。通过启用远程处理，系统将在提供的网络地址开始侦听，并获得通过网络连接到其他系统的能力。从应用程序的角度来看，本地或远程系统之间在 API 上没有区别，指向远程系统的`ActorRef`实例看起来与本地系统完全相同：它们可以发送消息、监视等。每个都`ActorRef`都包含主机名和端口信息并且甚至可以在网络上传递。这意味着在网络上，每个`ActorRef`都是该网络上某个actor的唯一标识符。

您需要为actor 消息启用[序列化](serialization.md)。在许多情况下，[使用 Jackson 进行序列化](serialization-jackson.md)是一个不错的选择，如果您没有其他偏好，我们建议您这样做。

远程处理不是服务器-客户端技术。所有使用远程处理的系统都可以联系网络上的任何其他系统，如果它们拥有`ActorRef`指向那些系统的指向。这意味着启用远程处理的每个系统都充当“服务器”，同一网络上的任意系统都可以连接到该“服务器”。

## 选择交通工具

有三种替代方案可以使用底层传输。它由`akka.remote.artery.transport`具有可能值的属性配置：

- `tcp`- 基于[Akka Streams TCP](https://doc-akka-io.translate.goog/docs/akka/current/stream/stream-io.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#streaming-tcp)（如果其他未配置，则默认）
- `tls-tcp`- 与`tcp`使用[Akka Streams TLS 的](https://doc-akka-io.translate.goog/docs/akka/current/stream/stream-io.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#tls)加密相同
- `aeron-udp`- 基于[Aeron (UDP)](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://github.com/real-logic/aeron)

如果您不确定选择什么，最好的选择是使用默认值，即`tcp`.

Aeron (UDP) 传输是一种高性能传输，应该用于需要高吞吐量和低延迟的系统。当系统空闲或处于低消息速率时，它比 TCP 使用更多的 CPU。Aeron 没有加密。

TCP 和 TLS 传输是使用 Akka Streams TCP/TLS 实现的。这是需要加密时的选择，但它也可以与没有 TLS 的普通 TCP 一起使用。当UDP不能使用时，这也是显而易见的选择。它具有非常好的性能（高吞吐量和低延迟），但高吞吐量时的延迟可能不如 Aeron 传输。与 Aeron 运输相比，它的操作复杂性更低，在集装箱环境中出现问题的风险也更低。

Aeron 需要 64 位 JVM 才能可靠工作，并且仅在 Linux、Mac 和 Windows 上得到官方支持。它可能适用于其他 Unix，例如 Solaris，但尚未进行足够的测试以使其得到正式支持。如果您使用的是 Big Endian 处理器，例如 Sparc，则建议使用 TCP。

​         笔记        

从一种传输更改为另一种传输时，不支持[滚动更新](https://doc-akka-io.translate.goog/docs/akka/current/additional/rolling-updates.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)。

## 从经典远程处理迁移

请参阅[从经典远程处理迁移](https://doc-akka-io.translate.goog/docs/akka/current/project/migration-guide-2.5.x-2.6.x.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#classic-to-artery)

## 规范地址

为了使远程处理正常工作，其中每个系统都可以向同一网络上的任何其他系统发送消息（例如，一个系统将消息转发到第三个系统，而第三个系统直接回复到发送方系统）对于每个系统具有*唯一的、全球可访问的*地址和端口。该地址是系统唯一名称的一部分，其他系统将使用该地址打开与其的连接并发送消息。这意味着如果一台主机有多个名称（不同的 DNS 记录指向同一个 IP 地址），那么其中只有一个可以是*规范的*.如果消息到达系统但它包含的主机名与预期的规范名称不同，则该消息将被丢弃。如果允许系统有多个名称，那么`ActorRef`实例之间的相等性检查将不再可信，这将违反参与者在给定网络上具有全局唯一引用的基本假设。因此，这也意味着 localhost 地址（例如*127.0.0.1*）不能在一般情况下使用（除了本地开发），因为它们不是真实网络中的唯一地址。

在使用网络地址转换 (NAT) 或涉及其他网络桥接的情况下，重要的是配置系统以使其了解外部可见的规范地址与主机端口对之间存在差异用于监听连接。有关详细信息，请参阅[NAT 后面或 Docker 容器中的 Akka](https://doc-akka-io.translate.goog/docs/akka/current/remoting-artery.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#remote-configuration-nat-artery)。

## 获取对远程参与者的引用

为了与演员交流，有必要拥有它的`ActorRef`. 在本地情况下，通常是 actor 的创建者（的调用者`actorOf()`）获得了`ActorRef`一个 actor，然后它可以将其发送给其他 actor。换句话说：

- Actor 可以通过接收来自它的消息（因为它当时可用）或在远程消息内部（例如*PleaseReply(message: String, remoteActorRef: ActorRef)*）来获取远程 Actor 的引用`sender()`

或者，参与者可以使用 查找位于已知路径上的另一个参与者`ActorSelection`。即使在启用远程处理的系统中，这些方法也可用：

- 远程查找：用于在远程节点上查找角色 `actorSelection(path)`
- 远程创建：用于在远程节点上创建一个actor `actorOf(Props(...), actorName)`

在接下来的部分中，将详细描述这两种替代方案。

### 查找远程 Actor

`actorSelection(path)`将获得`ActorSelection`远程节点上的Actor，例如：

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/remoting-artery.html#tab0)

  ​          `val selection =  context.actorSelection("akka://actorSystemName@10.0.0.1:25520/user/actorName") `        

- [          爪哇         ](https://doc.akka.io/docs/akka/current/remoting-artery.html#tab1)

从上面的示例中可以看出，以下模式用于在远程节点上查找参与者：

```
akka://<actor system>@<hostname>:<port>/<actor path>
```

​         笔记        

与早期的远程处理不同，协议字段始终是*akka，*因为不再支持可插拔传输。

一旦您获得了演员的选择，您就可以像与本地演员一样与它互动，例如：

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/remoting-artery.html#tab0)

  ​          `selection ! "Pretty awesome feature" `        

- [          爪哇         ](https://doc.akka.io/docs/akka/current/remoting-artery.html#tab1)

要获取一个`ActorRef`for ，`ActorSelection`您需要向选择发送一条消息并使用`sender`来自参与者的回复的引用。有一个内置`Identify`消息，所有 Actor 都会理解并自动回复`ActorIdentity`包含`ActorRef`. 这也可以使用 的`resolveOne`方法来完成，该方法`ActorSelection`返回`Future`匹配的`ActorRef`。

有关如何形成和使用角色地址和路径的更多详细信息，请参阅[角色参考、路径和地址](https://doc-akka-io.translate.goog/docs/akka/current/general/addressing.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)。

​         笔记        

发送到实际在发送actor系统中的actor的消息不会通过远程actor引用提供者传递。它们由本地演员参考提供者直接交付。

除了提供更好的性能之外，这还意味着如果您配置远程侦听的主机名实际上无法从同一个 actor 系统中解析，则此类消息将（可能违反直觉）传递得很好。

## 远程安全

一个`ActorSystem`不应该经由阿卡远程（动脉）通过纯的Aeron / UDP或TCP被暴露于不信任的网络（例如因特网）。它应该受到网络安全的保护，例如防火墙。如果认为这不够保护[，](https://doc-akka-io.translate.goog/docs/akka/current/remoting-artery.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#remote-tls)则应启用[具有相互身份验证的 TLS](https://doc-akka-io.translate.goog/docs/akka/current/remoting-artery.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#remote-tls)。

最佳实践是 Akka 远程节点只能从相邻网络访问。请注意，如果 TLS 启用了相互身份验证，则仍然存在攻击者可以通过破坏具有由同一内部 PKI 树颁发的证书的任何节点来访问有效证书的风险。

默认情况下，Akka 中禁用[Java 序列化](https://doc-akka-io.translate.goog/docs/akka/current/serialization.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#java-serialization)。这也是安全最佳实践，因为它有多个[已知的攻击面](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://community.hpe.com/t5/Security-Research/The-perils-of-Java-deserialization/ba-p/6838995)。



### 为 Akka 远程处理配置 SSL/TLS

通过使用传输，SSL 可以用作远程`tls-tcp`传输：

```
akka.remote.artery {
  transport = tls-tcp
}
```

接下来必须配置实际的 SSL/TLS 参数：

```
akka.remote.artery {
  transport = tls-tcp

  ssl.config-ssl-engine {
    key-store = "/example/path/to/mykeystore.jks"
    trust-store = "/example/path/to/mytruststore.jks"

    key-store-password = ${SSL_KEY_STORE_PASSWORD}
    key-password = ${SSL_KEY_PASSWORD}
    trust-store-password = ${SSL_TRUST_STORE_PASSWORD}

    protocol = "TLSv1.2"

    enabled-algorithms = [TLS_DHE_RSA_WITH_AES_128_GCM_SHA256]
  }
}
```

始终使用[环境变量替换](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://github.com/lightbend/config%23optional-system-or-env-variable-overrides)密码。不要在配置文件中定义真正的密码。

根据[RFC 7525](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://tools.ietf.org/html/rfc7525)，与 TLS 1.2 一起使用的推荐算法（在撰写本文档时）是：

- TLS_DHE_RSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
- TLS_DHE_RSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384

在配置系统之前，您应该始终检查有关安全性和算法建议的最新信息。

Lightbend 的 SSL-Config 库的[生成 X.509 证书](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://lightbend.github.io/ssl-config/CertificateGeneration.html%23using-keytool)部分详细记录了创建和使用密钥库和证书。

由于 Akka 远程处理本质上是[对等的，因此](https://doc-akka-io.translate.goog/docs/akka/current/general/remoting.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#symmetric-communication)需要在参与集群的每个远程处理节点上配置密钥库和信任库。

在 JVM 上设置安全性时，官方[Java 安全套接字扩展文档](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/JSSERefGuide.html)以及[有关创建 KeyStore 和 TrustStores](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://docs.oracle.com/cd/E19509-01/820-3503/6nf1il6er/index.html)的[Oracle 文档](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://docs.oracle.com/cd/E19509-01/820-3503/6nf1il6er/index.html)都是研究的重要资源。请在故障排除和配置 SSL 时查阅这些资源。

默认情况下启用 TLS 对等方之间的相互身份验证。相互身份验证意味着连接的被动端（TLS 服务器端）也将请求并验证来自连接对等方的证书。如果没有这种模式，只有客户端请求和验证证书。虽然 Akka 是一种点对点技术，但节点之间的每个连接都是从一侧（“客户端”）向另一侧（“服务器”）开始的。

请注意，如果 TLS 启用了相互身份验证，则仍然存在攻击者可以通过破坏具有由同一内部 PKI 树颁发的证书的任何节点来访问有效证书的风险。

建议您使用`akka.remote.artery.ssl.config-ssl-engine.hostname-verification=on`. 启用后，它将验证目标主机名是否与对等方证书中的主机名匹配。

在主机名是动态的且预先未知的部署中，关闭主机名验证是有意义的。

您有几个选择如何设置证书和主机名验证：

- 为所有节点设置一组密钥和一个证书并

  禁用

  主机名检查          

  - 单个密钥集和单个证书分发到所有节点。证书可以是自签名的，因为它既可以作为身份验证的证书，也可以作为受信任的证书分发。
  - 如果密钥/证书丢失，其他人可以连接到您的集群。
  - 向集群添加节点很简单，因为可以将密钥材料部署/分发到新节点。

- 为所有节点设置一组密钥和一个证书，其中包含所有主机名并

  启用

  主机名检查。          

  - 这意味着只有证书中提到的主机才能连接到集群。
  - 但是，如果您与之交谈的节点实际上是它应该是的节点（或者它是否是其他节点之一），则无法检查它。这似乎是一个小限制，因为无论如何您都必须信任 Akka 集群中的所有集群节点。
  - 证书可以自签名，在这种情况下，在所有节点上分发和信任相同的单个证书（但请参阅下一个项目符号）
  - 添加新节点意味着其主机名需要符合证书中的受信任主机名。这意味着首先要预见新主机，使用通配符证书，或者使用完整的 CA，以便以后如果要添加更多节点，您可以颁发更多证书（但是您已经进入下一个解决方案的领域）。
  - 如果证书被盗，它只能用于从可通过证书中受信任的主机名访问的节点连接到集群。它需要篡改 DNS 以允许其他节点访问集群（但是，在内部设置中篡改 DNS 可能比在 Internet 规模上更容易）。

- 有一个 CA，然后有密钥/证书，每个节点一个，并

  启用

  主机名检查。          

  - 基本上类似于 Internet HTTPS，但您只信任内部 CA，然后为每个新节点颁发证书。
  - 需要PKI，CA证书在所有节点上都是可信的，单独的证书用于认证。
  - 仅分发 CA 证书和节点的密钥/证书。
  - 如果密钥/证书被盗，则只有同一个节点可以访问集群（除非 DNS 也被篡改）。您可以撤销单个证书。

另请参阅[远程配置](https://doc-akka-io.translate.goog/docs/akka/current/remoting-artery.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#remote-configuration-artery)部分中的设置说明。

​         笔记        

在 Linux 上使用 SHA1PRNG 时，建议将其指定`-Djava.security.egd=file:/dev/urandom`为 JVM 的参数以防止阻塞。它不那么安全，因为它重复使用了种子。

### 不信任模式

一旦actor系统可以远程连接到另一个actor系统，它原则上可以向包含在该远程系统中的任何actor发送任何可能的消息。一个例子可能是向`PoisonPill`系统监护人发送，关闭该系统。这并不总是需要的，可以通过以下设置禁用它：

```
akka.remote.artery.untrusted-mode = on
```

这不允许发送系统消息（actor 生命周期命令、DeathWatch 等）以及任何扩展`PossiblyHarmful`到设置了此标志的系统的消息。尽管如此，如果客户端发送它们，它们会被丢弃并记录（在调试级别以减少拒绝服务攻击的可能性）。`PossiblyHarmful`涵盖了预定义的消息，如`PoisonPill`和`Kill`，但它也可以作为标记特征添加到用户定义的消息中。

​         警告        

不可信模式本身并不能完全抵御攻击。它使执行恶意或意外操作变得稍微困难一些，但应该注意的是，仍然不应启用[Java 序列化](https://doc-akka-io.translate.goog/docs/akka/current/serialization.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#java-serialization)。在不受信任的网络中运行时，可以通过网络安全（例如防火墙）和/或[通过相互身份验证](https://doc-akka-io.translate.goog/docs/akka/current/remoting-artery.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#remote-tls)启用[TLS](https://doc-akka-io.translate.goog/docs/akka/current/remoting-artery.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#remote-tls)来实现额外保护。

在不信任模式下，默认情况下会丢弃与演员选择一起发送的消息，但可以向配置中定义的特定演员授予接收演员选择消息的权限：

```
akka.remote.artery.trusted-selection-paths = ["/user/receptionist", "/user/namingService"]
```

实际消息必须仍然不是 类型`PossiblyHarmful`。

总之，当通过远程处理层传入时，配置为不可信模式的系统会忽略以下操作：

- 远程部署（这也意味着没有远程监督）
- 远程死亡守望
- `system.stop()`, `PoisonPill`,`Kill`
- 发送从`PossiblyHarmful`标记接口扩展的任何消息，其中包括`Terminated`
- 与参与者选择一起发送的消息，除非目的地在`trusted-selection-paths`.

​         笔记        

启用不可信模式不会取消客户端自由选择其消息发送目标的能力，这意味着不受上述规则禁止的消息可以发送到远程系统中的任何参与者。对于面向客户端的系统来说，最好只包含一组明确定义的入口点 Actor，然后将请求（可能在执行验证之后）转发到另一个包含实际工作人员 Actor 的 Actor 系统。如果这两个服务器端系统之间的消息传递是使用本地完成的`ActorRef`（它们可以在同一个 JVM 中的参与者系统之间安全地交换），您可以通过标记它们来限制此接口上的消息，`PossiblyHarmful`以便客户端无法伪造它们。

## 隔离

Akka 远程处理使用 TCP 或 Aeron 作为底层消息传输。Aeron 正在使用 UDP，并添加了与 TCP 非常相似的可靠传输和会话语义。这意味着消息的顺序被保留，这是[Actor 消息排序保证](https://doc-akka-io.translate.goog/docs/akka/current/general/message-delivery-reliability.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#message-ordering)所必需的。在正常情况下，所有消息都会被传递，但在某些情况下，消息可能不会传递到目的地：

- 在 TCP 连接或 Aeron 会话中断时的网络分区期间，一旦分区结束，这会自动恢复
- 在没有流量控制的情况下发送太多消息从而填满出站发送队列（`outbound-message-queue-size`配置）
- 如果消息的序列化或反序列化失败（只会丢弃该消息）
- 如果远程基础结构中发生意外异常

简而言之，Actor 消息传递是“最多一次”，如[消息传递可靠性中所述](https://doc-akka-io.translate.goog/docs/akka/current/general/message-delivery-reliability.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)

Akka 中的某些消息称为系统消息，并且不能删除这些消息，因为这会导致系统之间的状态不一致。此类消息主要用于两个功能；远程死亡观察和远程部署。这些消息由 Akka  远程处理通过确认每条消息并重新发送未确认的消息来保证“恰好一次”。如果无论如何都无法传递系统消息，则与目标系统的关联将无法恢复，并且会向远程系统上的所有监视参与者发出终止信号。它被置于所谓的隔离状态。如果不使用远程监视或远程部署，通常不会发生隔离。

每个`ActorSystem`实例都有一个唯一标识符 (UID)，当系统使用相同的主机名和端口重新启动时，这对于区分系统的化身很重要。隔离的是特定的化身 (UID)。从此状态恢复的唯一方法是重新启动其中一个参与者系统。

发送到隔离系统和从隔离系统接收的邮件将被丢弃。但是，可以将消息发送`actorSelection`到隔离系统的地址，这对于探测系统是否已重新启动很有用。

在以下情况下，协会将被隔离：

- 集群节点从集群成员中删除。
- Remote failure detector triggers, i.e. remote watch is used. This is different when [Akka Cluster](https://doc-akka-io.translate.goog/docs/akka/current/cluster-usage.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem) is used. The unreachable observation by the cluster failure detector  can go back to reachable if the network partition heals. A cluster  member is not quarantined when the failure detector triggers.
- Overflow of the system message delivery buffer, e.g. because of too many `watch` requests at the same time (`system-message-buffer-size` config).
- Unexpected exception occurs in the control subchannel of the remoting infrastructure.

The UID of the `ActorSystem` is exchanged in a  two-way handshake when the first message is sent to a destination. The  handshake will be retried until the other system replies and no other  messages will pass through until the handshake is completed. If the  handshake cannot be established within a timeout (`handshake-timeout` config) the association is stopped (freeing up resources). Queued  messages will be dropped if the handshake cannot be established. It will not be quarantined, because the UID is unknown. New handshake attempt  will start when next message is sent to the destination.

Handshake requests are actually also sent periodically to be  able to establish a working connection when the destination system has  been restarted.

### Watching Remote Actors

Watching a remote actor is API wise not different than watching a local actor, as described in [Lifecycle Monitoring aka DeathWatch](https://doc-akka-io.translate.goog/docs/akka/current/actors.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#deathwatch). However, it is important to note, that unlike in the local case,  remoting has to handle when a remote actor does not terminate in a  graceful way sending a system message to notify the watcher actor about  the event, but instead being hosted on a system which stopped abruptly  (crashed). These situations are handled by the built-in failure  detector.

### Failure Detector

Under the hood remote death watch uses heartbeat messages and a failure detector to generate `Terminated` message from network failures and JVM crashes, in addition to graceful termination of watched actor.

The heartbeat arrival times is interpreted by an implementation of [The Phi Accrual Failure Detector](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=http://www.jaist.ac.jp/~defago/files/pdf/IS_RR_2004_010.pdf).

The suspicion level of failure is given by a value called *phi*. The basic idea of the phi failure detector is to express the value of *phi* on a scale that is dynamically adjusted to reflect current network conditions.

The value of *phi* is calculated as:

```
phi = -log10(1 - F(timeSinceLastHeartbeat))
```

where F is the cumulative distribution function of a normal  distribution with mean and standard deviation estimated from historical  heartbeat inter-arrival times.

In the [Remote Configuration](https://doc-akka-io.translate.goog/docs/akka/current/remoting-artery.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#remote-configuration-artery) you can adjust the `akka.remote.watch-failure-detector.threshold` to define when a *phi* value is considered to be a failure.

A low `threshold` is prone to generate many false positives but ensures a quick detection in the event of a real crash. Conversely, a high `threshold` generates fewer mistakes but needs more time to detect actual crashes. The default `threshold` is 10 and is appropriate for most situations. However in cloud  environments, such as Amazon EC2, the value could be increased to 12 in  order to account for network issues that sometimes occur on such  platforms.

The following chart illustrates how *phi* increase with increasing time since the previous heartbeat.

![phi1.png](https://doc.akka.io/docs/akka/current/images/phi1.png)

Phi 是根据历史到达时间的平均值和标准偏差计算得出的。上图是标准偏差为 200 毫秒的示例。如果心跳以较小的偏差到达，则曲线变得更陡峭，即可以更快地确定故障。对于 100 ms 的标准偏差，曲线看起来像这样。

![phi2.png](https://doc.akka.io/docs/akka/current/images/phi2.png)

为了能够应对突发的异常情况，例如垃圾收集暂停和瞬时网络故障，故障检测器配置了一个余量，`akka.remote.watch-failure-detector.acceptable-heartbeat-pause`。您可能需要根据您的环境调整它的[远程配置](https://doc-akka-io.translate.goog/docs/akka/current/remoting-artery.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#remote-configuration-artery)。这是曲线在`acceptable-heartbeat-pause`配置为 3 秒时的样子。

![phi3.png](https://doc.akka.io/docs/akka/current/images/phi3.png)

## 序列化

您需要为actor 消息启用[序列化](https://doc-akka-io.translate.goog/docs/akka/current/serialization.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)。在许多情况下，[使用 Jackson 进行序列化](https://doc-akka-io.translate.goog/docs/akka/current/serialization-jackson.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)是一个不错的选择，如果您没有其他偏好，我们建议您这样做。



### 基于 ByteBuffer 的序列化

Artery 引入了一种新的序列化机制，它允许`ByteBufferSerializer`直接写入共享`java.nio.ByteBuffer`而不是被迫`Array[Byte]`为每个序列化消息分配和返回一个。对于高吞吐量消息传递，此 API 更改可以产生显着的性能优势，因此我们建议更改您的序列化程序以使用此新机制。

这个新 API 还可以很好地与新版本的 Google Protocol Buffers 和其他序列化库配合使用，这些库获得了直接与 ByteBuffers 进行序列化的能力。

由于新功能只改变了字节的读写方式，而序列化基础设施的其余部分保持不变，我们建议先阅读[序列化](https://doc-akka-io.translate.goog/docs/akka/current/serialization.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)文档。

实现一个`akka.serialization.ByteBufferSerializer`与任何其他序列化程序相同的工作方式，

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/remoting-artery.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/remoting-artery.html#tab1)

因此，为 Artery 实现序列化器就像实现这个接口一样简单，并像往常一样绑定序列化器（在[Serialization 中进行了](https://doc-akka-io.translate.goog/docs/akka/current/serialization.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)解释）。

实现通常应该扩展，`SerializerWithStringManifest`并且除了`ByteBuffer`基于`toBinary`和`fromBinary`方法之外，还实现基于数组的`toBinary`和`fromBinary`方法。不使用时将使用基于数组的方法`ByteBuffer`，例如在 Akka Persistence 中。

请注意，基于数组的方法可以通过委托来实现，如下所示：

- [          斯卡拉         ](https://doc.akka.io/docs/akka/current/remoting-artery.html#tab0)

  ​          ``

- [          爪哇         ](https://doc.akka.io/docs/akka/current/remoting-artery.html#tab1)

## 具有远程目标的路由器

将远程处理与[路由](https://doc-akka-io.translate.goog/docs/akka/current/routing.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)结合起来是绝对可行的。

远程部署的路由池可以配置为：

```

来源akka.actor.deployment {
  /parent/remotePool {
    router = round-robin-pool
    nr-of-instances = 10
    target.nodes = ["tcp://app@10.0.0.2:2552", "akka://app@10.0.0.3:2552"]
  }
}
```

此配置设置将克隆的定义演员`Props`的的`remotePool`10倍，部署在两个给定的目标节点分布均匀。

使用远程部署的路由池时，您必须确保所有参数`Props`都可以[序列化](https://doc-akka-io.translate.goog/docs/akka/current/serialization.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)。

一组远程参与者可以配置为：

```

来源akka.actor.deployment {
  /parent/remoteGroup2 {
    router = round-robin-group
    routees.paths = [
      "akka://app@10.0.0.1:2552/user/workers/w1",
      "akka://app@10.0.0.2:2552/user/workers/w1",
      "akka://app@10.0.0.3:2552/user/workers/w1"]
  }
}
```

此配置设置将向定义的远程参与者路径发送消息。它要求您在具有匹配路径的远程节点上创建目标参与者。这不是由路由器完成的。

## 动脉有什么新鲜事

Artery 是旧远程模块的重新实现，旨在提高性能和稳定性。它主要与旧实现的源代码兼容，并且在许多情况下是直接替代品。与以前的实现相比，动脉的主要特点：

- 基于 Akka Streams TCP/TLS 或[Aeron](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://github.com/real-logic/Aeron) (UDP) 而不是 Netty TCP
- 专注于高吞吐量、低延迟的通信
- 通过使用专用子通道，将内部控制消息与用户消息隔离，从而提高稳定性并减少流量大时错误的故障检测。
- 主要是免分配操作
- 支持大消息的单独子通道，以避免干扰较小的消息
- 压缩线上的actor路径以减少较小消息的开销
- 支持直接使用 ByteBuffers 进行更快的序列化/反序列化
- 内置 Java 飞行记录器 (JFR) 可帮助调试实施问题，而不会因实施特定事件而污染用户日志
- 提供跨主要 Akka 版本的协议稳定性，以支持大规模系统的滚动更新

与之前实现的主要不兼容更改是 an 的字符串表示的协议字段`ActorRef`始终是*akka*而不是之前使用的*akka.tcp*或*akka.ssl.tcp*。配置属性也不同。

## 性能调优

### 车道

消息序列化和反序列化可能是远程通信的瓶颈。因此，支持并行入站和出站通道以并行执行不同目标参与者的序列化和其他任务。使用多个通道对于入站消息最有价值，因为来自所有远程系统的所有入站消息共享相同的入站流。对于出站消息，每个远程目标系统已经有一个流，因此多个出站通道只会在发送到同一目标系统中的不同参与者时增加价值。

通道的选择基于接收者 ActorRef 的一致散列，以保留每个接收者的消息排序。

请注意，由于多个通道引入了异步边界`inbound-lanes=1`，`outbound-lanes=1`因此可以实现最低延迟。

另请注意，并行任务的总量受 约束，`remote-dispatcher`线程池大小不应超过 CPU 内核数减去应用程序中实际处理消息的余量，即实际上池大小应小于核心数。

见`inbound-lanes`和`outbound-lanes`在[参考配置](https://doc-akka-io.translate.goog/docs/akka/current/general/configuration-reference.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#config-akka-remote-artery)的默认值。

### 大消息专用子通道

用户定义的远程参与者之间的所有通信都与 Akka 内部消息的通道隔离，因此大型用户消息无法阻止紧急系统消息。虽然这为 Akka  服务提供了良好的隔离，但默认情况下所有用户通信都通过共享网络连接发生。当一些参与者发送大消息时，这可能导致其他消息遭受更高的延迟，因为他们需要等到完整的消息在共享通道上传输（因此，共享瓶颈）。在这些情况下，将具有不同 QoS 要求的参与者分开通常是有帮助的：大消息与低延迟。

如果配置，Akka 远程处理会为大消息提供专用通道。由于不能违反*参与者*消息顺序，因此通道实际上专用于*参与者*而不是消息，以确保所有消息都按发送顺序到达。通过使用必须在发送方和接收方的参与者系统配置中指定的路径模式，可以在给定路径上分配参与者以使用此专用通道：

```
akka.remote.artery.large-message-destinations = [
   "/user/largeMessageActor",
   "/user/largeMessagesGroup/*",
   "/user/anotherGroup/*/largeMesssages",
   "/user/thirdGroup/**",
]
```

这意味着发送给以下参与者的所有消息都将通过专用的大型消息通道：

- `/user/largeMessageActor`
- `/user/largeMessageActorGroup/actor1`
- `/user/largeMessageActorGroup/actor2`
- `/user/anotherGroup/actor1/largeMessages`
- `/user/anotherGroup/actor2/largeMessages`
- `/user/thirdGroup/actor3/`
- `/user/thirdGroup/actor4/actor5`

发往与这些模式不匹配的参与者的消息像以前一样使用默认通道发送。

要注意大消息，您可以启用负载大小（以字节为单位）大于配置的消息类型的日志记录`log-frame-size-exceeding`。

```
akka.remote.artery {
  log-frame-size-exceeding = 10000b
}
```

示例日志消息：

```
[INFO] Payload size for [java.lang.String] is [39068] bytes. Sent to Actor[akka://Sys@localhost:53039/user/destination#-1908386800]
[INFO] New maximum payload size for [java.lang.String] is [44068] bytes. Sent to Actor[akka://Sys@localhost:53039/user/destination#-1908386800].
```

大消息通道仍然不能用于超大消息，每条消息最多几 MB。另一种方法是使用[Reliable Delivery](https://doc-akka-io.translate.goog/docs/akka/current/typed/reliable-delivery.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem)，它支持自动[拆分大消息](https://doc-akka-io.translate.goog/docs/akka/current/typed/reliable-delivery.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#chunk-large-messages)并在接收端重新组合它们。

### 外部共享 Aeron 媒体驱动程序

Aeron 传输在所谓的[媒体驱动程序中运行](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://github.com/real-logic/Aeron/wiki/Media-Driver-Operation)。默认情况下，Akka 启动嵌入在与应用程序相同的 JVM 进程中的媒体驱动程序。这很方便，并且通过只需要启动和监控一个进程来简化操作问题。

媒体驱动程序可能会使用相当多的 CPU 资源。如果您在同一台机器上运行多个 Akka 应用程序 JVM，那么通过将其作为单独的进程运行来共享媒体驱动程序是明智的。

媒体驱动程序还具有与普通应用程序不同的资源使用特性，因此将媒体驱动程序作为单独的进程运行会更高效和稳定。

鉴于 Aeron jar 文件位于类路径中，可以使用以下命令启动独立媒体驱动程序：

```
java io.aeron.driver.MediaDriver
```

所需的类路径：

```
Agrona-0.5.4.jar:aeron-driver-1.0.1.jar:aeron-client-1.0.1.jar
```

您可以在[Maven Central](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://search.maven.org/)上找到这些 jar 文件，或者您可以使用首选构建工具创建一个包。

您可以将[Aeron 属性](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://github.com/real-logic/Aeron/wiki/Configuration-Options)作为命令行*-D*系统属性传递：

```
-Daeron.dir=/dev/shm/aeron
```

您还可以在文件中定义 Aeron 属性：

```
java io.aeron.driver.MediaDriver config/aeron.properties
```

此类属性文件的示例：

```
aeron.mtu.length=16384
aeron.socket.so_sndbuf=2097152
aeron.socket.so_rcvbuf=2097152
aeron.rcv.buffer.length=16384
aeron.rcv.initial.window.length=2097152
agrona.disable.bounds.checks=true

aeron.threading.mode=SHARED_NETWORK

# low latency settings
#aeron.threading.mode=DEDICATED
#aeron.sender.idle.strategy=org.agrona.concurrent.BusySpinIdleStrategy
#aeron.receiver.idle.strategy=org.agrona.concurrent.BusySpinIdleStrategy

# use same director in akka.remote.artery.advanced.aeron-dir config
# of the Akka application
aeron.dir=/dev/shm/aeron
```

在[Aeron 文档中](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://github.com/real-logic/Aeron/wiki/Media-Driver-Operation)阅读有关媒体驱动程序的更多信息。

要使用 Akka 应用程序中的外部媒体驱动程序，您需要定义以下两个配置属性：

```
akka.remote.artery.advanced.aeron {
  embedded-media-driver = off
  aeron-dir = /dev/shm/aeron
}
```

在`aeron-dir`必须与你启动媒体驱动器的目录相匹配，即在`aeron.dir`财产。

然后可以通过指向同一目录将多个 Akka 应用程序配置为使用相同的媒体驱动程序。

请注意，如果媒体驱动程序进程停止，正在使用它的 Akka 应用程序也将停止。

### Aeron 调校

请参阅有关[性能测试的](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://github.com/real-logic/Aeron/wiki/Performance-Testing)Aeron 文档。

### 微调 CPU 使用延迟权衡

Artery 专为低延迟而设计，因此当系统大部分时间处于空闲状态时，它可能会占用 CPU。这并不总是可取的。使用 Aeron 传输时，可以使用以下配置调整 CPU 使用率和延迟之间的权衡：

```
# Values can be from 1 to 10, where 10 strongly prefers low latency
# and 1 strongly prefers less CPU usage
akka.remote.artery.advanced.aeron.idle-cpu-level = 1
```

通过将此值设置为较低的数字，它告诉 Akka 在其专用于[自旋等待的](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://en.wikipedia.org/wiki/Busy_waiting)线程上执行更长的“休眠”时间，从而在没有立即执行的任务时减少 CPU 负载，但代价是对某个任务的反应时间更长。事件实际发生时。值得注意的是，在持续高吞吐量期间，此设置没有太大区别，因为线程主要有要执行的任务。这也意味着在高吞吐量下（但低于最大容量），系统的延迟可能比低消息速率下的更少。



## 远程配置

Akka 中有许多与远程处理相关的配置属性。我们参考[参考配置](https://doc-akka-io.translate.goog/docs/akka/current/general/configuration-reference.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#config-akka-remote-artery)了解更多信息。

​         笔记        

以编程方式设置诸如侦听 IP 和端口号之类的属性最好使用以下内容：

```

来源ConfigFactory.parseString("akka.remote.artery.canonical.hostname=\"1.2.3.4\"")
    .withFallback(ConfigFactory.load());
```



### Akka 在 NAT 后面或在 Docker 容器中

在涉及网络地址转换 (NAT)、负载均衡器或 Docker 容器的设置中，Akka 绑定到的主机名和端口对将不同于用于从外部连接到系统的“逻辑”主机名和端口对。这需要特殊的配置来设置远程处理的逻辑和绑定对。

```
akka {
  remote {
    artery {
      canonical.hostname = my.domain.com      # external (logical) hostname
      canonical.port = 8000                   # external (logical) port

      bind.hostname = local.address # internal (bind) hostname
      bind.port = 25520              # internal (bind) port
    }
 }
}
```

您可以[使用 docker-compose 示例项目](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://developer.lightbend.com/start/?group%3Dakka%26project%3Dakka-sample-cluster-docker-compose-scala)查看[集群，](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://developer.lightbend.com/start/?group%3Dakka%26project%3Dakka-sample-cluster-docker-compose-scala)以了解它在实践中的样子。 

### 在 Docker/Kubernetes 中运行

`aeron-udp`在容器化环境中使用时，必须特别注意媒体驱动程序在 ram 磁盘上运行。默认情况下，它位于在`/dev/shm`大多数物理 Linux 机器上将安装为系统内存大小的一半。

Docker 和 Kubernetes 挂载了一个 64Mb 的 ram 磁盘。这不太可能足够大。对于 docker，这可以用`--shm-size="512mb"`.

在 Kubernetes 中，（还）没有直接支持设置`shm`大小。而是将`EmptyDir`with 类型安装`Memory`到`/dev/shm`例如 deployment.yml 中：

```
spec:
  containers:
  - name: artery-udp-cluster
    // rest of container spec...
    volumeMounts:
    - mountPath: /dev/shm
      name: media-driver
  volumes:
  - name: media-driver
    emptyDir:
      medium: Memory
      name: media-driver
```

目前没有办法限制内存空目录的大小，但有一个添加它的[拉取请求](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://github.com/kubernetes/kubernetes/pull/63641)。

挂载中使用的任何空间都将计入容器的内存使用量。

### 飞行记录仪

在 JDK 11 Artery 上运行时，可以通过[Java Flight Recorder (JFR)](https://translate.google.com/website?sl=auto&tl=zh-CN&ajax=1&elem=1&se=1&u=https://openjdk.java.net/jeps/328)获得特定的飞行记录。飞行记录器通过检测 JDK 11 自动启用，但如果需要，可以通过设置`akka.java-flight-recorder.enabled = false`.

默认情况下，启用 JFR 时会发出低开销动脉特定事件，高开销事件需要自定义设置模板，并且不会通过`profiling`JFR 模板自动启用。要启用这些，请创建`profiling`模板的副本并启用所有`Akka`子类别事件，例如通过 JMC GUI。

----

[经典远程处理（Deplicated） ](https://doc-akka-io.translate.goog/docs/akka/current/remoting.html)         

----

[英文原文](https://doc.akka.io/docs/akka/current/remoting-artery.html)