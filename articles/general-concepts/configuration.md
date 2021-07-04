# 配置
你可以在不定义任何配置的情况下开始使用 Akka，因为提供了合理的默认值。稍后，你可能需要修改设置以更改默认行为或适应特定的运行时环境。你可以修改的典型设置示例：

- [日志级别和日志记录器后端](../typed/logging.md)
- [启用集群理](../typed/cluster.md)
- [消息序列化](../cluster/serialization.md)
- [调度器优化](../typed/dispatchers.md)

Akka 使用「[Typesafe Config Library](https://github.com/lightbend/config)」，这对于配置你自己的应用程序或使用或不使用 Akka 构建的库也是一个不错的选择。这个库是用 Java 实现的，没有外部依赖关系；你应该看看它的文档，特别是关于「[Config Library 文档](https://github.com/lightbend/config/blob/master/README.md)」。

## 从哪里读取配置？
Akka 的所有配置（`configuration`）都保存在`ActorSystem`实例中，或者换句话说，从外部看，`ActorSystem`是配置信息的唯一消费者。在构造 Actor 系统时，可以传入`Config`对象，也可以不传入，其中第二种情况等同于传递`ConfigFactory.load()`（使用正确的类加载器）。这意味着默认解析是类路径根目录下的所有`application.conf`、`application.json`和`application.properties`文件。有关详细信息，请参阅上述文档。然后，Actor 系统合并在类路径根目录下找到的所有`reference.conf`资源，以形成最终的配置，以供内部使用。

```java
appConfig.withFallback(ConfigFactory.defaultReference(classLoader))
```
其原理是代码从不包含默认值，而是依赖于存在相关库提供的`reference.conf`中的值。

作为系统属性给出的覆盖具有最高优先级，请参阅「[HOCON](https://github.com/lightbend/config/blob/master/HOCON.md)」规范（靠近底部）。另外值得注意的是，应用程序的默认配置可以使用`config.resource`属性覆盖，还有更多内容，请参阅「[Config](https://github.com/typesafehub/config/blob/master/README.md)」文档。

- **注释**：如果你正在编写 Akka 应用程序，请将你的配置保存在类路径根目录下的`application.conf`中。如果你正在编写基于 Akka 的库，请将其配置保存在 JAR 文件根目录下的`reference.conf`中。

## 使用 JarJar、OneJar、Assembly 或任何  jar-bundler 时

- **警告**：Akka 的配置方法很大程度上依赖于每个`module/jar`都有自己的`reference.conf`文件，所有这些都将被配置发现并加载。不幸的是，这也意味着如果你将多个 Jar 放入或合并到同一个 Jar 中，那么你还需要合并所有`reference.conf`。否则，所有默认值将丢失，Akka 将不起作用。

有关如何在捆绑时合并`reference.conf`资源的信息，请参阅[部署文档](../additional/deploy.md)。

## 自定义 application.conf
一个自定义的`application.conf`配置可能如下所示：

```json
# In this file you can override any option defined in the reference files.
# Copy in parts of the reference files and modify as you please.

akka {

  # Logger config for Akka internals and classic actors, the new API relies
  # directly on SLF4J and your config for the logger backend.

  # Loggers to register at boot time (akka.event.Logging$DefaultLogger logs
  # to STDOUT)
  loggers = ["akka.event.slf4j.Slf4jLogger"]

  # Log level used by the configured loggers (see "loggers") as soon
  # as they have been started; before that, see "stdout-loglevel"
  # Options: OFF, ERROR, WARNING, INFO, DEBUG
  loglevel = "DEBUG"

  # Log level for the very basic logger activated during ActorSystem startup.
  # This logger prints the log messages to stdout (System.out).
  # Options: OFF, ERROR, WARNING, INFO, DEBUG
  stdout-loglevel = "DEBUG"

  # Filter of log events that is used by the LoggingAdapter before
  # publishing log events to the eventStream.
  logging-filter = "akka.event.slf4j.Slf4jLoggingFilter"

  actor {
    provider = "cluster"

    default-dispatcher {
      # Throughput for default Dispatcher, set to 1 for as fair as possible
      throughput = 10
    }
  }

  remote.artery {
    # The port clients should connect to.
    canonical.port = 4711
  }
}
```
## 包含文件
有时，包含另一个配置文件可能很有用，例如，如果你有一个`application.conf`，具有所有与环境无关的设置，然后覆盖特定环境的某些设置。

使用`-Dconfig.resource=/dev.conf`指定系统属性将加载`dev.conf`文件，其中包括`application.conf`

- **dev.conf**

```
include "application"

akka {
  loglevel = "DEBUG"
}
```
[HOCON](https://github.com/typesafehub/config/blob/master/HOCON.md) 规范中解释了更高级的包含和替换机制。

## 配置日志
如果系统或配置属性`akka.log-config-on-start`设置为`on`，那么当 Actor 系统启动时，则将完整的配置打印在 INFO 级别。当你不确定使用了什么配置时，这很有用。

如果有疑问，你可以在使用配置对象构建 Actor 系统之前或之后检查它们：

```sh
Welcome to Scala 2.12 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0).
Type in expressions to have them evaluated.
Type :help for more information.

scala> import com.typesafe.config._
import com.typesafe.config._

scala> ConfigFactory.parseString("a.b=12")
res0: com.typesafe.config.Config = Config(SimpleConfigObject({"a" : {"b" : 12}}))

scala> res0.root.render
res1: java.lang.String =
{
    # String: 1
    "a" : {
        # String: 1
        "b" : 12
    }
}
```

每个项目前面的注释给出了有关设置起点（文件和行号）的详细信息以及可能出现的注释，例如在参考配置中。合并引用和 Actor 系统的设置可以如下打印：

```scala
val system = ActorSystem(rootBehavior, "MySystem")
system.logConfiguration()
```
## 关于类加载器的额外话题
在配置文件的几个地方，可以指定由 Akka 实例化的某个对象的完全限定类名。这是使用 Java 反射完成的，Java 反射又使用类加载器。在应用程序容器或 OSGi 包等具有挑战性的环境中获得正确的方法并不总是很简单的，Akka 的当前方法是，每个`ActorSystem`实现都保存当前线程的上下文类加载器（如果可用，否则只存储其自己的加载器，如`this.getClass.getClassLoader`）并将其用于所有反射访问。这意味着将 Akka 放在根类路径上会从产生`NullPointerException`：这种奇怪的地方是不支持的。

## 应用程序特定设置
配置也可用于特定于应用程序的设置。一个好的做法是将这些设置放在「[扩展](https://doc.akka.io/docs/akka/current/extending-akka.html#extending-akka-settings)」中。

### 配置多个 ActorSystem
如果你有多个`ActorSystem`（或者你正在编写一个库，并且有一个`ActorSystem`可能与应用程序的`ActorSystem`分离），那么你可能希望将每个系统的配置分开。

考虑到`ConfigFactory.load()`合并了整个类路径中具有匹配名称的所有资源，利用该功能区分配置层次结构中的 Actor 系统是最容易：

```
myapp1 {
  akka.loglevel = "WARNING"
  my.own.setting = 43
}
myapp2 {
  akka.loglevel = "ERROR"
  app2.setting = "appname"
}
my.own.setting = 42
my.other.setting = "hello"
```

```scala
val config = ConfigFactory.load()
val app1 = ActorSystem(rootBehavior, "MyApp1", config.getConfig("myapp1").withFallback(config))
val app2 = ActorSystem(rootBehavior, "MyApp2", config.getConfig("myapp2").withOnlyPath("akka").withFallback(config))
```

这两个示例演示了“提取子树（`lift-a-subtree`）”技巧之间的不同：在第一个例子中，从 Actor 系统中访问的配置是：

```
akka.loglevel = "WARNING"
my.own.setting = 43
my.other.setting = "hello"
// plus myapp1 and myapp2 subtrees
// （译者：提取了 `myapp1`的配置和共享配置，并且`myapp1`覆盖共享配置中的同名项）
```
在第二种情况下，只提取“akka”子树，结果如下：

```
akka.loglevel = "ERROR"
my.own.setting = 42
my.other.setting = "hello"
// plus myapp1 and myapp2 subtrees
// （译者：只提取特定配置中的 `akka.loglevel` 和共享配置，并且`myappN`中的同名配置互相覆盖）
```
- **注释**：配置库非常强大，说明其所有功能超出了本文的范围。尤其不包括如何在其他文件中包含其他配置文件（参见「[包含文件](#包含文件)」中的一个小示例）以及通过路径替换复制配置树的部分。

在实例化`ActorSystem`时，还可以通过其他方式以编程方式指定和分析配置。

```scala
import akka.actor.typed.ActorSystem
import com.typesafe.config.ConfigFactory
val customConf = ConfigFactory.parseString("""
  akka.log-config-on-start = on
""")
// ConfigFactory.load sandwiches customConfig between default reference
// config and default overrides, and then resolves it.
val system = ActorSystem(rootBehavior, "MySystem", ConfigFactory.load(customConf))
```

## 从自定义位置读取配置
你可以在代码或使用系统属性中替换或补充`application.conf`。

如果你使用的是`ConfigFactory.load()`（默认情况下 Akka 会这样做），那么可以通过定义`-Dconfig.resource=whatever`、`-Dconfig.file=whatever`或`-Dconfig.url=whatever`来替换`application.conf`。

在用`-Dconfig.resource`和`friends`指定的替换文件中，如果你仍然想使用`application.{conf,json,properties}`，可以使用`include "application"`。在`include "application"`之前指定的设置将被包含的文件覆盖，而在`include "application"`之后指定的设置将覆盖包含的文件。

在代码中，有许多自定义选项。

`ConfigFactory.load()`有几个重载；这些重载允许你指定一些夹在系统属性（重写）和默认值（来自`reference.conf`）之间的内容，替换常规`application.{conf,json,properties}`并替换` -Dconfig.file`和它的朋友们。

`ConfigFactory.load()`的最简单变体采用资源基名称（`resource basename`），而不是应用程序；然后将使用`myname.conf`、`myname.json`和`myname.properties`而不是`application.{conf,json,properties}`。

最灵活的变体采用`Config`对象，你可以使用`ConfigFactory`中的任何方法加载该对象。例如，你可以使用`ConfigFactory.parseString()`在代码中放置配置字符串，也可以制作映射和`ConfigFactory.parseMap()`，或者加载文件。

你还可以将自定义配置与常规配置结合起来，这可能看起来像：

```scala
// make a Config with just your special setting
val myConfig = ConfigFactory.parseString("something=somethingElse");
// load the normal config stack (system props,
// then application.conf, then reference.conf)
val regularConfig = ConfigFactory.load();
// override regular stack with myConfig
val combined = myConfig.withFallback(regularConfig);
// put the result in between the overrides
// (system props) and defaults again
val complete = ConfigFactory.load(combined);
// create ActorSystem
val system = ActorSystem(rootBehavior, "myname", complete);
```
使用`Config`对象时，请记住蛋糕中有三个“层”：

- `ConfigFactory.defaultOverrides()`（系统属性）
- 应用程序的设置
- `ConfigFactory.defaultReference()`（`reference.conf`）

通常的目标是定制中间层，而不让其他两层单独使用。

- `ConfigFactory.load()`加载整个堆栈
- `ConfigFactory.load()`的重载允许你指定不同的中间层
- `ConfigFactory.parse()`变体加载单个文件或资源

要堆叠两层，请使用`override.withFallback(fallback)`；尝试将系统属性（`defaultOverrides()`）保持在顶部，并将`reference.conf`（`defaultReference()`）保持在底部。

请记住，你通常可以在`application.conf`中添加另一个`include`语句，而不是编写代码。`application.conf`顶部的`include`将被`application.conf`的其余部分覆盖，而底部的`include`将覆盖前面的内容。

## 参考配置列表
每个 Akka 模块都有一个带有默认值的参考配置文件。这些`reference.conf`文件列在[默认配置](configuration-reference.md)中

----------

[默认配置 ](configuration-reference.md)

----------
**英文原文链接**：[Configuration](https://doc.akka.io/docs/akka/current/general/configuration.html).