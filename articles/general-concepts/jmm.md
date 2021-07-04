# Akka 和 Java 内存模型
使用 LightBend 平台（包括 Scala 和 Akka）的一个主要好处是简化了并发软件的编写过程。本文讨论了 LightBend 平台，特别是 Akka 如何在并发应用程序中处理共享内存。

## Java 内存模型

在 Java 5 之前，Java 内存模型（JMM）的定义存在问题。当多个线程访问共享内存时，可能会得到各种奇怪的结果，例如：

- 线程看不到其他线程写入的值：可见性问题。
- 可能由于未按预期的顺序执行指令从而导致观察到其他线程发生“不可能”的行为：指令重新排序问题。

随着 Java 5 中 JSR 133 的实现，许多问题得到了解决。JMM 是基于“先于发生（`happens-before`）”关系的一组规则，它约束一个内存访问必须发生在另一个内存访问之前的时间。这些规则的两个例子是：

- **监视器锁规则**：在每次后续获取同一个锁之前，必须先释放这个锁。
- **`volatile`变量规则**：`volatile`变量的写入必须发生在同一个`volatile`变量的每次后续读取之前。

虽然 JMM 看起来很复杂，但是规范试图在易用性和编写性能、可扩展并发数据结构的能力之间找到一个平衡点。

## Actors 和 Java 内存模型
通过 Akka 中的 Actor 实现，多个线程可以通过两种方式在共享内存上执行操作：

- 如果消息发送给某个 Actor（例如由另一个 Actor）。在大多数情况下，消息是不可变的，但是如果该消息不是一个正确构造的不可变对象，由于没有“先于发生”规则，则接收者可能会看到部分初始化的数据结构，甚至可能凭空看到不存在的值（`longs/doubles`）。
- 如果 Actor 在处理消息时更改其内部状态，并在稍后处理另一条消息时访问该状态。重要的是要认识到， 在Actor 模型中你不能保证同一线程将对不同的消息执行相同的 Actor。

为了防止 Actor 出现可见性和重新排序问题，Akka 保证以下两条“先于发生”规则：

- **Actor 发送规则**：向 Actor 发送消息的过程发生在同一个 Actor 接收消息之前。
- **Actor 后续处理规则**：一条消息的处理发生在同一个 Actor 处理下一条消息之前。

```
注释：

在外行术语中，这意味着当 Actor 处理下一条消息时，Actor 内部字段的更改是可见的。因此，Actor 中的字段不必是`volatile`或其它等价模式的。
```

这两个规则仅适用于同一个 Actor 实例，如果使用不同的 Actor，则这两个规则无效。

## Futures 和 Java 存储模型
`Future`的“先于发生”完成于任何注册到它的回调被执行之前。

我们建议不要使用非`final`变量（使用 Java 中的`final`和 Scala 中的`val`），如果选择使用非`final`变量，则必须标记`volatile`，以便字段的当前值对回调可见。

如果你打算使用引用，请务必确保引用的实例是线程安全的。我们强烈建议远离使用锁的对象，因为它可能会导致性能问题，在最坏的情况下还会导致死锁。这就是同步的危险。

## Actors 和共享可变状态

由于 Akka 在 JVM 上运行，所以仍然需要遵循一些规则。

最重要的是，您不能将内部可变状态暴露给其他线程：

```java
class MyActor(context: ActorContext[MyActor.Command]) extends AbstractBehavior[MyActor.Command](context) {
  import MyActor._

  var state = ""
  val mySet = mutable.Set[String]()

  def onMessage(cmd: MyActor.Command) = cmd match {
    case Message(text, otherActor) =>
      // 非常糟糕: 共享可变量将允许其它线程修改你的状态，
      // 甚至更糟糕的是，你将陷于怪异的竞态状态。
      otherActor ! mySet

      implicit val ec = context.executionContext

      // 错误的用例
      // 非常糟糕: 共享变量可能让你的程序莫名奇妙崩溃。
      Future { state = "This will race" }

      // 错误的用例
      // 非常糟糕: 共享变量可能让你的程序莫名奇妙崩溃。
      expensiveCalculation().foreach { result =>
        state = s"new state: $result"
      }

      // 正确的实现
      // 将 Future（期物）结果转换成消息并发送给自己。
      val futureResult = expensiveCalculation()
      context.pipeToSelf(futureResult) {
        case Success(result) => UpdateState(result)
        case Failure(ex)     => throw ex
      }

      // 另一个错误的用例
      // 在一个 actor 的询问回调（Future）中修改可变状态
      import akka.actor.typed.scaladsl.AskPattern._
      implicit val timeout = Timeout(5.seconds) // needed for `ask` below
      implicit val scheduler = context.system.scheduler
      val future: Future[String] = otherActor.ask(Query)
      future.foreach { result =>
        state = result
      }

      // 用 context.ask 将结果转换成消息并发送给自己
      context.ask(otherActor, Query) {
        case Success(result) => UpdateState(result)
        case Failure(ex)     => throw ex
      }
      this

    case UpdateState(newState) =>
      // 如果 `newState` 是可变量，为了确保它成为不可变量，我们需要对它进行防御性拷贝。
      state = newState
      this
  }
}
```
- 消息应该是不可变的，这是为了避免共享可变状态陷阱。

----------

[消息传递可靠性](message-delivery-reliability.md)         

----------
**英文原文链接**：[Akka and the Java Memory Model](https://doc.akka.io/docs/akka/current/general/remoting.html).