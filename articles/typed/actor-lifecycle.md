# Actor 生命周期
## 依赖

为了使用 Akka Actor 类型，你需要将以下依赖添加到你的项目中：

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

## 介绍

Actor 是一种有状态的资源，必须显式启动和停止。

需要注意的是，Actor 在不再被引用时不会自动停止，每个创建的 Actor 也必须显式销毁。唯一的简化是停止父 Actor 时也会递归地停止该父级创建的所有子 Actor。`ActorSystem`关闭时，所有Actor也会自动停止。

```
提示:

ActorSystem 是一种用于分配线程的重量级结构，所以请为每个逻辑应用程序创建一个。通常每个JVM 进程只需要一个 ActorSystem。
```

## 创建Actors

一个 Actor 可以创建或生成任意数量的子 Actor，而子 Actor 又可以生成自己的子 Actor，从而形成一个 Actor 层次。「[ActorSystem](https://doc.akka.io/japi/akka/2.5/?akka/actor/typed/ActorSystem.html)」承载层次结构，并且在`ActorSystem`层次结构的顶部只能有一个根 Actor。一个子 Actor 的生命周期是与其父 Actor 联系在一起的，一个子 Actor 可以在任何时候停止自己或被停止，但永远不能比父 Actor 活得更久。

### ActorContext

可以出于多种目的访问 ActorContext，例如：

- 监督和产生子Actor
- 观察其它Actor并接收`Terminated(otherActor)`事件当被观察的Actor永久停止时。
- 记录日志
- 创建消息适配器
- 与另一个Actor的请求-响应交互（ask）
- 访问`self` ActorRef

如果一个行为需要使用`ActorContext`，例如使用 `context.self`产生子actor，可以通过在`Behaviors.setup`中使用包装构造来获得：

```scala
object HelloWorldMain {

  final case class SayHello(name: String)

  def apply(): Behavior[SayHello] =
    Behaviors.setup { context =>
      val greeter = context.spawn(HelloWorld(), "greeter")

      Behaviors.receiveMessage { message =>
        val replyTo = context.spawn(HelloWorldBot(max = 3), message.name)
        greeter ! HelloWorld.Greet(message.name, replyTo)
        Behaviors.same
      }
    }
}
```

#### ActorContext 的线程安全性

`ActorContext`中的许多方法不是线程安全的，并且

- 不能被`scala.concurrent.Future`中的回调线程访问
- 不得在多个 actor 实例之间共享
- 只能在普通actor消息处理线程中使用

### 守护者 Actor
顶级 Actor，也称为 user 守护者 Actor，与`ActorSystem`一起创建。发送到 Actor 系统的消息被定向到根 Actor。根 Actor 是由用于创建`ActorSystem`的行为生成的，例如下面的示例中名为`HelloWorldMain`的Actor：

```java
val system: ActorSystem[HelloWorldMain.SayHello] =
  ActorSystem(HelloWorldMain(), "hello")

system ! HelloWorldMain.SayHello("World")
system ! HelloWorldMain.SayHello("Akka")
```

对于非常简单的应用程序，监护人可能包含实际的应用程序逻辑和消息处理。但是一旦应用程序将处理多个问题，监护人就应该只负责引导，生成子系统，并监视它们的生命周期。

一旦监护人Actor停止，这也将停止`ActorSystem`.

当`ActorSystem.terminate`被调用时，[协调关闭](coordinated-shutdown.md)流程将按特定顺序停止Actor和服务。







### 繁衍子级

子 Actor 是由「[ActorContext](https://doc.akka.io/japi/akka/2.5/?akka/actor/typed/javadsl/ActorContext.html)」的“繁殖”产生的。在下面的示例中，当根 Actor 启动时，它生成一个由行为`HelloWorld.greeter`描述的子 Actor。此外，当根 Actor 收到`Start`消息时，它将创建由行为`HelloWorldBot.bot`定义的子 Actor：

```java
public abstract static class HelloWorldMain {
  private HelloWorldMain() {}

  public static class Start {
    public final String name;

    public Start(String name) {
      this.name = name;
    }
  }

  public static final Behavior<Start> main =
      Behaviors.setup(
          context -> {
            final ActorRef<HelloWorld.Greet> greeter =
                context.spawn(HelloWorld.greeter, "greeter");

            return Behaviors.receiveMessage(
                message -> {
                  ActorRef<HelloWorld.Greeted> replyTo =
                      context.spawn(HelloWorldBot.bot(0, 3), message.name);
                  greeter.tell(new HelloWorld.Greet(message.name, replyTo));
                  return Behaviors.same();
                });
          });
}
```

要在生成 Actor 时指定调度器，请使用「[DispatcherSelector](https://doc.akka.io/japi/akka/2.5/?akka/actor/typed/DispatcherSelector.html)」。如果未指定，则 Actor 将使用默认调度器，有关详细信息，请参阅「[默认调度器](https://doc.akka.io/docs/akka/current/dispatchers.html#default-dispatcher)」。

```java
public static final Behavior<Start> main =
    Behaviors.setup(
        context -> {
          final String dispatcherPath = "akka.actor.default-blocking-io-dispatcher";

          Props props = DispatcherSelector.fromConfig(dispatcherPath);
          final ActorRef<HelloWorld.Greet> greeter =
              context.spawn(HelloWorld.greeter, "greeter", props);

          return Behaviors.receiveMessage(
              message -> {
                ActorRef<HelloWorld.Greeted> replyTo =
                    context.spawn(HelloWorldBot.bot(0, 3), message.name);
                greeter.tell(new HelloWorld.Greet(message.name, replyTo));
                return Behaviors.same();
              });
        });
```

通过参阅「[Actors](https://doc.akka.io/docs/akka/current/typed/actors.html#introduction)」，以了解上述示例。

### SpawnProtocol

守护者 Actor 应该负责初始化任务并创建应用程序的初始 Actor，但有时你可能希望从守护者 Actor 的外部生成新的 Actor。例如，每个 HTTP 请求创建一个 Actor。

这并不难你在行为中的实现，但是由于这是一个常见的模式，因此有一个预定义的消息协议和行为的实现。它可以用作`ActorSystem`的守护者 Actor，可能与`Behaviors.setup`结合使用以启动一些初始任务或 Actor。然后，可以通过调用`SpawnProtocol.Spawn`从外部启动子 Actor，生成到系统的 Actor 引用。使用`ask`时，这类似于`ActorSystem.actorOf`如何在非类型化 Actor 中使用，不同之处在于`ActorRef`的`CompletionStage`返回。

守护者行为可定义为：

```java
import akka.actor.typed.Behavior;
import akka.actor.typed.SpawnProtocol;
import akka.actor.typed.javadsl.Behaviors;

public abstract static class HelloWorldMain {
  private HelloWorldMain() {}

  public static final Behavior<SpawnProtocol> main =
      Behaviors.setup(
          context -> {
            // Start initial tasks
            // context.spawn(...)

            return SpawnProtocol.behavior();
          });
}
```

而`ActorSystem`可以用这种`main`行为来创建，并生成其他 Actor：

```java
import akka.actor.typed.ActorRef;
import akka.actor.typed.ActorSystem;
import akka.actor.typed.Props;
import akka.actor.typed.javadsl.AskPattern;
import akka.util.Timeout;

final ActorSystem<SpawnProtocol> system = ActorSystem.create(HelloWorldMain.main, "hello");
final Duration timeout = Duration.ofSeconds(3);

CompletionStage<ActorRef<HelloWorld.Greet>> greeter =
    AskPattern.ask(
        system,
        replyTo ->
            new SpawnProtocol.Spawn<>(HelloWorld.greeter, "greeter", Props.empty(), replyTo),
        timeout,
        system.scheduler());

Behavior<HelloWorld.Greeted> greetedBehavior =
    Behaviors.receive(
        (context, message) -> {
          context.getLog().info("Greeting for {} from {}", message.whom, message.from);
          return Behaviors.stopped();
        });

CompletionStage<ActorRef<HelloWorld.Greeted>> greetedReplyTo =
    AskPattern.ask(
        system,
        replyTo -> new SpawnProtocol.Spawn<>(greetedBehavior, "", Props.empty(), replyTo),
        timeout,
        system.scheduler());

greeter.whenComplete(
    (greeterRef, exc) -> {
      if (exc == null) {
        greetedReplyTo.whenComplete(
            (greetedReplyToRef, exc2) -> {
              if (exc2 == null) {
                greeterRef.tell(new HelloWorld.Greet("Akka", greetedReplyToRef));
              }
            });
      }
    });
```

还可以在 Actor 层次结构的其他位置使用`SpawnProtocol`。它不必是根守护者 Actor。

## 停止 Actors

Actor 可以通过返回`Behaviors.stopped`作为下一个行为来停止自己。

通过使用父 Actor 的`ActorContext`的`stop`方法，在子 Actor 完成当前消息的处理后，可以强制停止子 Actor。只有子 Actor 才能这样被阻止。

子 Actor 将作为父级关闭过程的一部分被停止。

停止 Actor 产生的`PostStop`信号可用于清除资源。请注意，可以选择将处理此类`PostStop`信号的行为定义为`Behaviors.stopped`的参数。如果 Actor 在突然停止时能优雅地停止自己，则需要不同的操作。

下面是一个示例：

```java
import java.util.concurrent.TimeUnit;

import akka.actor.typed.ActorSystem;
import akka.actor.typed.Behavior;
import akka.actor.typed.PostStop;
import akka.actor.typed.javadsl.Behaviors;


public abstract static class JobControl {
  // no instances of this class, it's only a name space for messages
  // and static methods
  private JobControl() {}

  static interface JobControlLanguage {}

  public static final class SpawnJob implements JobControlLanguage {
    public final String name;

    public SpawnJob(String name) {
      this.name = name;
    }
  }

  public static final class GracefulShutdown implements JobControlLanguage {

    public GracefulShutdown() {}
  }

  public static final Behavior<JobControlLanguage> mcpa =
      Behaviors.receive(JobControlLanguage.class)
          .onMessage(
              SpawnJob.class,
              (context, message) -> {
                context.getSystem().log().info("Spawning job {}!", message.name);
                context.spawn(Job.job(message.name), message.name);
                return Behaviors.same();
              })
          .onSignal(
              PostStop.class,
              (context, signal) -> {
                context.getSystem().log().info("Master Control Programme stopped");
                return Behaviors.same();
              })
          .onMessage(
              GracefulShutdown.class,
              (context, message) -> {
                context.getSystem().log().info("Initiating graceful shutdown...");

                // perform graceful stop, executing cleanup before final system termination
                // behavior executing cleanup is passed as a parameter to Actor.stopped
                return Behaviors.stopped(
                    () -> {
                      context.getSystem().log().info("Cleanup!");
                    });
              })
          .onSignal(
              PostStop.class,
              (context, signal) -> {
                context.getSystem().log().info("Master Control Programme stopped");
                return Behaviors.same();
              })
          .build();
}

public static class Job {
  public static Behavior<JobControl.JobControlLanguage> job(String name) {
    return Behaviors.receiveSignal(
        (context, PostStop) -> {
          context.getSystem().log().info("Worker {} stopped", name);
          return Behaviors.same();
        });
  }
}

final ActorSystem<JobControl.JobControlLanguage> system =
    ActorSystem.create(JobControl.mcpa, "B6700");

system.tell(new JobControl.SpawnJob("a"));
system.tell(new JobControl.SpawnJob("b"));

// sleep here to allow time for the new actors to be started
Thread.sleep(100);

system.tell(new JobControl.GracefulShutdown());

system.getWhenTerminated().toCompletableFuture().get(3, TimeUnit.SECONDS);
```

----------



----------
**英文原文链接**：[Actor lifecycle](https://doc.akka.io/docs/akka/current/typed/actor-lifecycle.html).