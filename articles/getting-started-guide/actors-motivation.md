# 为什么现代系统需要新的编程模型？
几十年前，卡尔·休伊特（`Carl Hewitt`）提出了 Actor 模型，将其作为在高性能网络中处理并行任务的一种方法——当时还没有这种环境。如今，硬件和基础设施能力已经赶上并超过了休伊特的设想。因此，当用传统面向对象编程（`OOP`）模型构建苛刻需求的分布式系统遇到无法完全解决的挑战时，可以从 Actor 模型中获益。

如今，Actor 模型不仅被认为是一种高效的解决方案，而且已经在生产中为一些世界上要求极高的应用程序证明了这一点。为了突出 Actor 模型所能解决的问题，本主题将讨论传统编程假设与现代多线程、多 CPU 架构的现实之间的不匹配问题：

- [封装的挑战](#封装的挑战)
- [共享内存在现代计算机架构中的错觉](#共享内存在现代计算机架构中的错觉)
- [调用栈的假象](#调用栈的假象)

## 封装的挑战
OOP 的核心之一是封装。封装意味着不能直接从外部访问对象的内部数据；只能通过调用一组预设的方法来修改它。对象负责将安全的操作暴露出来以保护其封装数据的不变性。

例如，对有序二叉树实现的操作不得违反树的顺序不变性。调用方希望顺序是完整的，并且在查询树中某个数据块时，他们需要对这个约束有深意依赖。

当我们分析 OOP 运行时行为时，可以绘制一个消息序列图，显示方法调用的交互关系。例如：

![object123-1](../../images/actors-motivation/object123-1.png)

不幸的是，上面的图表并不能准确地表示实例在执行期间的生命周期。实际上，一个线程执行所有这些调用，不变量的强制执行是发生在调用该方法的同一个线程上得。更新执行线图，它将如下所示：

![object123-2](../../images/actors-motivation/object123-2.png)

当你试图对多个线程所发生的事情进行建模时，可以清晰地看到该说明的意义。突然间，我们画得很整齐的流程图变得不合适了。我们可以尝试演示出多个线程访问同一实例时得情况：

![object123-3](../../images/actors-motivation/object123-3.png)

如上图所示，在这一部分中，两个线程进入同一个方法。不幸的是，对象的封装模型对该部分中发生得事情不能做出任何保证。两个调用的指令可以以任意方式交错，这样就消除了两个线程之间在没有某种协调的情况下保持不变性的希望。现在，可以想象这个由多个线程的存在造成的问题。

解决这个问题的常用方法是在这些方法周围添加一个锁。虽然这样可以确保在任何给定的时间内最多有一个线程进入该方法，但这是一个非常昂贵的策略：

- 锁严重限制了并发性，它们在现代 CPU 架构上非常昂贵，需要操作系统做重量级的调度以挂起线程并在稍后恢复。
- 当调用线程被阻塞时，它不能执行任何其他有意义的工作。即使在桌面应用程序中，这也是不可接受的，我们希望让面向用户的应用程序部分（`UI`）即使在后台作业长时间运行时也能有所响应。因为在后端，阻塞完全是浪费。有人可能认为可以通过启动新的线程来补偿这一点，但线程也是一个代价高昂的抽象。
- 锁还会带来新的威胁：死锁。

这些现实导致了一种没有赢家的（`no-win`）的局面：

- 如果没有足够的锁，状态就容易受到破坏。
- 使用太多的锁，性能就会受到影响，很容易导致死锁。

另外，锁只能在本地很好的工作。当涉及到跨多台机器协调时，唯一的选择是分布式锁。不幸的是，分布式锁的效率比本地锁低几个数量级，并且通常还会给扩展带来硬限制。分布式锁协议需要跨多台机器在网络上进行多次往返通信，因此其造成机大的影响就是延迟。

在面向对象语言中，我们通常很少考虑线程或线性执行路径。我们通常将系统设想为一个对象实例网络，这些对象实例对方法调用作出反应，修改其内部状态，然后通过方法调用相互通信，从而推动整个应用程序状态的前进：

![object-interact-1](../../images/actors-motivation/object-interact-1.png)

但是，在多线程分布式环境中，实际发生的情况是线程通过以下方法调用“遍历”对象实例网络。因此，线程才是真正推动执行的因素：

![object-interact-2](../../images/actors-motivation/object-interact-2.png)

**总结**：

- 对象只能在单线程访问时保证封装，多线程执行几乎总是导致其内部状态损坏。
- 虽然锁似乎是支持多线程封装的补救方法，但实际上它们效率低下，而且很容易在任何实际规模的应用程序中导致死锁。
- 锁在本地工作，虽然可以使用分布式锁，但其提供的扩展能力有限。

## 共享内存在现代计算机架构中的错觉

在`80 ~ 90`年代的编程模型概念中，写一个变量意味着直接写进了一个内存位置（暂时忽视局部变量有可能只存在于寄存器中的事实）。在现代架构中，如果我们稍微简化一些，CPU 将写入「[缓存线](https://en.wikipedia.org/wiki/CPU_cache)」，而不是直接写入内存。这些缓存在大多数时候都指向 CPU 内核的本地缓存，也就是说，一个内核的写操作对于另一个内核是不可见的。为了使本地的更改对另一个内核可见，从而对另一个线程可见，需要将缓存线同步到另一个内核的缓存线。

在 JVM 上，我们必须通过使用`volatile`标记或原子包装器（`Atomic wrappers`）显式地修饰要在线程间共享的内存变量。否则，我们只能在锁定的部分中访问它们。为什么我们不把所有变量都标记为`volatile`变量呢？因为跨内核传送缓存线（`cache line`）是一项非常昂贵的操作！这样做将隐式地挂起涉及的核心所执行的额外工作，并导致缓存一致性协议（该协议用于在主内存和其他 CPU 之间传输缓存线）上出现瓶颈。结果就是运行速度严重变慢。

即使对于了解这种情况的开发人员来说，找出哪些内存位置应该标记为`volatile`，或者使用哪些原子结构也是一门黑暗的艺术。

**总结**：

- 没有真正的共享内存，CPU 内核像网络上的计算机一样，将数据块（缓存线）显式地传递给彼此。CPU 间通信和网络通信的共性比许多现实都要大。实际上，无论是在 CPU 间还是在通过网络连接的计算机间传递消息都是常态。
- 与通过标记为共享或使用原子数据结构的变量隐藏消息传递方面不同，更加规范和有原则的方法是将状态保持在并发实体的本地，并通过消息在并发实体之间显式地传播数据或事件。

## 调用栈的假象
今天，我们经常把调用栈（`call stacks`）视为理所当然。但是，它们是在一个并发编程不那么重要的时代发明的，因为多 CPU 系统并不常见。调用栈不跨线程，因此没有能力支持异步调用链。

当线程打算将任务委托给“后台”时，就会出现问题。在实践中，这实际上意味着委托给另一个线程。这不能是简单的方法/函数调用，因为调用对于线程来说是严格限制的在本地的。但通常会发生的情况是，“调用者”将一个对象放入一个工作线程（“被调用者”）共享的内存位置，而后者又在某个事件循环中接收它。这允许“调用者”线程继续执行其他任务。

第一个问题是，如何通知“调用者”任务完成了？并且，当一个任务因异常而失败时，会出现一个更严重的问题。异常传播到哪里？它将被传播到工作线程的异常处理程序，完全忽略实际的“调用者”是谁：

![main-worker-thread](../../images/actors-motivation/main-worker-thread.png)

这是一个严重的问题。工作线程（`worker thread`）如何处理这种情况？它可能无法解决问题，因为它通常不知道失败任务的目的。“调用者”线程需要以某种方式得到通知，但是没有调用栈来释放异常。失败通知只能通过一个附加通道（`side-channel`）完成，例如，将错误代码放在“调用者”线程预期结果应该在的地方。如果此通知不到位，则“调用者”永远不会收到失败通知，任务将丢失！**这与网络系统的工作方式惊人地相似，在这种情况下，消息/请求可能会丢失/失败，而没有任何通知**。

当真的发生了错误，一个工作线程遇到了一个 bug，最后陷入了一个不可恢复的情况时，这种糟糕的情况会变得更糟。例如，由 bug 引起的内部异常会冒泡到线程的根目录，并使线程关闭。这立即引发了一个问题，谁应该重新启动由线程承载的服务的正常运行，以及如何将其恢复到已知的良好状态？乍一看，这似乎是可以管理的，但我们突然遇到了一种新的、意想不到的现象：线程当前正在处理的实际任务不再是从共享内存位置中获取任务（通常是队列）。实际上，由于异常会上升到顶部，因此会弹出所有调用栈，任务状态会完全丢失！**即使这是不涉及网络的本地通信，我们也丢失了一条消息（消息丢失是可以预料的）。**

**总结**：

- 为了在当前系统上实现任何有意义的并发性和表现，线程必须以有效的方式相互委托任务且不会阻塞。对于这种任务委托并发风格（对于网络/分布式计算更是如此），基于调用栈的错误处理会出现故障，因此需要引入新的显式错误信号机制。失败应该成为域模型（`domain model`）的一部分。
- 具有工作委托的并发系统需要处理服务故障，并有原则地从故障中恢复。此类服务的客户端需要知道，任务/消息可能会在重新启动时丢失。即使没有发生丢失，响应也可能由于先前排队的任务（长队列）、垃圾收集等而被任意延迟。面对这些情况，并发系统应该以超时的形式处理响应截止时间，就像网络/分布式系统一样。

接下来，让我们看看如何使用 Actor 模型来克服这些挑战。

---------

**名词解析：缓存线**

- `cache line`，数据以固定大小的块在内存和缓存之间传输，称为缓存线或缓存块。当缓存线从内存复制到缓存中时，会创建一个缓存项。缓存项将包括复制的数据以及请求的内存位置（称为标记）。当处理器需要读取或写入主内存中的一个位置时，它首先检查缓存中的相应缓存项。缓存检查可能包含该地址的任何缓存线中请求的内存位置的内容。如果处理器发现内存位置在缓存中，则会发生缓存命中。但是，如果处理器在缓存中找不到内存位置，则会发生缓存未命中。在缓存命中的情况下，处理器会立即读取或写入缓存线中的数据。对于缓存未命中，缓存分配一个新缓存项并从主内存复制数据，然后从缓存的内容完成请求。

----------

[Actor 模型如何满足现代分布式系统的需求 ](actor-intro.md)

----------
**英文原文链接**：[Why modern systems need a new programming model](https://doc.akka.io/docs/akka/current/guide/actors-motivation.html).