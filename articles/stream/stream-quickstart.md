# Streams 快速入门指南

## 依赖

要使用 Akka Streams，请将模块添加到您的项目中：

```scala
val AkkaVersion = "2.6.15"
libraryDependencies += "com.typesafe.akka" %% "akka-stream" % AkkaVersion
```

```
Note:

Akka Streams 的 Java 和 Scala DSL 都捆绑在同一个 JAR 中。为了获得流畅的开发体验，在使用 Eclipse 或 IntelliJ 等 IDE 时，您可以禁用自动`javadsl`导入器在 Scala 中工作时建议导入，反之亦然。请参阅[IDE 提示](https://doc-akka-io.translate.goog/docs/akka/current/additional/ide.html)。
```



## 第一步

流通常从一个源开始，所以这也是我们使用 Akka 流的启动方式。在我们创建一个之前，我们需要先导入完整的流工具：

```scala
import akka.stream._
import akka.stream.scaladsl._
```

如果要在阅读快速入门指南时执行代码示例，还需要以下导入：

```scala
import akka.{ Done, NotUsed }
import akka.actor.ActorSystem
import akka.util.ByteString
import scala.concurrent._
import scala.concurrent.duration._
import java.nio.file.Paths
```



还有一个对象用来启动 Akka `ActorSystem` 和执行您的代码。将`ActorSystem`定义为隐式使其可被自动注入流，而无需在运行它们时手动传递它：

```scala
object Main extends App {
  implicit val system: ActorSystem = ActorSystem("QuickStart")
  // Code here
}
```

现在我们将从一个相当简单的源开始，发射出 1 到 100 的整数：

```scala
val source: Source[Int, NotUsed] = Source(1 to 100)
```

所述`Source`具有两个类型参数：第一个是其所产生的元件类型，第二个“物化值”（`materialized value`）表示源在运行过程中可能产生的一些辅助值（例如一个网络源可能提供关于绑定端口或对等方的地址等信息）的类型。如果不产生任何辅助信息则使用`akka.NotUsed`类型。一个简单的产出整数的源就属于这一类——运行我们的流只会产生 `NotUsed`附加信息.

创建此源意味着我们已经描述了如何发出前 100 个自然数，但此源尚未激活。为了得到这些数字，我们必须运行它：

```scala
source.runForeach(i => println(i))
```

这一行将使用消费者函数来消费源——在本例中，我们将数字打印到控制台——并将这个小的流传递给一个 Actor去执行。我们将“run”作为方法名称的一部分来标识这是一个用于激活运算的函数；还有其他运行 Akka Streams 的函数，它们都遵循这种模式命名。

在一个`scala.App`中运行此源时，您可能会注意到它不会终止，因为`ActorSystem`从未终止。幸运的是`runForeach`返回一个在流结束时解析的：`Future[Done]`

```scala
val done: Future[Done] = source.runForeach(i => println(i))

implicit val ec = system.dispatcher
done.onComplete(_ => system.terminate())
```

Akka Streams 中一个好的方面是`Source`是对您想要运行的内容的描述，就像架构师的蓝图一样，它可以重复使用，合并到更大的设计中。例如我们可以选择对整数源进行转换并最终将其写入文件：

```scala
val factorials = source.scan(BigInt(1))((acc, next) => acc * next)

val result: Future[IOResult] =
  factorials.map(num => ByteString(s"$num\n")).runWith(FileIO.toPath(Paths.get("factorials.txt")))
```

首先，我们使用`scan`运算符在整个流上执行某个计算：以数字 1（BigInt(1) ）为初始，我们将每个传入的数字逐个相乘；scan 操作发出初始值，然后逐个计算结果。这产生了一系列阶乘数，我们将其保存为一个新的源以供以后重用——请注意，这是实际上还没有计算任何东西，这是对我们运行流后想要计算的内容的描述。然后我们将生成的一系列数字的流转换为生成`ByteString`对象的流并逐行记录在文本文件中。这个流在运行时将生成一个文件作为数据的接收者。在 Akka Streams 的术语中，这称为`Sink.IOResult` ，它是在 Akka Streams 中返回的一种IO 类型，用于告诉您消费了多少字节或元素，以及流是正常终止还是异常终止。

### 一个完整的示例

这是另一个可以运行的完整示例：

```scala
import akka.NotUsed
import akka.actor.ActorSystem
import akka.stream.scaladsl._

final case class Author(handle: String)

final case class Hashtag(name: String)

final case class Tweet(author: Author, timestamp: Long, body: String) {
  def hashtags: Set[Hashtag] =
    body
      .split(" ")
      .collect {
        case t if t.startsWith("#") => Hashtag(t.replaceAll("[^#\\w]", ""))
      }
      .toSet
}

val akkaTag = Hashtag("#akka")

val tweets: Source[Tweet, NotUsed] = Source(
  Tweet(Author("rolandkuhn"), System.currentTimeMillis, "#akka rocks!") ::
  Tweet(Author("patriknw"), System.currentTimeMillis, "#akka !") ::
  Tweet(Author("bantonsson"), System.currentTimeMillis, "#akka !") ::
  Tweet(Author("drewhk"), System.currentTimeMillis, "#akka !") ::
  Tweet(Author("ktosopl"), System.currentTimeMillis, "#akka on the rocks!") ::
  Tweet(Author("mmartynas"), System.currentTimeMillis, "wow #akka !") ::
  Tweet(Author("akkateam"), System.currentTimeMillis, "#akka rocks!") ::
  Tweet(Author("bananaman"), System.currentTimeMillis, "#bananas rock!") ::
  Tweet(Author("appleman"), System.currentTimeMillis, "#apples rock!") ::
  Tweet(Author("drama"), System.currentTimeMillis, "we compared #apples to #oranges!") ::
  Nil)

  implicit val system: ActorSystem = ActorSystem("reactive-tweets")

  tweets
    .filterNot(_.hashtags.contains(akkaTag)) // Remove all tweets containing #akka hashtag
    .map(_.hashtags) // Get all sets of hashtags ...
    .reduce(_ ++ _) // ... and reduce them to a single set, removing duplicates across all tweets
    .mapConcat(identity) // Flatten the set of hashtags to a stream of hashtags
    .map(_.name.toUpperCase) // Convert all hashtags to upper case
    .runWith(Sink.foreach(println)) // Attach the Flow to a Sink that will finally print the hashtags
```



## 可重复使用的部件

Akka Streams 的优点之一——也是其他流库不提供的——不仅源可以像蓝图一样重用，所有其他元素也可以如此。我们可以将写入文件的`Sink`和将从`Source`传入的字符串转换成`ByteString`所需的预处理步骤打包在一起做成可重用部件，在 Akka Streams 中，这称为 `Flow`。用于表达这样一个流的“语言”是从左到右的（就像简单的英语一样），因此它（的左边）需要一个“开放”的输入做为其源：

```scala
def lineSink(filename: String): Sink[String, Future[IOResult]] =
  Flow[String].map(s => ByteString(s + "\n")).toMat(FileIO.toPath(Paths.get(filename)))(Keep.right)
```

我们将每个字符串转换为`ByteString`并以此为 `Flow` 处理的起点，然后将其提供给我们之前学习的文件写入类`Sink`。由此导出的蓝图是`Sink[String, Future[IOResult]]` ，这意味着它接受字符串作为其输入，并且将在物化时创建 `Future[IOResult]` 类型的辅助信息（当我们将 Source 或 Flow 进行串联操作时，辅助信息的类型——称为“物化值”——通常由最左边的起点给出；但是因为我们要求由右边的 `FileIO.toPath` Sink 提供该类型，所以我们需要告知以`Keep.right`参数）

现在我们可以新建`Sink`并将它附加到我们的`factorials`源——该源将对数字做一个小小的调整，将之转换为字符串：

```scala
factorials.map(_.toString).runWith(lineSink("factorial2.txt"))
```



## 基于时间的处理

在我们开始查看一个更复杂的示例之前，我们先探索 Akka Streams 可以为流式传输提供什么特性。从`factorials`源开始，我们现在在传输该流的过程中，将它与另外一个流做有拉链式组合（zipping），这个（新的）流将表达为它将发射出数字 0 到 100：`factorials`源发射的第一个数字是 0 的阶乘，第二个是 1 的阶乘，依此类推上。我们通过将二者结合将得到形如`"3! = 6"`的字符串。

```scala
factorials
  .zipWith(Source(0 to 100))((num, idx) => s"$idx! = $num")
  .throttle(1, 1.second)
  .runForeach(println)
```

到目前为止，所有操作都与时间无关，并且可以严格地对元素集合以相同的方式执行。下一行表明我们实际上可以让流以特定的速度流动：我们使用`throttle`运算符将流减慢到每秒 1 个元素。

如果你运行这个程序，你会看到每秒打印一行。不过，有一个不是立即可见的方面值得一提：如果您尝试将流设置为每个流产生 10 亿个数字，那么您会注意到您的 JVM 不会因 OutOfMemoryError  而崩溃，甚至您还会注意到流会运行在后台，并且是异步的（这也是为什么辅助信息是`Future`类型）。使这项工作成功的秘诀是 Akka Streams 隐式地在所有的地方都实现了流量控制，所有操作符都接受背压。这允许 `throttle` 操作符向其所有上游数据源发出信号表明它只能以特定速率接受元素——当传入速率高于每秒一个时，`throttle` 操作符将对上游执行*背压*断言。

简而言之，这对于 Akka Streams 的所有内容都有效——掩盖在几十个源和接收器，以及更多可选择的流转换运算符之下的事实，另请参阅[运算符索引](operators/index.md)。

# 反应式Tweets

流处理的一个典型用例是从一个我们希望对数据流中的数据做提取或聚合。在此示例中，我们将考虑使用消费推文数据流并从中提取有关 Akka 的信息。

我们还将考虑所有非阻塞流解决方案固有的问题：*“如果订阅者太慢而无法跟上实时数据流怎么办？”* 。传统上，解决方案通常是缓冲元素，但这可能——而且通常会——导致最终的缓冲区溢出和此类系统的不稳定。相反，Akka Streams 依赖于内部背压信号，允许控制在这种情况下应该发生的事情。

以下是我们将在整个快速入门示例中使用的数据模型：

```scala
final case class Author(handle: String)

final case class Hashtag(name: String)

final case class Tweet(author: Author, timestamp: Long, body: String) {
  def hashtags: Set[Hashtag] =
    body
      .split(" ")
      .collect {
        case t if t.startsWith("#") => Hashtag(t.replaceAll("[^#\\w]", ""))
      }
      .toSet
}

val akkaTag = Hashtag("#akka")
```

```
提示

如果您想首先了解所用词汇的概述，而不是一头扎进实际示例中，您可以查看文档的[核心概念](stream-flows-and-basics.md)和[定义和运行流](stream-flows-and-basics.md#defining-and-running-streams)章节，然后返回到此快速入门以了解如何将它们拼凑成一个简单的示例应用程序。
```

## 转换与消费简单的流

我们将要查看的示例应用程序是一个简单的 Twitter 摘要流，我们希望从中提取某些信息，例如查找用户的推文找出包含句柄`#akka`的推文。

通过创建一个`ActorSystem`来完成环境准备，它将负责运行我们将要创建的流：

```scala
implicit val system: ActorSystem = ActorSystem("reactive-tweets")
```

假设我们已经有了一个推文流。在 Akka 中，这表示为：`Source[Out, M]`

```scala
val tweets: Source[Tweet, NotUsed]
```

流总是从`Source[Out,M1]`开始流出，然后通过`Flow[In,Out,M2]`元素或更高级的操作符，最终在`Sink[In,M3]`中结束消耗（暂时先忽略类型参数`M1`, `M2` 和 `M3`，目前它们与这些类所以生产或消费的数据类型不相关——它们是“物化类型”，我们将在[下面讨论](stream-quickstart.md#materialized-values-quick)）

这些操作符对于使用过 Scala Collections 库的人来说应该很熟悉，但是它们操作的是流而不是数据集合（这是一个非常重要的区别，因为某些操作只在流中有意义，反之亦然）：

```scala
val authors: Source[Author, NotUsed] =
  tweets.filter(_.hashtags.contains(akkaTag)).map(_.author)
```

最后，为了[物化](stream-flows-and-basics.md#stream-materialization)和运行流计算，我们需要将 Flow 附加到Sink上以使 Flow 开始运行。最简单的方法是在源上调用一个 `runWith(sink)·`。 为方便起见，有一批常用的 Sink 被预先定义成 Sink 伴随对象的方法。现在让我们用它来打印每个作者：

```scala
authors.runWith(Sink.foreach(println))
```

或者使用简写版本（仅针对最流行的Sink定义，例如`Sink.fold`和`Sink.foreach`）：

```scala
authors.runForeach(println)
```

物化和运行流总是需要在隐式范围内存在一个`ActorSystem·`（或通过明确的传入，例如：`.runWith(sink)(system)`）。

完整的代码片段如下所示：

```scala
implicit val system: ActorSystem = ActorSystem("reactive-tweets")

val authors: Source[Author, NotUsed] =
  tweets.filter(_.hashtags.contains(akkaTag)).map(_.author)

authors.runWith(Sink.foreach(println))
```

## 在流中展平序列

在上一节中，我们研究了元素的 1:1 关系，这是最常见的情况，但有时我们可能希望从一个元素映射到多个元素并接收“扁平化”流，类似于`flatMap`在 Scala 集合上的工作. 为了从我们的推文流中获得扁平化的主题标签流，我们可以使用`mapConcat`运算符：

```scala
val hashtags: Source[Hashtag, NotUsed] = tweets.mapConcat(_.hashtags.toList)
```

```
提示

`flatMap`由于它与 for-comprehensions 和 monadic 组合很接近，所以有意识地避免使用了这个名字。它有两个问题：首先，由于存在有死锁的风险，在有界流处理中通常不希望通过串联进行展平（合并是首选策略）；其次，monad 法则不适用于我们的 flatMap 实现（因为活性问题）。

请注意，`mapConcat`只需要提供的函数返回一个iterable（f: Out => immutable.Iterable[T]），而 flatMap 则要求流操作必须从头到尾一直进行。
```

## 广播流

现在假设我们想要保留所有主题标签，以及来自这个实时流的所有作者姓名。例如，我们想将所有截获的作者写入一个文件，并将所有主题标签写入磁盘上的另一个文件。这意味着我们必须将源流分成两个流来对这些不同的文件做写入处理。

可用于形成此类“扇出”（或“扇入”）结构的元素在 Akka Streams 中称为“结点”。我们将在本示例中使用的其中之一称为`Broadcast`，它将元素从其输入端口发射到其所有输出端口。

Akka Streams 有意将线性流结构 (Flow) 与非线性分支结构 (Graph) 分开，以便为这两种情况提供最方便的 API。图（Graph）可以表达任意复杂的流设置，代价是不像集合转换那样熟悉。

用`GraphDSL`构造方法生成图，如下：

```scala
val writeAuthors: Sink[Author, NotUsed] = ???
val writeHashtags: Sink[Hashtag, NotUsed] = ???
val g = RunnableGraph.fromGraph(GraphDSL.create() { implicit b =>
  import GraphDSL.Implicits._

  val bcast = b.add(Broadcast[Tweet](2))
  tweets ~> bcast.in
  bcast.out(0) ~> Flow[Tweet].map(_.author) ~> writeAuthors
  bcast.out(1) ~> Flow[Tweet].mapConcat(_.hashtags.toList) ~> writeHashtags
  ClosedShape
})
g.run()
```

如您所见，在内部`GraphDSL`我们使用隐式图构建器`b`，并使用`~>`“边际运算符”（也读作“connect”或“via”或“to”）构造出可变图。运算符是通过导入隐式提供的`GraphDSL.Implicits._`。

`GraphDSL.create`返回一个`Graph`，在本例中是一个`Graph[ClosedShape, NotUsed]`，这是*一个完全连通图*或“*封闭的*”——表示没有未连接的输入或输出部分。由于它是封闭的，因此可以用`RunnableGraph.fromGraph`将图转换为`RunnableGraph`。可以通过执行`RunnableGraph`的 `run()`将流物化出来。

`Graph`和`RunnableGraph`都是*不可变的，线程安全的，并可安全地共享的*。

图形还可以具有其他几种形状之一，带有一个或多个未连接的端口。具有未连接的端口的图表示它是一个*部分图*。[模块化、组合和层次](stream-composition.md)结构中详细解释了有关在大型结构中组合和嵌套图的概念。还可以将复杂的计算图包装为 Flows、Sink 或 Sources，这将在[从部分图构建 Sources、Sinks 和 Flows](stream-graphs.md)中详细解释。

## 背压如何工作

Akka Streams 的主要优点之一是它们*总是*将背压信息从流接收器（订阅者）传播到它们的源（发布者）。它不是可选功能，而是始终处于启用状态。要了解有关 Akka Streams 和所有其他 Reactive Streams 兼容实现使用的背压协议的更多信息，请阅读[背压解释](stream-flows-and-basics.md#back-pressure-explained)。

像这样的应用程序（不使用 Akka Streams）经常面临的一个典型问题是它们无法足够快地处理传入的数据，无论是暂时的还是设计上的，并且将开始缓冲传入的数据，直到没有更多的空间可以缓冲，导致`OutOfMemoryError`或其他严重的服务响应降级。使用 Akka Streams 可以而且必须显式处理缓冲。例如，如果我们只对“*缓冲 10 个最近的推文*”感兴趣，则可以使用`buffer`元素表示：

```scala
tweets.buffer(10, OverflowStrategy.dropHead).map(slowComputation).runWith(Sink.ignore)
```

该`buffer`显式要求提供一个 `OverflowStrategy`，它定义了缓冲区在满时如何应对接收到的新元素。提供的策略包括删除最早的元素 ( `dropHead`)、删除整个缓冲区、发出错误信号等。确保选择最适合您的用例的策略。

## 值的物化

到目前为止，我们只使用 Flows 处理数据并在某种外部 Sink 中消费它——无论是通过打印还是将它们存储在某个外部系统中。然而，有时我们可能对可以从物化处理管道中获得的某些值感兴趣。例如，我们想知道我们处理了多少条推文。虽然这个问题在推文无限流的情况下并不那么明显（在流设置中回答这个问题的一种方法是创建一个描述为“*到目前为止*，我们已经处理过 N 条推文的计数器流”），但通常可以处理有限流并得出一个不错的结果，例如元素总数。

首先，让我们写一个这样的元素计数器`Sink.fold`，看看类型是怎样的：

```cala
val count: Flow[Tweet, Int, NotUsed] = Flow[Tweet].map(_ => 1)

val sumSink: Sink[Int, Future[Int]] = Sink.fold[Int, Int](0)(_ + _)

val counterGraph: RunnableGraph[Future[Int]] =
  tweets.via(count).toMat(sumSink)(Keep.right)

val sum: Future[Int] = counterGraph.run()

sum.foreach(c => println(s"Total tweets processed: $c"))
```

首先，我们准备一个可重用的`Flow`对象，它将每条传入的推文映射为 value 为`1`的整数。我们将在 `Sink.fold` 中将它们合并，它将合并流中的所有`Int`元素以求和，并将其结果作为`Future[Int]`提供出来。接下来，我们将通过`via`将`tweets`流链接到`count`。最后，我们使用`toMat`将 Flow 连接到先前准备好的 Sink。

还记得`Source[+Out, +Mat]`, `Flow[-In, +Out, +Mat]` 和 `Sink[-In, +Mat]`上那些神秘的`Mat`类型参数吗？它们表示这些处理部件在实现时返回的值的类型。当您将这些链接在一起时，您可以明确地组合它们的物化值。在我们的示例中，我们使用了预定义函数`Keep.right`，它告诉实现只关心当前附加到右侧的运算符的物化类型。`sumSink`的物化类型是 `Future[Int]`，由于使用了 `Keep.right`，结果`RunnableGraph`也有一个`Future[Int]`类型参数。

这一步还*没有*物化处理管道，它只是准备了 Flow 的描述，它现在连接到一个 Sink，因此它可以执行`run()`了，就如同其类型所示：`RunnableGraph[Future[Int]]`。接下来我们调用`run()`，它运行 Flow并实现物化。通过调用在`RunnableGraph[T]`上调用`run()`返回的值的类型是`T`。在我们的例子中，这种类型在完成后将包含我们的流的总长度。如果流失败，这个未来将以`Failure`结束。

A`RunnableGraph`可以被多次重用和物化，因为它只是流的“蓝图”。这意味着，如果我们物化一个流，例如一个在一分钟内消费实时推文流，那么以下两个物化的物化值将不同，如下例所示：

```scala
val sumSink = Sink.fold[Int, Int](0)(_ + _)
val counterRunnableGraph: RunnableGraph[Future[Int]] =
  tweetsInMinuteFromNow.filter(_.hashtags contains akkaTag).map(t => 1).toMat(sumSink)(Keep.right)

// materialize the stream once in the morning
val morningTweetsCount: Future[Int] = counterRunnableGraph.run()

// and once in the evening, reusing the flow
val eveningTweetsCount: Future[Int] = counterRunnableGraph.run()
```



Akka Streams 中的许多元素提供了物化值，它们，它们可用于获取计算结果或控制这些元素，我们将在[Stream Materialization](stream-flows-and-basics.md#stream-materialization)中详细讨论。总结这一节，现在我们知道当我们运行以下这个“线性运算”时在幕后发生了什么，它相当于上面的多行版本：

```scala
val sum: Future[Int] = tweets.map(t => 1).runWith(sumSink)
```

```
提示

runWith()`是一种方便的方法，它会自动忽略任何其他运算符的物化值，除了由它`runWith()`本身附加的那些运算符。在上面的示例中，它用`Keep.right`做为转换物化值的组合器。
```

----

[Akka Streams 背后的设计原则](stream-design.md)

----

https://doc.akka.io/docs/akka/current/stream/stream-quickstart.html

