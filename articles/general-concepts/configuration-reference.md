默认配置

每个 Akka 模块都有一个带有默认值的`reference.conf`文件。

定义您自己的`application.conf`来覆盖，而不是直接修改缺省值。如果您不您的定义是否正确，[Akka Config Checker](https://doc.akka.io/docs/akka-enhancements/current/config-checker.html)是用于查找潜在配置问题的有用工具。

`reference.conf`文件的目的是为，例如Akka库，定义在应用程序未提供定义值时使用的默认值。它也是用来记录配置属性意义的好地方。一个库最好不要在它自己的`reference.conf`中覆盖最初由另一个库的 `reference.conf`定义的属性，因为这样在加载配置时，有效值可能会不确定。

### akka-actor

```scala
####################################
# Akka Actor Reference Config File #
####################################

# 包含了所有缺省配置的这是 reference config 文件
# 请在你自己的 application.conf 中覆盖这些值.

# Akka 版本，请检查 Akka 的运行时的版本。
include "version"

akka {
  # 加载模块的根目录。
  home = ""

  # 引导时注册的日志记录器 (akka.event.Logging$DefaultLogger logs
  # to STDOUT)
  loggers = ["akka.event.Logging$DefaultLogger"]

  # 日志事件过滤器被用在 LoggingAdapter 将日志发布到eventStream 之前。它可被用于
  # 对日志源进行细化。缺省的实现是基于`loglevel`的。
  # FQCN 全域名限定符过滤器。FQCN 类必须实现 akka.event.LoggingFilter接口，并且它的公共构
  # 造函数必须具有(akka.actor.ActorSystem.Settings, akka.event.EventStream) 参数.
  logging-filter = "akka.event.DefaultLoggingFilter"

  # 指定缺省的日志分发器
  loggers-dispatcher = "akka.actor.default-dispatcher"

  # 日志器在 ActorSystem 启动的时候异步注册并新建。并且由于它们是Actor，
  # 因此需要为它们定义超时时限。
  logger-startup-timeout = 5s

  # 当日志器（see “loggers”）被加载，并且在启用 “stdout-loglevel” 之前，所使用的日志级别
  # Options: OFF, ERROR, WARNING, INFO, DEBUG
  loglevel = "INFO"

  # 当ActorSystem 启动并激活非常基础的日志器时所使用的日志级别。
  # 这个日志及录器将日志信息打印到标准输出设备(System.out).
  # Options: OFF, ERROR, WARNING, INFO, DEBUG
  stdout-loglevel = "WARNING"

  # 当 actor 系统启动的时候将完整的日志信息设置为 INFO 级别。
  # 这对于当你不确定应该使用什么配置的时候非常有用。
  log-config-on-start = off

  # 当消息被发送到死信邮箱或被标识为 `DeadLetter`,`Dropped`或`UnhandleMessage`发布到
  # eventStram 的时候的日志级别。
  # 可能的值:
  # on: 记录所有的死信
  # off: 不记录死信
  # n: 正整数, 记录死信的数量
  log-dead-letters = 10

  # 有能力在 actor 系统停止的时候停止对死信进行记录。
  # 只有当 ‘log-dead-letters’ 选项处于有效设置的时候。
  log-dead-letters-during-shutdown = off

  # 在 log-dead-letters 的配置生效的状态下，该配置项将定时重设配置。
  # infinite: 永远暂停日志;
  # 定时重设 (比如：每5分钟).
  log-dead-letters-suspend-duration = 5 minutes

  # Actor 系统启动的时候将会罗列出以下扩展库的全域名限定符(FQCN)。
  # 扩展库是在启动时加载的常规扩展，并且允许自动加载出现在classpath中的第三方库。
  # 这是通过对以下配置附加新的条目来实现的:
  # 'library-extensions += "Extension"' in the library `reference.conf`.
  #
  # 以下配置不应该被最终的用户应用程序的`application.conf`直接修改，而应该是通过配置
  # `extensions` 来实现附加新扩展库。
  #
  library-extensions = ${?akka.library-extensions} ["akka.serialization.SerializationExtension$"]

  # Actor 系统启动的时候将会罗列出以下扩展库的全域名限定符(FQCN)。
  # 它的格式应该是：'extensions = ["foo", "bar"]' etc.
  # 更多的有关扩展库的细节请参考 Akka 文档.
  extensions = []

  # 由此 ActorSystem 创建的切换线程是否应该是守护线程.
  daemonic = off

  # 当系统故障发生的时候，例如 OutOfMemoryError，是否通过 System.exit(-1) 关闭 JVM shutdown。
  jvm-exit-on-fatal-error = on

  # Akk 缺省缺省将会装备 JVM shutdown 钩子，例如在 CoordinatedShutdown 和 Artery 内部
  # 这个属性不会禁止用户通过 `CoordinatedShutdown#addCancellableJvmShutdownHook` 自
  # 定义注册的钩子.
  # 这个属性与下面的 `akka.coordinated-shutdown.run-by-jvm-shutdown-hook` 属性有关联.
  # 这个属性具有禁用那些由应用程序，或者高阶框架例如 Play等，通过 `CoordinatedShutdown`
  # 来安装的 JVM 关闭钩子这一类程序的能力。
  jvm-shutdown-hooks = on

  # 所有模块的版本必须相同，如果它们不同，启动将失败。这项检测可以被禁止并且只在日志中警告，
  # 但是不建议这么做。
  fail-mixed-versions = on

  # 有些模块 (目前只支持远程模块) 可以将事件定制发送到 Java 的飞行记录仪中，如果运行在JDK 11
  # 或更高版本之上. 如果你处于某些原因，可以通过以下设置禁用它。
  java-flight-recorder {
    enabled = true
  }

  actor {

    # Either one of "local", "remote" or "cluster" or the
    # FQCN of the ActorRefProvider to be used; the below is the built-in default,
    # note that "remote" and "cluster" requires the akka-remote and akka-cluster
    # artifacts to be on the classpath.
    provider = "local"

    # The guardian "/user" will use this class to obtain its supervisorStrategy.
    # It needs to be a subclass of akka.actor.SupervisorStrategyConfigurator.
    # In addition to the default there is akka.actor.StoppingSupervisorStrategy.
    guardian-supervisor-strategy = "akka.actor.DefaultSupervisorStrategy"

    # Timeout for Extension creation and a few other potentially blocking
    # initialization tasks.
    creation-timeout = 20s

    # Serializes and deserializes (non-primitive) messages to ensure immutability,
    # this is only intended for testing.
    serialize-messages = off

    # Serializes and deserializes creators (in Props) to ensure that they can be
    # sent over the network, this is only intended for testing. Purely local deployments
    # as marked with deploy.scope == LocalScope are exempt from verification.
    serialize-creators = off

    # If serialize-messages or serialize-creators are enabled classes that starts with
    # a prefix listed here are not verified.
    no-serialization-verification-needed-class-prefix = ["akka."]

    # Timeout for send operations to top-level actors which are in the process
    # of being started. This is only relevant if using a bounded mailbox or the
    # CallingThreadDispatcher for a top-level actor.
    unstarted-push-timeout = 10s

    # TypedActor deprecated since 2.6.0.
    typed {
      # Default timeout for the deprecated TypedActor (not the new actor APIs in 2.6)
      # methods with non-void return type.
      timeout = 5s
    }

    # Mapping between ´deployment.router' short names to fully qualified class names
    router.type-mapping {
      from-code = "akka.routing.NoRouter"
      round-robin-pool = "akka.routing.RoundRobinPool"
      round-robin-group = "akka.routing.RoundRobinGroup"
      random-pool = "akka.routing.RandomPool"
      random-group = "akka.routing.RandomGroup"
      balancing-pool = "akka.routing.BalancingPool"
      smallest-mailbox-pool = "akka.routing.SmallestMailboxPool"
      broadcast-pool = "akka.routing.BroadcastPool"
      broadcast-group = "akka.routing.BroadcastGroup"
      scatter-gather-pool = "akka.routing.ScatterGatherFirstCompletedPool"
      scatter-gather-group = "akka.routing.ScatterGatherFirstCompletedGroup"
      tail-chopping-pool = "akka.routing.TailChoppingPool"
      tail-chopping-group = "akka.routing.TailChoppingGroup"
      consistent-hashing-pool = "akka.routing.ConsistentHashingPool"
      consistent-hashing-group = "akka.routing.ConsistentHashingGroup"
    }

    deployment {

      # deployment id pattern - on the format: /parent/child etc.
      default {

        # The id of the dispatcher to use for this actor.
        # If undefined or empty the dispatcher specified in code
        # (Props.withDispatcher) is used, or default-dispatcher if not
        # specified at all.
        dispatcher = ""

        # The id of the mailbox to use for this actor.
        # If undefined or empty the default mailbox of the configured dispatcher
        # is used or if there is no mailbox configuration the mailbox specified
        # in code (Props.withMailbox) is used.
        # If there is a mailbox defined in the configured dispatcher then that
        # overrides this setting.
        mailbox = ""

        # routing (load-balance) scheme to use
        # - available: "from-code", "round-robin", "random", "smallest-mailbox",
        #              "scatter-gather", "broadcast"
        # - or:        Fully qualified class name of the router class.
        #              The class must extend akka.routing.CustomRouterConfig and
        #              have a public constructor with com.typesafe.config.Config
        #              and optional akka.actor.DynamicAccess parameter.
        # - default is "from-code";
        # Whether or not an actor is transformed to a Router is decided in code
        # only (Props.withRouter). The type of router can be overridden in the
        # configuration; specifying "from-code" means that the values specified
        # in the code shall be used.
        # In case of routing, the actors to be routed to can be specified
        # in several ways:
        # - nr-of-instances: will create that many children
        # - routees.paths: will route messages to these paths using ActorSelection,
        #   i.e. will not create children
        # - resizer: dynamically resizable number of routees as specified in
        #   resizer below
        router = "from-code"

        # number of children to create in case of a router;
        # this setting is ignored if routees.paths is given
        nr-of-instances = 1

        # within is the timeout used for routers containing future calls
        within = 5 seconds

        # number of virtual nodes per node for consistent-hashing router
        virtual-nodes-factor = 10

        tail-chopping-router {
          # interval is duration between sending message to next routee
          interval = 10 milliseconds
        }

        routees {
          # Alternatively to giving nr-of-instances you can specify the full
          # paths of those actors which should be routed to. This setting takes
          # precedence over nr-of-instances
          paths = []
        }

        # To use a dedicated dispatcher for the routees of the pool you can
        # define the dispatcher configuration inline with the property name
        # 'pool-dispatcher' in the deployment section of the router.
        # For example:
        # pool-dispatcher {
        #   fork-join-executor.parallelism-min = 5
        #   fork-join-executor.parallelism-max = 5
        # }

        # Routers with dynamically resizable number of routees; this feature is
        # enabled by including (parts of) this section in the deployment
        resizer {

          enabled = off

          # The fewest number of routees the router should ever have.
          lower-bound = 1

          # The most number of routees the router should ever have.
          # Must be greater than or equal to lower-bound.
          upper-bound = 10

          # Threshold used to evaluate if a routee is considered to be busy
          # (under pressure). Implementation depends on this value (default is 1).
          # 0:   number of routees currently processing a message.
          # 1:   number of routees currently processing a message has
          #      some messages in mailbox.
          # > 1: number of routees with at least the configured pressure-threshold
          #      messages in their mailbox. Note that estimating mailbox size of
          #      default UnboundedMailbox is O(N) operation.
          pressure-threshold = 1

          # Percentage to increase capacity whenever all routees are busy.
          # For example, 0.2 would increase 20% (rounded up), i.e. if current
          # capacity is 6 it will request an increase of 2 more routees.
          rampup-rate = 0.2

          # Minimum fraction of busy routees before backing off.
          # For example, if this is 0.3, then we'll remove some routees only when
          # less than 30% of routees are busy, i.e. if current capacity is 10 and
          # 3 are busy then the capacity is unchanged, but if 2 or less are busy
          # the capacity is decreased.
          # Use 0.0 or negative to avoid removal of routees.
          backoff-threshold = 0.3

          # Fraction of routees to be removed when the resizer reaches the
          # backoffThreshold.
          # For example, 0.1 would decrease 10% (rounded up), i.e. if current
          # capacity is 9 it will request an decrease of 1 routee.
          backoff-rate = 0.1

          # Number of messages between resize operation.
          # Use 1 to resize before each message.
          messages-per-resize = 10
        }

        # Routers with dynamically resizable number of routees based on
        # performance metrics.
        # This feature is enabled by including (parts of) this section in
        # the deployment, cannot be enabled together with default resizer.
        optimal-size-exploring-resizer {

          enabled = off

          # The fewest number of routees the router should ever have.
          lower-bound = 1

          # The most number of routees the router should ever have.
          # Must be greater than or equal to lower-bound.
          upper-bound = 10

          # probability of doing a ramping down when all routees are busy
          # during exploration.
          chance-of-ramping-down-when-full = 0.2

          # Interval between each resize attempt
          action-interval = 5s

          # If the routees have not been fully utilized (i.e. all routees busy)
          # for such length, the resizer will downsize the pool.
          downsize-after-underutilized-for = 72h

          # Duration exploration, the ratio between the largest step size and
          # current pool size. E.g. if the current pool size is 50, and the
          # explore-step-size is 0.1, the maximum pool size change during
          # exploration will be +- 5
          explore-step-size = 0.1

          # Probability of doing an exploration v.s. optimization.
          chance-of-exploration = 0.4

          # When downsizing after a long streak of underutilization, the resizer
          # will downsize the pool to the highest utiliziation multiplied by a
          # a downsize ratio. This downsize ratio determines the new pools size
          # in comparison to the highest utilization.
          # E.g. if the highest utilization is 10, and the down size ratio
          # is 0.8, the pool will be downsized to 8
          downsize-ratio = 0.8

          # When optimizing, the resizer only considers the sizes adjacent to the
          # current size. This number indicates how many adjacent sizes to consider.
          optimization-range = 16

          # The weight of the latest metric over old metrics when collecting
          # performance metrics.
          # E.g. if the last processing speed is 10 millis per message at pool
          # size 5, and if the new processing speed collected is 6 millis per
          # message at pool size 5. Given a weight of 0.3, the metrics
          # representing pool size 5 will be 6 * 0.3 + 10 * 0.7, i.e. 8.8 millis
          # Obviously, this number should be between 0 and 1.
          weight-of-latest-metric = 0.5
        }
      }

      "/IO-DNS/inet-address" {
        mailbox = "unbounded"
        router = "consistent-hashing-pool"
        nr-of-instances = 4
      }

      "/IO-DNS/inet-address/*" {
        dispatcher = "akka.actor.default-blocking-io-dispatcher"
      }

      "/IO-DNS/async-dns" {
        mailbox = "unbounded"
        router = "round-robin-pool"
        nr-of-instances = 1
      }
    }

    default-dispatcher {
      # Must be one of the following
      # Dispatcher, PinnedDispatcher, or a FQCN to a class inheriting
      # MessageDispatcherConfigurator with a public constructor with
      # both com.typesafe.config.Config parameter and
      # akka.dispatch.DispatcherPrerequisites parameters.
      # PinnedDispatcher must be used together with executor=thread-pool-executor.
      type = "Dispatcher"

      # Which kind of ExecutorService to use for this dispatcher
      # Valid options:
      #  - "default-executor" requires a "default-executor" section
      #  - "fork-join-executor" requires a "fork-join-executor" section
      #  - "thread-pool-executor" requires a "thread-pool-executor" section
      #  - "affinity-pool-executor" requires an "affinity-pool-executor" section
      #  - A FQCN of a class extending ExecutorServiceConfigurator
      executor = "default-executor"

      # This will be used if you have set "executor = "default-executor"".
      # If an ActorSystem is created with a given ExecutionContext, this
      # ExecutionContext will be used as the default executor for all
      # dispatchers in the ActorSystem configured with
      # executor = "default-executor". Note that "default-executor"
      # is the default value for executor, and therefore used if not
      # specified otherwise. If no ExecutionContext is given,
      # the executor configured in "fallback" will be used.
      default-executor {
        fallback = "fork-join-executor"
      }

      # This will be used if you have set "executor = "affinity-pool-executor""
      # Underlying thread pool implementation is akka.dispatch.affinity.AffinityPool.
      # This executor is classified as "ApiMayChange".
      affinity-pool-executor {
        # Min number of threads to cap factor-based parallelism number to
        parallelism-min = 4

        # The parallelism factor is used to determine thread pool size using the
        # following formula: ceil(available processors * factor). Resulting size
        # is then bounded by the parallelism-min and parallelism-max values.
        parallelism-factor = 0.8

        # Max number of threads to cap factor-based parallelism number to.
        parallelism-max = 64

        # Each worker in the pool uses a separate bounded MPSC queue. This value
        # indicates the upper bound of the queue. Whenever an attempt to enqueue
        # a task is made and the queue does not have capacity to accommodate
        # the task, the rejection handler created by the rejection handler specified
        # in "rejection-handler" is invoked.
        task-queue-size = 512

        # FQCN of the Rejection handler used in the pool.
        # Must have an empty public constructor and must
        # implement akka.actor.affinity.RejectionHandlerFactory.
        rejection-handler = "akka.dispatch.affinity.ThrowOnOverflowRejectionHandler"

        # Level of CPU time used, on a scale between 1 and 10, during backoff/idle.
        # The tradeoff is that to have low latency more CPU time must be used to be
        # able to react quickly on incoming messages or send as fast as possible after
        # backoff backpressure.
        # Level 1 strongly prefer low CPU consumption over low latency.
        # Level 10 strongly prefer low latency over low CPU consumption.
        idle-cpu-level = 5

        # FQCN of the akka.dispatch.affinity.QueueSelectorFactory.
        # The Class of the FQCN must have a public constructor with a
        # (com.typesafe.config.Config) parameter.
        # A QueueSelectorFactory create instances of akka.dispatch.affinity.QueueSelector,
        # that is responsible for determining which task queue a Runnable should be enqueued in.
        queue-selector = "akka.dispatch.affinity.FairDistributionHashCache"

        # When using the "akka.dispatch.affinity.FairDistributionHashCache" queue selector
        # internally the AffinityPool uses two methods to determine which task
        # queue to allocate a Runnable to:
        # - map based - maintains a round robin counter and a map of Runnable
        # hashcodes to queues that they have been associated with. This ensures
        # maximum fairness in terms of work distribution, meaning that each worker
        # will get approximately equal amount of mailboxes to execute. This is suitable
        # in cases where we have a small number of actors that will be scheduled on
        # the pool and we want to ensure the maximum possible utilization of the
        # available threads.
        # - hash based - the task - queue in which the runnable should go is determined
        # by using an uniformly distributed int to int hash function which uses the
        # hash code of the Runnable as an input. This is preferred in situations where we
        # have enough number of distinct actors to ensure statistically uniform
        # distribution of work across threads or we are ready to sacrifice the
        # former for the added benefit of avoiding map look-ups.
        fair-work-distribution {
          # The value serves as a threshold which determines the point at which the
          # pool switches from the first to the second work distribution schemes.
          # For example, if the value is set to 128, the pool can observe up to
          # 128 unique actors and schedule their mailboxes using the map based
          # approach. Once this number is reached the pool switches to hash based
          # task distribution mode. If the value is set to 0, the map based
          # work distribution approach is disabled and only the hash based is
          # used irrespective of the number of unique actors. Valid range is
          # 0 to 2048 (inclusive)
          threshold = 128
        }
      }

      # This will be used if you have set "executor = "fork-join-executor""
      # Underlying thread pool implementation is java.util.concurrent.ForkJoinPool
      fork-join-executor {
        # Min number of threads to cap factor-based parallelism number to
        parallelism-min = 8

        # The parallelism factor is used to determine thread pool size using the
        # following formula: ceil(available processors * factor). Resulting size
        # is then bounded by the parallelism-min and parallelism-max values.
        parallelism-factor = 1.0

        # Max number of threads to cap factor-based parallelism number to
        parallelism-max = 64

        # Setting to "FIFO" to use queue like peeking mode which "poll" or "LIFO" to use stack
        # like peeking mode which "pop".
        task-peeking-mode = "FIFO"
      }

      # This will be used if you have set "executor = "thread-pool-executor""
      # Underlying thread pool implementation is java.util.concurrent.ThreadPoolExecutor
      thread-pool-executor {
        # Keep alive time for threads
        keep-alive-time = 60s

        # Define a fixed thread pool size with this property. The corePoolSize
        # and the maximumPoolSize of the ThreadPoolExecutor will be set to this
        # value, if it is defined. Then the other pool-size properties will not
        # be used.
        #
        # Valid values are: `off` or a positive integer.
        fixed-pool-size = off

        # Min number of threads to cap factor-based corePoolSize number to
        core-pool-size-min = 8

        # The core-pool-size-factor is used to determine corePoolSize of the
        # ThreadPoolExecutor using the following formula:
        # ceil(available processors * factor).
        # Resulting size is then bounded by the core-pool-size-min and
        # core-pool-size-max values.
        core-pool-size-factor = 3.0

        # Max number of threads to cap factor-based corePoolSize number to
        core-pool-size-max = 64

        # Minimum number of threads to cap factor-based maximumPoolSize number to
        max-pool-size-min = 8

        # The max-pool-size-factor is used to determine maximumPoolSize of the
        # ThreadPoolExecutor using the following formula:
        # ceil(available processors * factor)
        # The maximumPoolSize will not be less than corePoolSize.
        # It is only used if using a bounded task queue.
        max-pool-size-factor  = 3.0

        # Max number of threads to cap factor-based maximumPoolSize number to
        max-pool-size-max = 64

        # Specifies the bounded capacity of the task queue (< 1 == unbounded)
        task-queue-size = -1

        # Specifies which type of task queue will be used, can be "array" or
        # "linked" (default)
        task-queue-type = "linked"

        # Allow core threads to time out
        allow-core-timeout = on
      }

      # How long time the dispatcher will wait for new actors until it shuts down
      shutdown-timeout = 1s

      # Throughput defines the number of messages that are processed in a batch
      # before the thread is returned to the pool. Set to 1 for as fair as possible.
      throughput = 5

      # Throughput deadline for Dispatcher, set to 0 or negative for no deadline
      throughput-deadline-time = 0ms

      # For BalancingDispatcher: If the balancing dispatcher should attempt to
      # schedule idle actors using the same dispatcher when a message comes in,
      # and the dispatchers ExecutorService is not fully busy already.
      attempt-teamwork = on

      # If this dispatcher requires a specific type of mailbox, specify the
      # fully-qualified class name here; the actually created mailbox will
      # be a subtype of this type. The empty string signifies no requirement.
      mailbox-requirement = ""
    }

    # Default separate internal dispatcher to run Akka internal tasks and actors on
    # protecting them against starvation because of accidental blocking in user actors (which run on the
    # default dispatcher)
    internal-dispatcher {
      type = "Dispatcher"
      executor = "fork-join-executor"
      throughput = 5
      fork-join-executor {
        parallelism-min = 4
        parallelism-factor = 1.0
        parallelism-max = 64
      }
    }

    default-blocking-io-dispatcher {
      type = "Dispatcher"
      executor = "thread-pool-executor"
      throughput = 1

      thread-pool-executor {
        fixed-pool-size = 16
      }
    }

    default-mailbox {
      # FQCN of the MailboxType. The Class of the FQCN must have a public
      # constructor with
      # (akka.actor.ActorSystem.Settings, com.typesafe.config.Config) parameters.
      mailbox-type = "akka.dispatch.UnboundedMailbox"

      # If the mailbox is bounded then it uses this setting to determine its
      # capacity. The provided value must be positive.
      # NOTICE:
      # Up to version 2.1 the mailbox type was determined based on this setting;
      # this is no longer the case, the type must explicitly be a bounded mailbox.
      mailbox-capacity = 1000

      # If the mailbox is bounded then this is the timeout for enqueueing
      # in case the mailbox is full. Negative values signify infinite
      # timeout, which should be avoided as it bears the risk of dead-lock.
      mailbox-push-timeout-time = 10s

      # For Actor with Stash: The default capacity of the stash.
      # If negative (or zero) then an unbounded stash is used (default)
      # If positive then a bounded stash is used and the capacity is set using
      # the property
      stash-capacity = -1
    }

    mailbox {
      # Mapping between message queue semantics and mailbox configurations.
      # Used by akka.dispatch.RequiresMessageQueue[T] to enforce different
      # mailbox types on actors.
      # If your Actor implements RequiresMessageQueue[T], then when you create
      # an instance of that actor its mailbox type will be decided by looking
      # up a mailbox configuration via T in this mapping
      requirements {
        "akka.dispatch.UnboundedMessageQueueSemantics" =
          akka.actor.mailbox.unbounded-queue-based
        "akka.dispatch.BoundedMessageQueueSemantics" =
          akka.actor.mailbox.bounded-queue-based
        "akka.dispatch.DequeBasedMessageQueueSemantics" =
          akka.actor.mailbox.unbounded-deque-based
        "akka.dispatch.UnboundedDequeBasedMessageQueueSemantics" =
          akka.actor.mailbox.unbounded-deque-based
        "akka.dispatch.BoundedDequeBasedMessageQueueSemantics" =
          akka.actor.mailbox.bounded-deque-based
        "akka.dispatch.MultipleConsumerSemantics" =
          akka.actor.mailbox.unbounded-queue-based
        "akka.dispatch.ControlAwareMessageQueueSemantics" =
          akka.actor.mailbox.unbounded-control-aware-queue-based
        "akka.dispatch.UnboundedControlAwareMessageQueueSemantics" =
          akka.actor.mailbox.unbounded-control-aware-queue-based
        "akka.dispatch.BoundedControlAwareMessageQueueSemantics" =
          akka.actor.mailbox.bounded-control-aware-queue-based
        "akka.event.LoggerMessageQueueSemantics" =
          akka.actor.mailbox.logger-queue
      }

      unbounded-queue-based {
        # FQCN of the MailboxType, The Class of the FQCN must have a public
        # constructor with (akka.actor.ActorSystem.Settings,
        # com.typesafe.config.Config) parameters.
        mailbox-type = "akka.dispatch.UnboundedMailbox"
      }

      bounded-queue-based {
        # FQCN of the MailboxType, The Class of the FQCN must have a public
        # constructor with (akka.actor.ActorSystem.Settings,
        # com.typesafe.config.Config) parameters.
        mailbox-type = "akka.dispatch.BoundedMailbox"
      }

      unbounded-deque-based {
        # FQCN of the MailboxType, The Class of the FQCN must have a public
        # constructor with (akka.actor.ActorSystem.Settings,
        # com.typesafe.config.Config) parameters.
        mailbox-type = "akka.dispatch.UnboundedDequeBasedMailbox"
      }

      bounded-deque-based {
        # FQCN of the MailboxType, The Class of the FQCN must have a public
        # constructor with (akka.actor.ActorSystem.Settings,
        # com.typesafe.config.Config) parameters.
        mailbox-type = "akka.dispatch.BoundedDequeBasedMailbox"
      }

      unbounded-control-aware-queue-based {
        # FQCN of the MailboxType, The Class of the FQCN must have a public
        # constructor with (akka.actor.ActorSystem.Settings,
        # com.typesafe.config.Config) parameters.
        mailbox-type = "akka.dispatch.UnboundedControlAwareMailbox"
      }

      bounded-control-aware-queue-based {
        # FQCN of the MailboxType, The Class of the FQCN must have a public
        # constructor with (akka.actor.ActorSystem.Settings,
        # com.typesafe.config.Config) parameters.
        mailbox-type = "akka.dispatch.BoundedControlAwareMailbox"
      }

      # The LoggerMailbox will drain all messages in the mailbox
      # when the system is shutdown and deliver them to the StandardOutLogger.
      # Do not change this unless you know what you are doing.
      logger-queue {
        mailbox-type = "akka.event.LoggerMailboxType"
      }
    }

    debug {
      # enable function of Actor.loggable(), which is to log any received message
      # at DEBUG level, see the “Testing Actor Systems” section of the Akka
      # Documentation at http://akka.io/docs
      receive = off

      # enable DEBUG logging of all AutoReceiveMessages (Kill, PoisonPill etc.)
      autoreceive = off

      # enable DEBUG logging of actor lifecycle changes
      lifecycle = off

      # enable DEBUG logging of all LoggingFSMs for events, transitions and timers
      fsm = off

      # enable DEBUG logging of subscription changes on the eventStream
      event-stream = off

      # enable DEBUG logging of unhandled messages
      unhandled = off

      # enable WARN logging of misconfigured routers
      router-misconfiguration = off
    }

    # SECURITY BEST-PRACTICE is to disable java serialization for its multiple
    # known attack surfaces.
    #
    # This setting is a short-cut to
    # - using DisabledJavaSerializer instead of JavaSerializer
    #
    # Completely disable the use of `akka.serialization.JavaSerialization` by the
    # Akka Serialization extension, instead DisabledJavaSerializer will
    # be inserted which will fail explicitly if attempts to use java serialization are made.
    #
    # The log messages emitted by such serializer SHOULD be treated as potential
    # attacks which the serializer prevented, as they MAY indicate an external operator
    # attempting to send malicious messages intending to use java serialization as attack vector.
    # The attempts are logged with the SECURITY marker.
    #
    # Please note that this option does not stop you from manually invoking java serialization
    #
    allow-java-serialization = off

    # Log warnings when the Java serialization is used to serialize messages.
    # Java serialization is not very performant and should not be used in production
    # environments unless you don't care about performance and security. In that case
    # you can turn this off.
    warn-about-java-serializer-usage = on

    # To be used with the above warn-about-java-serializer-usage
    # When warn-about-java-serializer-usage = on, and this warn-on-no-serialization-verification = off,
    # warnings are suppressed for classes extending NoSerializationVerificationNeeded
    # to reduce noise.
    warn-on-no-serialization-verification = on

    # Entries for pluggable serializers and their bindings.
    serializers {
      java = "akka.serialization.JavaSerializer"
      bytes = "akka.serialization.ByteArraySerializer"
      primitive-long = "akka.serialization.LongSerializer"
      primitive-int = "akka.serialization.IntSerializer"
      primitive-string = "akka.serialization.StringSerializer"
      primitive-bytestring = "akka.serialization.ByteStringSerializer"
      primitive-boolean = "akka.serialization.BooleanSerializer"
    }

    # Class to Serializer binding. You only need to specify the name of an
    # interface or abstract base class of the messages. In case of ambiguity it
    # is using the most specific configured class, or giving a warning and
    # choosing the “first” one.
    #
    # To disable one of the default serializers, assign its class to "none", like
    # "java.io.Serializable" = none
    serialization-bindings {
      "[B" = bytes
      "java.io.Serializable" = java

      "java.lang.String" = primitive-string
      "akka.util.ByteString$ByteString1C" = primitive-bytestring
      "akka.util.ByteString$ByteString1" = primitive-bytestring
      "akka.util.ByteString$ByteStrings" = primitive-bytestring
      "java.lang.Long" = primitive-long
      "scala.Long" = primitive-long
      "java.lang.Integer" = primitive-int
      "scala.Int" = primitive-int
      "java.lang.Boolean" = primitive-boolean
      "scala.Boolean" = primitive-boolean
    }

    # Configuration namespace of serialization identifiers.
    # Each serializer implementation must have an entry in the following format:
    # `akka.actor.serialization-identifiers."FQCN" = ID`
    # where `FQCN` is fully qualified class name of the serializer implementation
    # and `ID` is globally unique serializer identifier number.
    # Identifier values from 0 to 40 are reserved for Akka internal usage.
    serialization-identifiers {
      "akka.serialization.JavaSerializer" = 1
      "akka.serialization.ByteArraySerializer" = 4

      primitive-long = 18
      primitive-int = 19
      primitive-string = 20
      primitive-bytestring = 21
      primitive-boolean = 35
    }

  }

  serialization.protobuf {
    # deprecated, use `allowed-classes` instead
    whitelist-class = [
      "com.google.protobuf.GeneratedMessage",
      "com.google.protobuf.GeneratedMessageV3",
      "scalapb.GeneratedMessageCompanion",
      "akka.protobuf.GeneratedMessage",
      "akka.protobufv3.internal.GeneratedMessageV3"
    ]

    # Additional classes that are allowed even if they are not defined in `serialization-bindings`.
    # It can be exact class name or name of super class or interfaces (one level).
    # This is useful when a class is not used for serialization any more and therefore removed
    # from `serialization-bindings`, but should still be possible to deserialize.
    allowed-classes = ${akka.serialization.protobuf.whitelist-class}

  }

  # Used to set the behavior of the scheduler.
  # Changing the default values may change the system behavior drastically so make
  # sure you know what you're doing! See the Scheduler section of the Akka
  # Documentation for more details.
  scheduler {
    # The LightArrayRevolverScheduler is used as the default scheduler in the
    # system. It does not execute the scheduled tasks on exact time, but on every
    # tick, it will run everything that is (over)due. You can increase or decrease
    # the accuracy of the execution timing by specifying smaller or larger tick
    # duration. If you are scheduling a lot of tasks you should consider increasing
    # the ticks per wheel.
    # Note that it might take up to 1 tick to stop the Timer, so setting the
    # tick-duration to a high value will make shutting down the actor system
    # take longer.
    tick-duration = 10ms

    # The timer uses a circular wheel of buckets to store the timer tasks.
    # This should be set such that the majority of scheduled timeouts (for high
    # scheduling frequency) will be shorter than one rotation of the wheel
    # (ticks-per-wheel * ticks-duration)
    # THIS MUST BE A POWER OF TWO!
    ticks-per-wheel = 512

    # This setting selects the timer implementation which shall be loaded at
    # system start-up.
    # The class given here must implement the akka.actor.Scheduler interface
    # and offer a public constructor which takes three arguments:
    #  1) com.typesafe.config.Config
    #  2) akka.event.LoggingAdapter
    #  3) java.util.concurrent.ThreadFactory
    implementation = akka.actor.LightArrayRevolverScheduler

    # When shutting down the scheduler, there will typically be a thread which
    # needs to be stopped, and this timeout determines how long to wait for
    # that to happen. In case of timeout the shutdown of the actor system will
    # proceed without running possibly still enqueued tasks.
    shutdown-timeout = 5s
  }

  io {

    # By default the select loops run on dedicated threads, hence using a
    # PinnedDispatcher
    pinned-dispatcher {
      type = "PinnedDispatcher"
      executor = "thread-pool-executor"
      thread-pool-executor.allow-core-timeout = off
    }

    tcp {

      # The number of selectors to stripe the served channels over; each of
      # these will use one select loop on the selector-dispatcher.
      nr-of-selectors = 1

      # Maximum number of open channels supported by this TCP module; there is
      # no intrinsic general limit, this setting is meant to enable DoS
      # protection by limiting the number of concurrently connected clients.
      # Also note that this is a "soft" limit; in certain cases the implementation
      # will accept a few connections more or a few less than the number configured
      # here. Must be an integer > 0 or "unlimited".
      max-channels = 256000

      # When trying to assign a new connection to a selector and the chosen
      # selector is at full capacity, retry selector choosing and assignment
      # this many times before giving up
      selector-association-retries = 10

      # The maximum number of connection that are accepted in one go,
      # higher numbers decrease latency, lower numbers increase fairness on
      # the worker-dispatcher
      batch-accept-limit = 10

      # The number of bytes per direct buffer in the pool used to read or write
      # network data from the kernel.
      direct-buffer-size = 128 KiB

      # The maximal number of direct buffers kept in the direct buffer pool for
      # reuse.
      direct-buffer-pool-limit = 1000

      # The duration a connection actor waits for a `Register` message from
      # its commander before aborting the connection.
      register-timeout = 5s

      # The maximum number of bytes delivered by a `Received` message. Before
      # more data is read from the network the connection actor will try to
      # do other work.
      # The purpose of this setting is to impose a smaller limit than the
      # configured receive buffer size. When using value 'unlimited' it will
      # try to read all from the receive buffer.
      max-received-message-size = unlimited

      # Enable fine grained logging of what goes on inside the implementation.
      # Be aware that this may log more than once per message sent to the actors
      # of the tcp implementation.
      trace-logging = off

      # Fully qualified config path which holds the dispatcher configuration
      # to be used for running the select() calls in the selectors
      selector-dispatcher = "akka.io.pinned-dispatcher"

      # Fully qualified config path which holds the dispatcher configuration
      # for the read/write worker actors
      worker-dispatcher = "akka.actor.internal-dispatcher"

      # Fully qualified config path which holds the dispatcher configuration
      # for the selector management actors
      management-dispatcher = "akka.actor.internal-dispatcher"

      # Fully qualified config path which holds the dispatcher configuration
      # on which file IO tasks are scheduled
      file-io-dispatcher = "akka.actor.default-blocking-io-dispatcher"

      # The maximum number of bytes (or "unlimited") to transfer in one batch
      # when using `WriteFile` command which uses `FileChannel.transferTo` to
      # pipe files to a TCP socket. On some OS like Linux `FileChannel.transferTo`
      # may block for a long time when network IO is faster than file IO.
      # Decreasing the value may improve fairness while increasing may improve
      # throughput.
      file-io-transferTo-limit = 512 KiB

      # The number of times to retry the `finishConnect` call after being notified about
      # OP_CONNECT. Retries are needed if the OP_CONNECT notification doesn't imply that
      # `finishConnect` will succeed, which is the case on Android.
      finish-connect-retries = 5

      # On Windows connection aborts are not reliably detected unless an OP_READ is
      # registered on the selector _after_ the connection has been reset. This
      # workaround enables an OP_CONNECT which forces the abort to be visible on Windows.
      # Enabling this setting on other platforms than Windows will cause various failures
      # and undefined behavior.
      # Possible values of this key are on, off and auto where auto will enable the
      # workaround if Windows is detected automatically.
      windows-connection-abort-workaround-enabled = off
    }

    udp {

      # The number of selectors to stripe the served channels over; each of
      # these will use one select loop on the selector-dispatcher.
      nr-of-selectors = 1

      # Maximum number of open channels supported by this UDP module Generally
      # UDP does not require a large number of channels, therefore it is
      # recommended to keep this setting low.
      max-channels = 4096

      # The select loop can be used in two modes:
      # - setting "infinite" will select without a timeout, hogging a thread
      # - setting a positive timeout will do a bounded select call,
      #   enabling sharing of a single thread between multiple selectors
      #   (in this case you will have to use a different configuration for the
      #   selector-dispatcher, e.g. using "type=Dispatcher" with size 1)
      # - setting it to zero means polling, i.e. calling selectNow()
      select-timeout = infinite

      # When trying to assign a new connection to a selector and the chosen
      # selector is at full capacity, retry selector choosing and assignment
      # this many times before giving up
      selector-association-retries = 10

      # The maximum number of datagrams that are read in one go,
      # higher numbers decrease latency, lower numbers increase fairness on
      # the worker-dispatcher
      receive-throughput = 3

      # The number of bytes per direct buffer in the pool used to read or write
      # network data from the kernel.
      direct-buffer-size = 128 KiB

      # The maximal number of direct buffers kept in the direct buffer pool for
      # reuse.
      direct-buffer-pool-limit = 1000

      # Enable fine grained logging of what goes on inside the implementation.
      # Be aware that this may log more than once per message sent to the actors
      # of the tcp implementation.
      trace-logging = off

      # Fully qualified config path which holds the dispatcher configuration
      # to be used for running the select() calls in the selectors
      selector-dispatcher = "akka.io.pinned-dispatcher"

      # Fully qualified config path which holds the dispatcher configuration
      # for the read/write worker actors
      worker-dispatcher = "akka.actor.internal-dispatcher"

      # Fully qualified config path which holds the dispatcher configuration
      # for the selector management actors
      management-dispatcher = "akka.actor.internal-dispatcher"
    }

    udp-connected {

      # The number of selectors to stripe the served channels over; each of
      # these will use one select loop on the selector-dispatcher.
      nr-of-selectors = 1

      # Maximum number of open channels supported by this UDP module Generally
      # UDP does not require a large number of channels, therefore it is
      # recommended to keep this setting low.
      max-channels = 4096

      # The select loop can be used in two modes:
      # - setting "infinite" will select without a timeout, hogging a thread
      # - setting a positive timeout will do a bounded select call,
      #   enabling sharing of a single thread between multiple selectors
      #   (in this case you will have to use a different configuration for the
      #   selector-dispatcher, e.g. using "type=Dispatcher" with size 1)
      # - setting it to zero means polling, i.e. calling selectNow()
      select-timeout = infinite

      # When trying to assign a new connection to a selector and the chosen
      # selector is at full capacity, retry selector choosing and assignment
      # this many times before giving up
      selector-association-retries = 10

      # The maximum number of datagrams that are read in one go,
      # higher numbers decrease latency, lower numbers increase fairness on
      # the worker-dispatcher
      receive-throughput = 3

      # The number of bytes per direct buffer in the pool used to read or write
      # network data from the kernel.
      direct-buffer-size = 128 KiB

      # The maximal number of direct buffers kept in the direct buffer pool for
      # reuse.
      direct-buffer-pool-limit = 1000

      # Enable fine grained logging of what goes on inside the implementation.
      # Be aware that this may log more than once per message sent to the actors
      # of the tcp implementation.
      trace-logging = off

      # Fully qualified config path which holds the dispatcher configuration
      # to be used for running the select() calls in the selectors
      selector-dispatcher = "akka.io.pinned-dispatcher"

      # Fully qualified config path which holds the dispatcher configuration
      # for the read/write worker actors
      worker-dispatcher = "akka.actor.internal-dispatcher"

      # Fully qualified config path which holds the dispatcher configuration
      # for the selector management actors
      management-dispatcher = "akka.actor.internal-dispatcher"
    }

    dns {
      # Fully qualified config path which holds the dispatcher configuration
      # for the manager and resolver router actors.
      # For actual router configuration see akka.actor.deployment./IO-DNS/*
      dispatcher = "akka.actor.internal-dispatcher"

      # Name of the subconfig at path akka.io.dns, see inet-address below
      #
      # Change to `async-dns` to use the new "native" DNS resolver,
      # which is also capable of resolving SRV records.
      resolver = "inet-address"

      # To-be-deprecated DNS resolver implementation which uses the Java InetAddress to resolve DNS records.
      # To be replaced by `akka.io.dns.async` which implements the DNS protocol natively and without blocking (which InetAddress does)
      inet-address {
        # Must implement akka.io.DnsProvider
        provider-object = "akka.io.InetAddressDnsProvider"

        # To set the time to cache name resolutions
        # Possible values:
        # default: sun.net.InetAddressCachePolicy.get() and getNegative()
        # forever: cache forever
        # never: no caching
        # n [time unit]: positive timeout with unit, for example 30s
        positive-ttl = default
        negative-ttl = default

        # How often to sweep out expired cache entries.
        # Note that this interval has nothing to do with TTLs
        cache-cleanup-interval = 120s
      }

      async-dns {
        provider-object = "akka.io.dns.internal.AsyncDnsProvider"

        # Set upper bound for caching successfully resolved dns entries
        # if the DNS record has a smaller TTL value than the setting that
        # will be used. Default is to use the record TTL with no cap.
        # Possible values:
        # forever: always use the minimum TTL from the found records
        # never: never cache
        # n [time unit] = cap the caching to this value
        positive-ttl = forever

        # Set how long the fact that a DNS record could not be found is
        # cached. If a new resolution is done while the fact is cached it will
        # be failed and not result in an actual DNS resolution. Default is
        # to never cache.
        # Possible values:
        # never: never cache
        # forever: cache a missing DNS record forever (you probably will not want to do this)
        # n [time unit] = cache for this long
        negative-ttl = never

        # Configures nameservers to query during DNS resolution.
        # Defaults to the nameservers that would be used by the JVM by default.
        # Set to a list of IPs to override the servers, e.g. [ "8.8.8.8", "8.8.4.4" ] for Google's servers
        # If multiple are defined then they are tried in order until one responds
        nameservers = default

        # The time that a request is allowed to live before being discarded
        # given no reply. The lower bound of this should always be the amount
        # of time to reasonably expect a DNS server to reply within.
        # If multiple name servers are provided then each gets this long to response before trying
        # the next one
        resolve-timeout = 5s

        # How often to sweep out expired cache entries.
        # Note that this interval has nothing to do with TTLs
        cache-cleanup-interval = 120s

        # Configures the list of search domains.
        # Defaults to a system dependent lookup (on Unix like OSes, will attempt to parse /etc/resolv.conf, on
        # other platforms, will not make any attempt to lookup the search domains). Set to a single domain, or
        # a list of domains, eg, [ "example.com", "example.net" ].
        search-domains = default

        # Any hosts that have a number of dots less than this will not be looked up directly, instead, a search on
        # the search domains will be tried first. This corresponds to the ndots option in /etc/resolv.conf, see
        # https://linux.die.net/man/5/resolver for more info.
        # Defaults to a system dependent lookup (on Unix like OSes, will attempt to parse /etc/resolv.conf, on
        # other platforms, will default to 1).
        ndots = default
      }
    }
  }


  # CoordinatedShutdown is an extension that will perform registered
  # tasks in the order that is defined by the phases. It is started
  # by calling CoordinatedShutdown(system).run(). This can be triggered
  # by different things, for example:
  # - JVM shutdown hook will by default run CoordinatedShutdown
  # - Cluster node will automatically run CoordinatedShutdown when it
  #   sees itself as Exiting
  # - A management console or other application specific command can
  #   run CoordinatedShutdown
  coordinated-shutdown {
    # The timeout that will be used for a phase if not specified with
    # 'timeout' in the phase
    default-phase-timeout = 5 s

    # Terminate the ActorSystem in the last phase actor-system-terminate.
    terminate-actor-system = on

    # Exit the JVM (System.exit(0)) in the last phase actor-system-terminate
    # if this is set to 'on'. It is done after termination of the
    # ActorSystem if terminate-actor-system=on, otherwise it is done
    # immediately when the last phase is reached.
    exit-jvm = off

    # Exit status to use on System.exit(int) when 'exit-jvm' is 'on'.
    exit-code = 0

    # Run the coordinated shutdown when the JVM process exits, e.g.
    # via kill SIGTERM signal (SIGINT ctrl-c doesn't work).
    # This property is related to `akka.jvm-shutdown-hooks` above.
    run-by-jvm-shutdown-hook = on

    # Run the coordinated shutdown when ActorSystem.terminate is called.
    # Enabling this and disabling terminate-actor-system is not a supported
    # combination (will throw ConfigurationException at startup).
    run-by-actor-system-terminate = on

    # When Coordinated Shutdown is triggered an instance of `Reason` is
    # required. That value can be used to override the default settings.
    # Only 'exit-jvm', 'exit-code' and 'terminate-actor-system' may be
    # overridden depending on the reason.
    reason-overrides {
      # Overrides are applied using the `reason.getClass.getName`.
      # Overrides the `exit-code` when the `Reason` is a cluster
      # Downing or a Cluster Join Unsuccessful event
      "akka.actor.CoordinatedShutdown$ClusterDowningReason$" {
        exit-code = -1
      }
      "akka.actor.CoordinatedShutdown$ClusterJoinUnsuccessfulReason$" {
        exit-code = -1
      }
    }

    #//#coordinated-shutdown-phases
    # CoordinatedShutdown is enabled by default and will run the tasks that
    # are added to these phases by individual Akka modules and user logic.
    #
    # The phases are ordered as a DAG by defining the dependencies between the phases
    # to make sure shutdown tasks are run in the right order.
    #
    # In general user tasks belong in the first few phases, but there may be use
    # cases where you would want to hook in new phases or register tasks later in
    # the DAG.
    #
    # Each phase is defined as a named config section with the
    # following optional properties:
    # - timeout=15s: Override the default-phase-timeout for this phase.
    # - recover=off: If the phase fails the shutdown is aborted
    #                and depending phases will not be executed.
    # - enabled=off: Skip all tasks registered in this phase. DO NOT use
    #                this to disable phases unless you are absolutely sure what the
    #                consequences are. Many of the built in tasks depend on other tasks
    #                having been executed in earlier phases and may break if those are disabled.
    # depends-on=[]: Run the phase after the given phases
    phases {

      # The first pre-defined phase that applications can add tasks to.
      # Note that more phases can be added in the application's
      # configuration by overriding this phase with an additional
      # depends-on.
      before-service-unbind {
      }

      # Stop accepting new incoming connections.
      # This is where you can register tasks that makes a server stop accepting new connections. Already
      # established connections should be allowed to continue and complete if possible.
      service-unbind {
        depends-on = [before-service-unbind]
      }

      # Wait for requests that are in progress to be completed.
      # This is where you register tasks that will wait for already established connections to complete, potentially
      # also first telling them that it is time to close down.
      service-requests-done {
        depends-on = [service-unbind]
      }

      # Final shutdown of service endpoints.
      # This is where you would add tasks that forcefully kill connections that are still around.
      service-stop {
        depends-on = [service-requests-done]
      }

      # Phase for custom application tasks that are to be run
      # after service shutdown and before cluster shutdown.
      before-cluster-shutdown {
        depends-on = [service-stop]
      }

      # Graceful shutdown of the Cluster Sharding regions.
      # This phase is not meant for users to add tasks to.
      cluster-sharding-shutdown-region {
        timeout = 10 s
        depends-on = [before-cluster-shutdown]
      }

      # Emit the leave command for the node that is shutting down.
      # This phase is not meant for users to add tasks to.
      cluster-leave {
        depends-on = [cluster-sharding-shutdown-region]
      }

      # Shutdown cluster singletons
      # This is done as late as possible to allow the shard region shutdown triggered in
      # the "cluster-sharding-shutdown-region" phase to complete before the shard coordinator is shut down.
      # This phase is not meant for users to add tasks to.
      cluster-exiting {
        timeout = 10 s
        depends-on = [cluster-leave]
      }

      # Wait until exiting has been completed
      # This phase is not meant for users to add tasks to.
      cluster-exiting-done {
        depends-on = [cluster-exiting]
      }

      # Shutdown the cluster extension
      # This phase is not meant for users to add tasks to.
      cluster-shutdown {
        depends-on = [cluster-exiting-done]
      }

      # Phase for custom application tasks that are to be run
      # after cluster shutdown and before ActorSystem termination.
      before-actor-system-terminate {
        depends-on = [cluster-shutdown]
      }

      # Last phase. See terminate-actor-system and exit-jvm above.
      # Don't add phases that depends on this phase because the
      # dispatcher and scheduler of the ActorSystem have been shutdown.
      # This phase is not meant for users to add tasks to.
      actor-system-terminate {
        timeout = 10 s
        depends-on = [before-actor-system-terminate]
      }
    }
    #//#coordinated-shutdown-phases
  }

}
```



### akka-actor-typed

```

akka.actor.typed {

  # List FQCN of `akka.actor.typed.ExtensionId`s which shall be loaded at actor system startup.
  # Should be on the format: 'extensions = ["com.example.MyExtId1", "com.example.MyExtId2"]' etc.
  # See the Akka Documentation for more info about Extensions
  extensions = []

  # List FQCN of extensions which shall be loaded at actor system startup.
  # Library extensions are regular extensions that are loaded at startup and are
  # available for third party library authors to enable auto-loading of extensions when
  # present on the classpath. This is done by appending entries:
  # 'library-extensions += "Extension"' in the library `reference.conf`.
  #
  # Should not be set by end user applications in 'application.conf', use the extensions property for that
  #
  library-extensions = ${?akka.actor.typed.library-extensions} []

  # Receptionist is started eagerly to allow clustered receptionist to gather remote registrations early on.
  library-extensions += "akka.actor.typed.receptionist.Receptionist$"

  # While an actor is restarted (waiting for backoff to expire and children to stop)
  # incoming messages and signals are stashed, and delivered later to the newly restarted
  # behavior. This property defines the capacity in number of messages of the stash
  # buffer. If the capacity is exceed then additional incoming messages are dropped.
  restart-stash-capacity = 1000

  # Typed mailbox defaults to the single consumber mailbox as balancing dispatcher is not supported
  default-mailbox {
    mailbox-type = "akka.dispatch.SingleConsumerOnlyUnboundedMailbox"
  }
}

# Load typed extensions by a classic extension.
akka.library-extensions += "akka.actor.typed.internal.adapter.ActorSystemAdapter$LoadTypedExtensions"

akka.actor {
  serializers {
    typed-misc = "akka.actor.typed.internal.MiscMessageSerializer"
    service-key = "akka.actor.typed.internal.receptionist.ServiceKeySerializer"
  }

  serialization-identifiers {
    "akka.actor.typed.internal.MiscMessageSerializer" = 24
    "akka.actor.typed.internal.receptionist.ServiceKeySerializer" = 26
  }

  serialization-bindings {
    "akka.actor.typed.ActorRef" = typed-misc
    "akka.actor.typed.internal.adapter.ActorRefAdapter" = typed-misc
    "akka.actor.typed.internal.receptionist.DefaultServiceKey" = service-key
  }
}

# When using Akka Typed (having akka-actor-typed in classpath) the
# akka.event.slf4j.Slf4jLogger is enabled instead of the DefaultLogger
# even though it has not been explicitly defined in `akka.loggers`
# configuration.
#
# Slf4jLogger will be used for all Akka classic logging via eventStream,
# including logging from Akka internals. The Slf4jLogger is then using
# an ordinary org.slf4j.Logger to emit the log events.
#
# The Slf4jLoggingFilter is also enabled automatically.
#
# This behavior can be disabled by setting this property to `off`.
akka.use-slf4j = on

akka.reliable-delivery {
  producer-controller {

    # To avoid head of line blocking from serialization and transfer
    # of large messages this can be enabled.
    # Large messages are chunked into pieces of the given size in bytes. The
    # chunked messages are sent separatetely and assembled on the consumer side.
    # Serialization and deserialization is performed by the ProducerController and
    # ConsumerController respectively instead of in the remote transport layer.
    chunk-large-messages = off

    durable-queue {
      # The ProducerController uses this timeout for the requests to
      # the durable queue. If there is no reply within the timeout it
      # will be retried.
      request-timeout = 3s

      # The ProducerController retries requests to the durable queue this
      # number of times before failing.
      retry-attempts = 10

      # The ProducerController retries sending the first message with this interval
      # until it has been confirmed.
      resend-first-interval = 1s
    }
  }

  consumer-controller {
    # Number of messages in flight between ProducerController and
    # ConsumerController. The ConsumerController requests for more messages
    # when half of the window has been used.
    flow-control-window = 50

    # The ConsumerController resends flow control messages to the
    # ProducerController with the resend-interval-min, and increasing
    # it gradually to resend-interval-max when idle.
    resend-interval-min = 2s
    resend-interval-max = 30s

    # If this is enabled lost messages will not be resent, but flow control is used.
    # This can be more efficient since messages don't have to be
    # kept in memory in the `ProducerController` until they have been
    # confirmed, but the drawback is that lost messages will not be delivered.
    only-flow-control = false
  }

  work-pulling {
    producer-controller = ${akka.reliable-delivery.producer-controller}
    producer-controller {
      # Limit of how many messages that can be buffered when there
      # is no demand from the consumer side.
      buffer-size = 1000

      # Ask timeout for sending message to worker until receiving Ack from worker
      internal-ask-timeout = 60s

      # Chunked messages not implemented for work-pulling yet. Override to not
      # propagate property from akka.reliable-delivery.producer-controller.
      chunk-large-messages = off
    }
  }
}
```



### akka-cluster-typed

```

############################################
# Akka Cluster Typed Reference Config File #
############################################

# This is the reference config file that contains all the default settings.
# Make your edits/overrides in your application.conf.

akka.cluster.typed.receptionist {
  # Updates with Distributed Data are done with this consistency level.
  # Possible values: local, majority, all, 2, 3, 4 (n)
  write-consistency = local

  # Period task to remove actor references that are hosted by removed nodes,
  # in case of abrupt termination.
  pruning-interval = 3 s

  # The periodic task to remove actor references that are hosted by removed nodes
  # will only remove entries older than this duration. The reason for this
  # is to avoid removing entries of nodes that haven't been visible as joining.
  prune-removed-older-than = 60 s

  # Shard the services over this many Distributed Data keys, with large amounts of different
  # service keys storing all of them in the same Distributed Data entry would lead to large updates
  # etc. instead the keys are sharded across this number of keys. This must be the same on all nodes
  # in a cluster, changing it requires a full cluster restart (stopping all nodes before starting them again)
  distributed-key-count = 5

  # Settings for the Distributed Data replicator used by Receptionist.
  # Same layout as akka.cluster.distributed-data.
  distributed-data = ${akka.cluster.distributed-data}
  # make sure that by default it's for all roles (Play loads config in different way)
  distributed-data.role = ""
}

akka.cluster.ddata.typed {
  # The timeout to use for ask operations in ReplicatorMessageAdapter.
  # This should be longer than the timeout given in Replicator.WriteConsistency and
  # Replicator.ReadConsistency. The replicator will always send a reply within those
  # timeouts so the unexpected ask timeout should not occur, but for cleanup in a
  # failure situation it must still exist.
  # If askUpdate, askGet or askDelete takes longer then this timeout a
  # java.util.concurrent.TimeoutException will be thrown by the requesting actor and
  # may be handled by supervision.
  replicator-message-adapter-unexpected-ask-timeout = 20 s
}

akka {
  actor {
    serialization-identifiers {
      "akka.cluster.typed.internal.AkkaClusterTypedSerializer" = 28
      "akka.cluster.typed.internal.delivery.ReliableDeliverySerializer" = 36
    }
    serializers {
      typed-cluster = "akka.cluster.typed.internal.AkkaClusterTypedSerializer"
      reliable-delivery = "akka.cluster.typed.internal.delivery.ReliableDeliverySerializer"
    }
    serialization-bindings {
      "akka.cluster.typed.internal.receptionist.ClusterReceptionist$Entry" = typed-cluster
      "akka.actor.typed.internal.pubsub.TopicImpl$MessagePublished" = typed-cluster
      "akka.actor.typed.delivery.internal.DeliverySerializable" = reliable-delivery
    }
  }
  cluster.configuration-compatibility-check.checkers {
    receptionist = "akka.cluster.typed.internal.receptionist.ClusterReceptionistConfigCompatChecker"
  }
}
```



### akka-cluster

```
######################################
# Akka Cluster Reference Config File #
######################################

# This is the reference config file that contains all the default settings.
# Make your edits/overrides in your application.conf.

akka {

  cluster {
    # Initial contact points of the cluster.
    # The nodes to join automatically at startup.
    # Comma separated full URIs defined by a string on the form of
    # "akka://system@hostname:port"
    # Leave as empty if the node is supposed to be joined manually.
    seed-nodes = []

    # How long to wait for one of the seed nodes to reply to initial join request.
    # When this is the first seed node and there is no positive reply from the other
    # seed nodes within this timeout it will join itself to bootstrap the cluster.
    # When this is not the first seed node the join attempts will be performed with
    # this interval.  
    seed-node-timeout = 5s

    # If a join request fails it will be retried after this period.
    # Disable join retry by specifying "off".
    retry-unsuccessful-join-after = 10s
    
    # The joining of given seed nodes will by default be retried indefinitely until
    # a successful join. That process can be aborted if unsuccessful by defining this
    # timeout. When aborted it will run CoordinatedShutdown, which by default will
    # terminate the ActorSystem. CoordinatedShutdown can also be configured to exit
    # the JVM. It is useful to define this timeout if the seed-nodes are assembled
    # dynamically and a restart with new seed-nodes should be tried after unsuccessful
    # attempts.   
    shutdown-after-unsuccessful-join-seed-nodes = off

    # Time margin after which shards or singletons that belonged to a downed/removed
    # partition are created in surviving partition. The purpose of this margin is that
    # in case of a network partition the persistent actors in the non-surviving partitions
    # must be stopped before corresponding persistent actors are started somewhere else.
    # This is useful if you implement downing strategies that handle network partitions,
    # e.g. by keeping the larger side of the partition and shutting down the smaller side.
    # Disable with "off" or specify a duration to enable.
    #
    # When using the `akka.cluster.sbr.SplitBrainResolver` as downing provider it will use
    # the akka.cluster.split-brain-resolver.stable-after as the default down-removal-margin
    # if this down-removal-margin is undefined.
    down-removal-margin = off

    # Pluggable support for downing of nodes in the cluster.
    # If this setting is left empty the `NoDowning` provider is used and no automatic downing will be performed.
    #
    # If specified the value must be the fully qualified class name of a subclass of
    # `akka.cluster.DowningProvider` having a public one argument constructor accepting an `ActorSystem`
    downing-provider-class = ""

    # Artery only setting
    # When a node has been gracefully removed, let this time pass (to allow for example
    # cluster singleton handover to complete) and then quarantine the removed node.
    quarantine-removed-node-after = 5s

    # If this is set to "off", the leader will not move 'Joining' members to 'Up' during a network
    # split. This feature allows the leader to accept 'Joining' members to be 'WeaklyUp'
    # so they become part of the cluster even during a network split. The leader will
    # move `Joining` members to 'WeaklyUp' after this configured duration without convergence.
    # The leader will move 'WeaklyUp' members to 'Up' status once convergence has been reached.
    allow-weakly-up-members = 7s

    # The roles of this member. List of strings, e.g. roles = ["A", "B"].
    # The roles are part of the membership information and can be used by
    # routers or other services to distribute work to certain member types,
    # e.g. front-end and back-end nodes.
    # Roles are not allowed to start with "dc-" as that is reserved for the
    # special role assigned from the data-center a node belongs to (see the
    # multi-data-center section below)
    roles = []
    
    # Run the coordinated shutdown from phase 'cluster-shutdown' when the cluster
    # is shutdown for other reasons than when leaving, e.g. when downing. This
    # will terminate the ActorSystem when the cluster extension is shutdown.
    run-coordinated-shutdown-when-down = on

    role {
      # Minimum required number of members of a certain role before the leader
      # changes member status of 'Joining' members to 'Up'. Typically used together
      # with 'Cluster.registerOnMemberUp' to defer some action, such as starting
      # actors, until the cluster has reached a certain size.
      # E.g. to require 2 nodes with role 'frontend' and 3 nodes with role 'backend':
      #   frontend.min-nr-of-members = 2
      #   backend.min-nr-of-members = 3
      #<role-name>.min-nr-of-members = 1
    }

    # Application version of the deployment. Used by rolling update features
    # to distinguish between old and new nodes. The typical convention is to use
    # 3 digit version numbers `major.minor.patch`, but 1 or two digits are also
    # supported.
    #
    # If no `.` is used it is interpreted as a single digit version number or as
    # plain alphanumeric if it couldn't be parsed as a number.
    #
    # It may also have a qualifier at the end for 2 or 3 digit version numbers such
    # as "1.2-RC1".
    # For 1 digit with qualifier, 1-RC1, it is interpreted as plain alphanumeric.
    #
    # It has support for https://github.com/dwijnand/sbt-dynver format with `+` or
    # `-` separator. The number of commits from the tag is handled as a numeric part.
    # For example `1.0.0+3-73475dce26` is less than `1.0.10+10-ed316bd024` (3 < 10).
    app-version = "0.0.0"

    # Minimum required number of members before the leader changes member status
    # of 'Joining' members to 'Up'. Typically used together with
    # 'Cluster.registerOnMemberUp' to defer some action, such as starting actors,
    # until the cluster has reached a certain size.
    min-nr-of-members = 1

    # Enable/disable info level logging of cluster events.
    # These are logged with logger name `akka.cluster.Cluster`.
    log-info = on

    # Enable/disable verbose info-level logging of cluster events
    # for temporary troubleshooting. Defaults to 'off'.
    # These are logged with logger name `akka.cluster.Cluster`.
    log-info-verbose = off

    # Enable or disable JMX MBeans for management of the cluster
    jmx.enabled = on

    # Enable or disable multiple JMX MBeans in the same JVM
    # If this is disabled, the MBean Object name is "akka:type=Cluster"
    # If this is enabled, them MBean Object names become "akka:type=Cluster,port=$clusterPortNumber"
    jmx.multi-mbeans-in-same-jvm = off

    # how long should the node wait before starting the periodic tasks
    # maintenance tasks?
    periodic-tasks-initial-delay = 1s

    # how often should the node send out gossip information?
    gossip-interval = 1s
    
    # discard incoming gossip messages if not handled within this duration
    gossip-time-to-live = 2s

    # how often should the leader perform maintenance tasks?
    leader-actions-interval = 1s

    # how often should the node move nodes, marked as unreachable by the failure
    # detector, out of the membership ring?
    unreachable-nodes-reaper-interval = 1s

    # How often the current internal stats should be published.
    # A value of 0s can be used to always publish the stats, when it happens.
    # Disable with "off".
    publish-stats-interval = off

    # The id of the dispatcher to use for cluster actors.
    # If specified you need to define the settings of the actual dispatcher.
    use-dispatcher = "akka.actor.internal-dispatcher"

    # Gossip to random node with newer or older state information, if any with
    # this probability. Otherwise Gossip to any random live node.
    # Probability value is between 0.0 and 1.0. 0.0 means never, 1.0 means always.
    gossip-different-view-probability = 0.8
    
    # Reduced the above probability when the number of nodes in the cluster
    # greater than this value.
    reduce-gossip-different-view-probability = 400

    # When a node is removed the removal is marked with a tombstone
    # which is kept at least this long, after which it is pruned, if there is a partition
    # longer than this it could lead to removed nodes being re-added to the cluster
    prune-gossip-tombstones-after = 24h

    # Settings for the Phi accrual failure detector (http://www.jaist.ac.jp/~defago/files/pdf/IS_RR_2004_010.pdf
    # [Hayashibara et al]) used by the cluster subsystem to detect unreachable
    # members.
    # The default PhiAccrualFailureDetector will trigger if there are no heartbeats within
    # the duration heartbeat-interval + acceptable-heartbeat-pause + threshold_adjustment,
    # i.e. around 5.5 seconds with default settings.
    failure-detector {

      # FQCN of the failure detector implementation.
      # It must implement akka.remote.FailureDetector and have
      # a public constructor with a com.typesafe.config.Config and
      # akka.actor.EventStream parameter.
      implementation-class = "akka.remote.PhiAccrualFailureDetector"

      # How often keep-alive heartbeat messages should be sent to each connection.
      heartbeat-interval = 1 s

      # Defines the failure detector threshold.
      # A low threshold is prone to generate many wrong suspicions but ensures
      # a quick detection in the event of a real crash. Conversely, a high
      # threshold generates fewer mistakes but needs more time to detect
      # actual crashes.
      threshold = 8.0

      # Number of the samples of inter-heartbeat arrival times to adaptively
      # calculate the failure timeout for connections.
      max-sample-size = 1000

      # Minimum standard deviation to use for the normal distribution in
      # AccrualFailureDetector. Too low standard deviation might result in
      # too much sensitivity for sudden, but normal, deviations in heartbeat
      # inter arrival times.
      min-std-deviation = 100 ms

      # Number of potentially lost/delayed heartbeats that will be
      # accepted before considering it to be an anomaly.
      # This margin is important to be able to survive sudden, occasional,
      # pauses in heartbeat arrivals, due to for example garbage collect or
      # network drop.
      acceptable-heartbeat-pause = 3 s

      # Number of member nodes that each member will send heartbeat messages to,
      # i.e. each node will be monitored by this number of other nodes.
      monitored-by-nr-of-members = 9
      
      # After the heartbeat request has been sent the first failure detection
      # will start after this period, even though no heartbeat message has
      # been received.
      expected-response-after = 1 s

    }

    # Configures multi-dc specific heartbeating and other mechanisms,
    # many of them have a direct counter-part in "one datacenter mode",
    # in which case these settings would not be used at all - they only apply,
    # if your cluster nodes are configured with at-least 2 different `akka.cluster.data-center` values.
    multi-data-center {

      # Defines which data center this node belongs to. It is typically used to make islands of the
      # cluster that are colocated. This can be used to make the cluster aware that it is running
      # across multiple availability zones or regions. It can also be used for other logical
      # grouping of nodes.
      self-data-center = "default"


      # Try to limit the number of connections between data centers. Used for gossip and heartbeating.
      # This will not limit connections created for the messaging of the application.
      # If the cluster does not span multiple data centers, this value has no effect.
      cross-data-center-connections = 5

      # The n oldest nodes in a data center will choose to gossip to another data center with
      # this probability. Must be a value between 0.0 and 1.0 where 0.0 means never, 1.0 means always.
      # When a data center is first started (nodes < 5) a higher probability is used so other data
      # centers find out about the new nodes more quickly
      cross-data-center-gossip-probability = 0.2

      failure-detector {
        # FQCN of the failure detector implementation.
        # It must implement akka.remote.FailureDetector and have
        # a public constructor with a com.typesafe.config.Config and
        # akka.actor.EventStream parameter.
        implementation-class = "akka.remote.DeadlineFailureDetector"
  
        # Number of potentially lost/delayed heartbeats that will be
        # accepted before considering it to be an anomaly.
        # This margin is important to be able to survive sudden, occasional,
        # pauses in heartbeat arrivals, due to for example garbage collect or
        # network drop.
        acceptable-heartbeat-pause = 10 s
        
        # How often keep-alive heartbeat messages should be sent to each connection.
        heartbeat-interval = 3 s
  
        # After the heartbeat request has been sent the first failure detection
        # will start after this period, even though no heartbeat message has
        # been received.
        expected-response-after = 1 s
      }
    }

    # If the tick-duration of the default scheduler is longer than the
    # tick-duration configured here a dedicated scheduler will be used for
    # periodic tasks of the cluster, otherwise the default scheduler is used.
    # See akka.scheduler settings for more details.
    scheduler {
      tick-duration = 33ms
      ticks-per-wheel = 512
    }

    debug {
      # Log heartbeat events (very verbose, useful mostly when debugging heartbeating issues).
      # These are logged with logger name `akka.cluster.ClusterHeartbeat`.
      verbose-heartbeat-logging = off

      # log verbose details about gossip
      verbose-gossip-logging = off
    }

    configuration-compatibility-check {

      # Enforce configuration compatibility checks when joining a cluster.
      # Set to off to allow joining nodes to join a cluster even when configuration incompatibilities are detected or
      # when the cluster does not support this feature. Compatibility checks are always performed and warning and
      # error messages are logged.
      #
      # This is particularly useful for rolling updates on clusters that do not support that feature. Since the old
      # cluster won't be able to send the compatibility confirmation to the joining node, the joining node won't be able
      # to 'know' if its allowed to join.
      enforce-on-join = on

      # Add named entry to this section with fully qualified class name of the JoinConfigCompatChecker
      # to enable.
      # Checkers defined in reference.conf can be disabled by application by using empty string value
      # for the named entry.
      checkers {
        akka-cluster = "akka.cluster.JoinConfigCompatCheckCluster"
      }

      # Some configuration properties might not be appropriate to transfer between nodes
      # and such properties can be excluded from the configuration compatibility check by adding
      # the paths of the properties to this list. Sensitive paths are grouped by key. Modules and third-party libraries
      # can define their own set of sensitive paths without clashing with each other (as long they use unique keys).
      #
      # All properties starting with the paths defined here are excluded, i.e. you can add the path of a whole
      # section here to skip everything inside that section.
      sensitive-config-paths {
        akka = [
          "user.home", "user.name", "user.dir",
          "socksNonProxyHosts", "http.nonProxyHosts", "ftp.nonProxyHosts",
          "akka.remote.secure-cookie",
          "akka.remote.classic.netty.ssl.security",
          # Pre 2.6 path, keep around to avoid sending things misconfigured with old paths
          "akka.remote.netty.ssl.security",
          "akka.remote.artery.ssl"
        ]
      }

    }
  }

  actor.deployment.default.cluster {
    # enable cluster aware router that deploys to nodes in the cluster
    enabled = off

    # Maximum number of routees that will be deployed on each cluster
    # member node.
    # Note that max-total-nr-of-instances defines total number of routees, but
    # number of routees per node will not be exceeded, i.e. if you
    # define max-total-nr-of-instances = 50 and max-nr-of-instances-per-node = 2
    # it will deploy 2 routees per new member in the cluster, up to
    # 25 members.
    max-nr-of-instances-per-node = 1
    
    # Maximum number of routees that will be deployed, in total
    # on all nodes. See also description of max-nr-of-instances-per-node.
    # For backwards compatibility reasons, nr-of-instances
    # has the same purpose as max-total-nr-of-instances for cluster
    # aware routers and nr-of-instances (if defined by user) takes
    # precedence over max-total-nr-of-instances. 
    max-total-nr-of-instances = 10000

    # Defines if routees are allowed to be located on the same node as
    # the head router actor, or only on remote nodes.
    # Useful for master-worker scenario where all routees are remote.
    allow-local-routees = on

    # Use members with all specified roles, or all members if undefined or empty.
    use-roles = []

    # Deprecated, since Akka 2.5.4, replaced by use-roles
    # Use members with specified role, or all members if undefined or empty.
    use-role = ""
  }

  # Protobuf serializer for cluster messages
  actor {
    serializers {
      akka-cluster = "akka.cluster.protobuf.ClusterMessageSerializer"
    }

    serialization-bindings {
      "akka.cluster.ClusterMessage" = akka-cluster
      "akka.cluster.routing.ClusterRouterPool" = akka-cluster
    }
    
    serialization-identifiers {
      "akka.cluster.protobuf.ClusterMessageSerializer" = 5
    }
    
  }

}

#//#split-brain-resolver

# To enable the split brain resolver you first need to enable the provider in your application.conf:
# akka.cluster.downing-provider-class = "akka.cluster.sbr.SplitBrainResolverProvider"

akka.cluster.split-brain-resolver {
  # Select one of the available strategies (see descriptions below):
  # static-quorum, keep-majority, keep-oldest, down-all, lease-majority
  active-strategy = keep-majority

  #//#stable-after
  # Time margin after which shards or singletons that belonged to a downed/removed
  # partition are created in surviving partition. The purpose of this margin is that
  # in case of a network partition the persistent actors in the non-surviving partitions
  # must be stopped before corresponding persistent actors are started somewhere else.
  # This is useful if you implement downing strategies that handle network partitions,
  # e.g. by keeping the larger side of the partition and shutting down the smaller side.
  # Decision is taken by the strategy when there has been no membership or
  # reachability changes for this duration, i.e. the cluster state is stable.
  stable-after = 20s
  #//#stable-after

  # When reachability observations by the failure detector are changed the SBR decisions
  # are deferred until there are no changes within the 'stable-after' duration.
  # If this continues for too long it might be an indication of an unstable system/network
  # and it could result in delayed or conflicting decisions on separate sides of a network
  # partition.
  # As a precaution for that scenario all nodes are downed if no decision is made within
  # `stable-after + down-all-when-unstable` from the first unreachability event.
  # The measurement is reset if all unreachable have been healed, downed or removed, or
  # if there are no changes within `stable-after * 2`.
  # The value can be on, off, or a duration.
  # By default it is 'on' and then it is derived to be 3/4 of stable-after, but not less than
  # 4 seconds.
  down-all-when-unstable = on

}
#//#split-brain-resolver

# Down the unreachable nodes if the number of remaining nodes are greater than or equal to
# the given 'quorum-size'. Otherwise down the reachable nodes, i.e. it will shut down that
# side of the partition. In other words, the 'size' defines the minimum number of nodes
# that the cluster must have to be operational. If there are unreachable nodes when starting
# up the cluster, before reaching this limit, the cluster may shutdown itself immediately.
# This is not an issue if you start all nodes at approximately the same time.
#
# Note that you must not add more members to the cluster than 'quorum-size * 2 - 1', because
# then both sides may down each other and thereby form two separate clusters. For example,
# quorum-size configured to 3 in a 6 node cluster may result in a split where each side
# consists of 3 nodes each, i.e. each side thinks it has enough nodes to continue by
# itself. A warning is logged if this recommendation is violated.
#//#static-quorum
akka.cluster.split-brain-resolver.static-quorum {
  # minimum number of nodes that the cluster must have
  quorum-size = undefined

  # if the 'role' is defined the decision is based only on members with that 'role'
  role = ""
}
#//#static-quorum

# Down the unreachable nodes if the current node is in the majority part based the last known
# membership information. Otherwise down the reachable nodes, i.e. the own part. If the
# the parts are of equal size the part containing the node with the lowest address is kept.
# Note that if there are more than two partitions and none is in majority each part
# will shutdown itself, terminating the whole cluster.
#//#keep-majority
akka.cluster.split-brain-resolver.keep-majority {
  # if the 'role' is defined the decision is based only on members with that 'role'
  role = ""
}
#//#keep-majority

# Down the part that does not contain the oldest member (current singleton).
#
# There is one exception to this rule if 'down-if-alone' is defined to 'on'.
# Then, if the oldest node has partitioned from all other nodes the oldest
# will down itself and keep all other nodes running. The strategy will not
# down the single oldest node when it is the only remaining node in the cluster.
#
# Note that if the oldest node crashes the others will remove it from the cluster
# when 'down-if-alone' is 'on', otherwise they will down themselves if the
# oldest node crashes, i.e. shutdown the whole cluster together with the oldest node.
#//#keep-oldest
akka.cluster.split-brain-resolver.keep-oldest {
  # Enable downing of the oldest node when it is partitioned from all other nodes
  down-if-alone = on

  # if the 'role' is defined the decision is based only on members with that 'role',
  # i.e. using the oldest member (singleton) within the nodes with that role
  role = ""
}
#//#keep-oldest

# Keep the part that can acquire the lease, and down the other part.
# Best effort is to keep the side that has most nodes, i.e. the majority side.
# This is achieved by adding a delay before trying to acquire the lease on the
# minority side.
#//#lease-majority
akka.cluster.split-brain-resolver.lease-majority {
  lease-implementation = ""

  # The recommended format for the lease name is "<service-name>-akka-sbr".
  # When lease-name is not defined, the name will be set to "<actor-system-name>-akka-sbr"
  lease-name = ""

  # This delay is used on the minority side before trying to acquire the lease,
  # as an best effort to try to keep the majority side.
  acquire-lease-delay-for-minority = 2s

  # Release the lease after this duration.
  release-after = 40s

  # If the 'role' is defined the majority/minority is based only on members with that 'role'.
  role = ""
}
#//#lease-majority
```



### akka-discovery

```

######################################################
# Akka Discovery Config                              #
######################################################

akka.actor.deployment {
  "/SD-DNS/async-dns" {
    mailbox = "unbounded"
    router = "round-robin-pool"
    nr-of-instances = 1
  }
}

akka.discovery {

  # Users MUST configure this value to set the default discovery method.
  #
  # The value can be an implementation config path name, such as "akka-dns",
  # which would attempt to resolve as `akka.discovery.akka-dns` which is expected
  # to contain a `class` setting. As fallback, the root `akka-dns` setting scope
  # would be used. If none of those contained a `class` setting, then the value is
  # assumed to be a class name, and an attempt is made to instantiate it.
  method = "<method>"

  # Config based service discovery
  config {
    class = akka.discovery.config.ConfigServiceDiscovery

    # Location of the services in configuration
    services-path = "akka.discovery.config.services"

    # A map of services to resolve from configuration.
    # See docs for more examples.
    # A list of endpoints with host/port where port is optional e.g.
    # services {
    #  service1 {
    #    endpoints = [
    #      {
    #        host = "cat.com"
    #        port = 1233
    #      },
    #      {
    #        host = "dog.com"
    #      }
    #    ]
    #  },
    #  service2 {
    #    endpoints = [
    #    {
    #        host = "fish.com"
    #        port = 1233
    #      }
    #    ]
    #  }
    # }
    services = {

    }
  }

  # Aggregate multiple service discovery mechanisms
  aggregate {
    class = akka.discovery.aggregate.AggregateServiceDiscovery

    # List of service discovery methods to try in order. E.g config then fall back to DNS
    # ["config", "akka-dns"]
    discovery-methods = []

  }

  # DNS based service discovery
  akka-dns {
    class = akka.discovery.dns.DnsServiceDiscovery
  }
}
```



### akka-coordination

```

akka.coordination {

  # Defaults for any lease implementation that doesn't include these properties
  lease {

    # FQCN of the implementation of the Lease
    lease-class = ""

    #defaults
    # if the node that acquired the leases crashes, how long should the lease be held before another owner can get it
    heartbeat-timeout = 120s

    # interval for communicating with the third party to confirm the lease is still held
    heartbeat-interval = 12s

    # lease implementations are expected to time out acquire and release calls or document
    # that they do not implement an operation timeout
    lease-operation-timeout = 5s

    #defaults
  }
}
```



### akka-multi-node-testkit

```

#############################################
# Akka Remote Testing Reference Config File #
#############################################

# This is the reference config file that contains all the default settings.
# Make your edits/overrides in your application.conf.

akka {
  testconductor {

    # Timeout for joining a barrier: this is the maximum time any participants
    # waits for everybody else to join a named barrier.
    barrier-timeout = 30s
    
    # Timeout for interrogation of TestConductor’s Controller actor
    query-timeout = 10s
    
    # Threshold for packet size in time unit above which the failure injector will
    # split the packet and deliver in smaller portions; do not give value smaller
    # than HashedWheelTimer resolution (would not make sense)
    packet-split-threshold = 100ms
    
    # amount of time for the ClientFSM to wait for the connection to the conductor
    # to be successful
    connect-timeout = 20s
    
    # Number of connect attempts to be made to the conductor controller
    client-reconnects = 30
    
    # minimum time interval which is to be inserted between reconnect attempts
    reconnect-backoff = 1s

    netty {
      # (I&O) Used to configure the number of I/O worker threads on server sockets
      server-socket-worker-pool {
        # Min number of threads to cap factor-based number to
        pool-size-min = 1

        # The pool size factor is used to determine thread pool size
        # using the following formula: ceil(available processors * factor).
        # Resulting size is then bounded by the pool-size-min and
        # pool-size-max values.
        pool-size-factor = 1.0

        # Max number of threads to cap factor-based number to
        pool-size-max = 2
      }

      # (I&O) Used to configure the number of I/O worker threads on client sockets
      client-socket-worker-pool {
        # Min number of threads to cap factor-based number to
        pool-size-min = 1

        # The pool size factor is used to determine thread pool size
        # using the following formula: ceil(available processors * factor).
        # Resulting size is then bounded by the pool-size-min and
        # pool-size-max values.
        pool-size-factor = 1.0

        # Max number of threads to cap factor-based number to
        pool-size-max = 2
      }
    }
  }
}
```



### akka-persistence-typed

```

akka.actor {

  serialization-identifiers."akka.persistence.typed.serialization.ReplicatedEventSourcingSerializer" = 40

  serializers.replicated-event-sourcing = "akka.persistence.typed.serialization.ReplicatedEventSourcingSerializer"

  serialization-bindings {
    "akka.persistence.typed.internal.VersionVector" = replicated-event-sourcing
    "akka.persistence.typed.crdt.Counter" = replicated-event-sourcing
    "akka.persistence.typed.crdt.Counter$Updated" = replicated-event-sourcing
    "akka.persistence.typed.crdt.ORSet" = replicated-event-sourcing
    "akka.persistence.typed.crdt.ORSet$DeltaOp" = replicated-event-sourcing
    "akka.persistence.typed.internal.ReplicatedEventMetadata" = replicated-event-sourcing
    "akka.persistence.typed.internal.ReplicatedSnapshotMetadata" = replicated-event-sourcing
    "akka.persistence.typed.internal.PublishedEventImpl" = replicated-event-sourcing
  }
}

akka.persistence.typed {

  # Persistent actors stash while recovering or persisting events,
  # this setting configures the default capacity of this stash.
  #
  # Stashing is always bounded to the size that is defined in this setting.
  # You can set it to large values, however "unbounded" buffering is not supported.
  # Negative or 0 values are not allowed.
  stash-capacity = 4096

  # Configure how to react when the event sourced stash overflows. This can happen in two scenarios:
  # when a event sourced actor is doing recovery, persisting or snapshotting and it gets more than
  # 'stash-capacity' commands, or if more than 'stash-capacity' commands are manually stashed with the
  # 'stash' effect.
  #
  # Possible options
  # - drop - the message is published as a akka.actor.typed.Dropped message on the event bus
  # - fail - an exception is thrown so that the actor is failed
  stash-overflow-strategy = "drop"

  # enables automatic DEBUG level logging of messages stashed automatically by an EventSourcedBehavior,
  # this may happen while it receives commands while it is recovering events or while it is persisting events
  log-stashing = off

  # By default, internal event sourced behavior logging are sent to
  # akka.persistence.typed.internal.EventSourcedBehaviorImpl
  # this can be changed by setting this to 'true' in which case the internal logging is sent to
  # the actor context logger.
  use-context-logger-for-internal-logging = false
}

akka.reliable-delivery {
  producer-controller {
    event-sourced-durable-queue {
      # Max duration for the exponential backoff for persist failures.
      restart-max-backoff = 10s

      # Snapshot after this number of events. See RetentionCriteria.
      snapshot-every = 1000

      # Number of snapshots to keep. See RetentionCriteria.
      keep-n-snapshots = 2

      # Delete events after snapshotting. See RetentionCriteria.
      delete-events = on

      # Cleanup entries that haven't be used for this duration.
      cleanup-unused-after = 3600s

      # The journal plugin to use, by default it will use the plugin configured by
      # `akka.persistence.journal.plugin`.
      journal-plugin-id = ""

      # The journal plugin to use, by default it will use the plugin configured by
      # `akka.persistence.snapshot-store.plugin`.
      snapshot-plugin-id = ""
    }
  }
}
```



### akka-persistence

```

###########################################################
# Akka Persistence Extension Reference Configuration File #
###########################################################

# This is the reference config file that contains all the default settings.
# Make your edits in your application.conf in order to override these settings.

# Directory of persistence journal and snapshot store plugins is available at the 
# Akka Community Projects page http://akka.io/community/

# Default persistence extension settings.
akka.persistence {

    # When starting many persistent actors at the same time the journal
    # and its data store is protected from being overloaded by limiting number
    # of recoveries that can be in progress at the same time. When
    # exceeding the limit the actors will wait until other recoveries have
    # been completed.   
    max-concurrent-recoveries = 50

    # Fully qualified class name providing a default internal stash overflow strategy.
    # It needs to be a subclass of akka.persistence.StashOverflowStrategyConfigurator.
    # The default strategy throws StashOverflowException.
    internal-stash-overflow-strategy = "akka.persistence.ThrowExceptionConfigurator"
    journal {
        # Absolute path to the journal plugin configuration entry used by 
        # persistent actor by default.
        # Persistent actor can override `journalPluginId` method 
        # in order to rely on a different journal plugin.
        plugin = ""
        # List of journal plugins to start automatically. Use "" for the default journal plugin.
        auto-start-journals = []
    }
    snapshot-store {
        # Absolute path to the snapshot plugin configuration entry used by
        # persistent actor by default.
        # Persistent actor can override `snapshotPluginId` method
        # in order to rely on a different snapshot plugin.
        # It is not mandatory to specify a snapshot store plugin.
        # If you don't use snapshots you don't have to configure it.
        # Note that Cluster Sharding is using snapshots, so if you
        # use Cluster Sharding you need to define a snapshot store plugin. 
        plugin = ""
        # List of snapshot stores to start automatically. Use "" for the default snapshot store.
        auto-start-snapshot-stores = []
    }
    # used as default-snapshot store if no plugin configured 
    # (see `akka.persistence.snapshot-store`)
    no-snapshot-store {
      class = "akka.persistence.snapshot.NoSnapshotStore"
    }
    # Default reliable delivery settings.
    at-least-once-delivery {
        # Interval between re-delivery attempts.
        redeliver-interval = 5s
        # Maximum number of unconfirmed messages that will be sent in one 
        # re-delivery burst.
        redelivery-burst-limit = 10000
        # After this number of delivery attempts a 
        # `ReliableRedelivery.UnconfirmedWarning`, message will be sent to the actor.
        warn-after-number-of-unconfirmed-attempts = 5
        # Maximum number of unconfirmed messages that an actor with 
        # AtLeastOnceDelivery is allowed to hold in memory.
        max-unconfirmed-messages = 100000
    }
    # Default persistent extension thread pools.
    dispatchers {
        # Dispatcher used by every plugin which does not declare explicit
        # `plugin-dispatcher` field.
        default-plugin-dispatcher {
            type = PinnedDispatcher
            executor = "thread-pool-executor"
        }
        # Default dispatcher for message replay.
        default-replay-dispatcher {
            type = Dispatcher
            executor = "fork-join-executor"
            fork-join-executor {
                parallelism-min = 2
                parallelism-max = 8
            }
        }
        # Default dispatcher for streaming snapshot IO
        default-stream-dispatcher {
            type = Dispatcher
            executor = "fork-join-executor"
            fork-join-executor {
                parallelism-min = 2
                parallelism-max = 8
            }
        }
    }

    # Fallback settings for journal plugin configurations.
    # These settings are used if they are not defined in plugin config section.
    journal-plugin-fallback {

      # Fully qualified class name providing journal plugin api implementation.
      # It is mandatory to specify this property.
      # The class must have a constructor without parameters or constructor with
      # one `com.typesafe.config.Config` parameter.
      class = ""

      # Dispatcher for the plugin actor.
      plugin-dispatcher = "akka.persistence.dispatchers.default-plugin-dispatcher"

      # Dispatcher for message replay.
      replay-dispatcher = "akka.persistence.dispatchers.default-replay-dispatcher"

      # Removed: used to be the Maximum size of a persistent message batch written to the journal.
      # Now this setting is without function, PersistentActor will write as many messages
      # as it has accumulated since the last write.
      max-message-batch-size = 200

      # If there is more time in between individual events gotten from the journal
      # recovery than this the recovery will fail.
      # Note that it also affects reading the snapshot before replaying events on
      # top of it, even though it is configured for the journal.
      recovery-event-timeout = 30s

      circuit-breaker {
        max-failures = 10
        call-timeout = 10s
        reset-timeout = 30s
      }

      # The replay filter can detect a corrupt event stream by inspecting
      # sequence numbers and writerUuid when replaying events.
      replay-filter {
        # What the filter should do when detecting invalid events.
        # Supported values:
        # `repair-by-discard-old` : discard events from old writers,
        #                           warning is logged
        # `fail` : fail the replay, error is logged
        # `warn` : log warning but emit events untouched
        # `off` : disable this feature completely
        mode = repair-by-discard-old

        # It uses a look ahead buffer for analyzing the events.
        # This defines the size (in number of events) of the buffer.
        window-size = 100

        # How many old writerUuid to remember
        max-old-writers = 10

        # Set this to `on` to enable detailed debug logging of each
        # replayed event.
        debug = off
      }
    }

    # Fallback settings for snapshot store plugin configurations
    # These settings are used if they are not defined in plugin config section.
    snapshot-store-plugin-fallback {

      # Fully qualified class name providing snapshot store plugin api
      # implementation. It is mandatory to specify this property if
      # snapshot store is enabled.
      # The class must have a constructor without parameters or constructor with
      # one `com.typesafe.config.Config` parameter.
      class = ""

      # Dispatcher for the plugin actor.
      plugin-dispatcher = "akka.persistence.dispatchers.default-plugin-dispatcher"

      circuit-breaker {
        max-failures = 5
        call-timeout = 20s
        reset-timeout = 60s
      }

      # Set this to true if successful loading of snapshot is not necessary.
      # This can be useful when it is alright to ignore snapshot in case of
      # for example deserialization errors. When snapshot loading fails it will instead
      # recover by replaying all events.
      # Don't set to true if events are deleted because that would
      # result in wrong recovered state if snapshot load fails.
      snapshot-is-optional = false

    }

  fsm {
    # PersistentFSM saves snapshots after this number of persistent
    # events. Snapshots are used to reduce recovery times.
    # When you disable this feature, specify snapshot-after = off.
    # To enable the feature, specify a number like snapshot-after = 1000
    # which means a snapshot is taken after persisting every 1000 events.
    snapshot-after = off
  }
}

# Protobuf serialization for the persistent extension messages.
akka.actor {
    serializers {
        akka-persistence-message = "akka.persistence.serialization.MessageSerializer"
        akka-persistence-snapshot = "akka.persistence.serialization.SnapshotSerializer"
    }
    serialization-bindings {
        "akka.persistence.serialization.Message" = akka-persistence-message
        "akka.persistence.serialization.Snapshot" = akka-persistence-snapshot
    }
    serialization-identifiers {
        "akka.persistence.serialization.MessageSerializer" = 7
        "akka.persistence.serialization.SnapshotSerializer" = 8
    }
}


###################################################
# Persistence plugins included with the extension #
###################################################

# In-memory journal plugin.
akka.persistence.journal.inmem {
    # Class name of the plugin.
    class = "akka.persistence.journal.inmem.InmemJournal"
    # Dispatcher for the plugin actor.
    plugin-dispatcher = "akka.actor.default-dispatcher"

    # Turn this on to test serialization of the events
    test-serialization = off
}

# Local file system snapshot store plugin.
akka.persistence.snapshot-store.local {
    # Class name of the plugin.
    class = "akka.persistence.snapshot.local.LocalSnapshotStore"
    # Dispatcher for the plugin actor.
    plugin-dispatcher = "akka.persistence.dispatchers.default-plugin-dispatcher"
    # Dispatcher for streaming snapshot IO.
    stream-dispatcher = "akka.persistence.dispatchers.default-stream-dispatcher"
    # Storage location of snapshot files.
    dir = "snapshots"
    # Number load attempts when recovering from the latest snapshot fails
    # yet older snapshot files are available. Each recovery attempt will try
    # to recover using an older than previously failed-on snapshot file 
    # (if any are present). If all attempts fail the recovery will fail and
    # the persistent actor will be stopped.
    max-load-attempts = 3
}

# LevelDB journal plugin.
# Note: this plugin requires explicit LevelDB dependency, see below. 
akka.persistence.journal.leveldb {
    # Class name of the plugin.
    class = "akka.persistence.journal.leveldb.LeveldbJournal"
    # Dispatcher for the plugin actor.
    plugin-dispatcher = "akka.persistence.dispatchers.default-plugin-dispatcher"
    # Dispatcher for message replay.
    replay-dispatcher = "akka.persistence.dispatchers.default-replay-dispatcher"
    # Storage location of LevelDB files.
    dir = "journal"
    # Use fsync on write.
    fsync = on
    # Verify checksum on read.
    checksum = off
    # Native LevelDB (via JNI) or LevelDB Java port.
    native = on
    # Number of deleted messages per persistence id that will trigger journal compaction
    compaction-intervals {
    }
}

# Shared LevelDB journal plugin (for testing only).
# Note: this plugin requires explicit LevelDB dependency, see below. 
akka.persistence.journal.leveldb-shared {
    # Class name of the plugin.
    class = "akka.persistence.journal.leveldb.SharedLeveldbJournal"
    # Dispatcher for the plugin actor.
    plugin-dispatcher = "akka.actor.default-dispatcher"
    # Timeout for async journal operations.
    timeout = 10s
    store {
        # Dispatcher for shared store actor.
        store-dispatcher = "akka.persistence.dispatchers.default-plugin-dispatcher"
        # Dispatcher for message replay.
        replay-dispatcher = "akka.persistence.dispatchers.default-replay-dispatcher"
        # Storage location of LevelDB files.
        dir = "journal"
        # Use fsync on write.
        fsync = on
        # Verify checksum on read.
        checksum = off
        # Native LevelDB (via JNI) or LevelDB Java port.
        native = on
        # Number of deleted messages per persistence id that will trigger journal compaction
        compaction-intervals {
        }
    }
}

akka.persistence.journal.proxy {
  # Class name of the plugin.
  class = "akka.persistence.journal.PersistencePluginProxy"
  # Dispatcher for the plugin actor.
  plugin-dispatcher = "akka.actor.default-dispatcher"
  # Set this to on in the configuration of the ActorSystem
  # that will host the target journal
  start-target-journal = off
  # The journal plugin config path to use for the target journal
  target-journal-plugin = ""
  # The address of the proxy to connect to from other nodes. Optional setting.
  target-journal-address = ""
  # Initialization timeout of target lookup
  init-timeout = 10s
}

akka.persistence.snapshot-store.proxy {
  # Class name of the plugin.
  class = "akka.persistence.journal.PersistencePluginProxy"
  # Dispatcher for the plugin actor.
  plugin-dispatcher = "akka.actor.default-dispatcher"
  # Set this to on in the configuration of the ActorSystem
  # that will host the target snapshot-store
  start-target-snapshot-store = off
  # The journal plugin config path to use for the target snapshot-store
  target-snapshot-store-plugin = ""
  # The address of the proxy to connect to from other nodes. Optional setting.
  target-snapshot-store-address = ""
  # Initialization timeout of target lookup
  init-timeout = 10s
}

# LevelDB persistence requires the following dependency declarations:
#
# SBT:
#       "org.iq80.leveldb"            % "leveldb"          % "0.7"
#       "org.fusesource.leveldbjni"   % "leveldbjni-all"   % "1.8"
#
# Maven:
#        <dependency>
#            <groupId>org.iq80.leveldb</groupId>
#            <artifactId>leveldb</artifactId>
#            <version>0.7</version>
#        </dependency>
#        <dependency>
#            <groupId>org.fusesource.leveldbjni</groupId>
#            <artifactId>leveldbjni-all</artifactId>
#            <version>1.8</version>
#        </dependency>
```



### akka-persistence-query

```

#######################################################
# Akka Persistence Query Reference Configuration File #
#######################################################

# This is the reference config file that contains all the default settings.
# Make your edits in your application.conf in order to override these settings.

#//#query-leveldb
# Configuration for the LeveldbReadJournal
akka.persistence.query.journal.leveldb {
  # Implementation class of the LevelDB ReadJournalProvider
  class = "akka.persistence.query.journal.leveldb.LeveldbReadJournalProvider"
  
  # Absolute path to the write journal plugin configuration entry that this 
  # query journal will connect to. That must be a LeveldbJournal or SharedLeveldbJournal.
  # If undefined (or "") it will connect to the default journal as specified by the
  # akka.persistence.journal.plugin property.
  write-plugin = ""
  
  # The LevelDB write journal is notifying the query side as soon as things
  # are persisted, but for efficiency reasons the query side retrieves the events 
  # in batches that sometimes can be delayed up to the configured `refresh-interval`.
  refresh-interval = 3s
  
  # How many events to fetch in one query (replay) and keep buffered until they
  # are delivered downstreams.
  max-buffer-size = 100
}
#//#query-leveldb
```



### akka-persistence-testkit

```

##################################################
# Akka Persistence Testkit Reference Config File #
##################################################

# This is the reference config file that contains all the default settings.
# Make your edits/overrides in your application.conf.

akka.persistence.testkit {

  # configuration for persistence testkit for events
  events {
    # enable serialization of the persisted events
    serialize = true
    # timeout for assertions
    assert-timeout = 3s
    # poll interval for assertions with timeout
    assert-poll-interval = 100millis
  }

  # configuration for persistence testkit for snapshots
  snapshots {
    # enable serialization of the persisted snapshots
    serialize = true
    # timeout for assertions
    assert-timeout = 3s
    # poll interval for assertions with timeout
    assert-poll-interval = 100millis
  }

}

akka.persistence.testkit.query {
  class = "akka.persistence.testkit.query.PersistenceTestKitReadJournalProvider"
}
```



### akka-remote artery

```

#####################################
# Akka Remote Reference Config File #
#####################################

# This is the reference config file that contains all the default settings.
# Make your edits/overrides in your application.conf.

# comments about akka.actor settings left out where they are already in akka-
# actor.jar, because otherwise they would be repeated in config rendering.
#
# For the configuration of the new remoting implementation (Artery) please look
# at the bottom section of this file as it is listed separately.

akka {

  actor {

    serializers {
      akka-containers = "akka.remote.serialization.MessageContainerSerializer"
      akka-misc = "akka.remote.serialization.MiscMessageSerializer"
      artery = "akka.remote.serialization.ArteryMessageSerializer"
      proto = "akka.remote.serialization.ProtobufSerializer"
      daemon-create = "akka.remote.serialization.DaemonMsgCreateSerializer"
      akka-system-msg = "akka.remote.serialization.SystemMessageSerializer"
    }

    serialization-bindings {
      "akka.actor.ActorSelectionMessage" = akka-containers

      "akka.remote.DaemonMsgCreate" = daemon-create

      "akka.remote.artery.ArteryMessage" = artery

      # Since akka.protobuf.Message does not extend Serializable but
      # GeneratedMessage does, need to use the more specific one here in order
      # to avoid ambiguity.
      # This is only loaded if akka-protobuf is on the classpath
      # It should not be used and users should migrate to using the protobuf classes
      # directly
      # Remove in 2.7
      "akka.protobuf.GeneratedMessage" = proto

      "akka.protobufv3.internal.GeneratedMessageV3" = proto

      # Since com.google.protobuf.Message does not extend Serializable but
      # GeneratedMessage does, need to use the more specific one here in order
      # to avoid ambiguity.
      # This com.google.protobuf serialization binding is only used if the class can be loaded,
      # i.e. com.google.protobuf dependency has been added in the application project.
      "com.google.protobuf.GeneratedMessage" = proto
      "com.google.protobuf.GeneratedMessageV3" = proto

      "akka.actor.Identify" = akka-misc
      "akka.actor.ActorIdentity" = akka-misc
      "scala.Some" = akka-misc
      "scala.None$" = akka-misc
      "java.util.Optional" = akka-misc
      "akka.actor.Status$Success" = akka-misc
      "akka.actor.Status$Failure" = akka-misc
      "akka.actor.ActorRef" = akka-misc
      "akka.actor.PoisonPill$" = akka-misc
      "akka.actor.Kill$" = akka-misc
      "akka.remote.RemoteWatcher$Heartbeat$" = akka-misc
      "akka.remote.RemoteWatcher$HeartbeatRsp" = akka-misc
      "akka.Done" = akka-misc
      "akka.NotUsed" = akka-misc
      "akka.actor.Address" = akka-misc
      "akka.remote.UniqueAddress" = akka-misc

      "akka.actor.ActorInitializationException" = akka-misc
      "akka.actor.IllegalActorStateException" = akka-misc
      "akka.actor.ActorKilledException" = akka-misc
      "akka.actor.InvalidActorNameException" = akka-misc
      "akka.actor.InvalidMessageException" = akka-misc
      "java.util.concurrent.TimeoutException" = akka-misc
      "akka.remote.serialization.ThrowableNotSerializableException" = akka-misc

      "akka.actor.LocalScope$" = akka-misc
      "akka.remote.RemoteScope" = akka-misc

      "com.typesafe.config.impl.SimpleConfig" = akka-misc
      "com.typesafe.config.Config" = akka-misc

      "akka.routing.FromConfig" = akka-misc
      "akka.routing.DefaultResizer" = akka-misc
      "akka.routing.BalancingPool" = akka-misc
      "akka.routing.BroadcastGroup" = akka-misc
      "akka.routing.BroadcastPool" = akka-misc
      "akka.routing.RandomGroup" = akka-misc
      "akka.routing.RandomPool" = akka-misc
      "akka.routing.RoundRobinGroup" = akka-misc
      "akka.routing.RoundRobinPool" = akka-misc
      "akka.routing.ScatterGatherFirstCompletedGroup" = akka-misc
      "akka.routing.ScatterGatherFirstCompletedPool" = akka-misc
      "akka.routing.SmallestMailboxPool" = akka-misc
      "akka.routing.TailChoppingGroup" = akka-misc
      "akka.routing.TailChoppingPool" = akka-misc
      "akka.remote.routing.RemoteRouterConfig" = akka-misc

      "akka.pattern.StatusReply" = akka-misc

      "akka.dispatch.sysmsg.SystemMessage" = akka-system-msg

      # Java Serializer is by default used for exceptions and will by default
      # not be allowed to be serialized, but in certain cases they are replaced
      # by `akka.remote.serialization.ThrowableNotSerializableException` if
      # no specific serializer has been defined:
      # - when wrapped in `akka.actor.Status.Failure` for ask replies
      # - when wrapped in system messages for exceptions from remote deployed child actors
      #
      # It's recommended that you implement custom serializer for exceptions that are
      # sent remotely, You can add binding to akka-misc (MiscMessageSerializer) for the
      # exceptions that have a constructor with single message String or constructor with
      # message String as first parameter and cause Throwable as second parameter. Note that it's not
      # safe to add this binding for general exceptions such as IllegalArgumentException
      # because it may have a subclass without required constructor.
      "java.lang.Throwable" = java
    }

    serialization-identifiers {
      "akka.remote.serialization.ProtobufSerializer" = 2
      "akka.remote.serialization.DaemonMsgCreateSerializer" = 3
      "akka.remote.serialization.MessageContainerSerializer" = 6
      "akka.remote.serialization.MiscMessageSerializer" = 16
      "akka.remote.serialization.ArteryMessageSerializer" = 17

      "akka.remote.serialization.SystemMessageSerializer" = 22

      # deprecated in 2.6.0, moved to akka-actor
      "akka.remote.serialization.LongSerializer" = 18
      # deprecated in 2.6.0, moved to akka-actor
      "akka.remote.serialization.IntSerializer" = 19
      # deprecated in 2.6.0, moved to akka-actor
      "akka.remote.serialization.StringSerializer" = 20
      # deprecated in 2.6.0, moved to akka-actor
      "akka.remote.serialization.ByteStringSerializer" = 21
    }

    deployment {

      default {

        # if this is set to a valid remote address, the named actor will be
        # deployed at that node e.g. "akka://sys@host:port"
        remote = ""

        target {

          # A list of hostnames and ports for instantiating the children of a
          # router
          #   The format should be on "akka://sys@host:port", where:
          #    - sys is the remote actor system name
          #    - hostname can be either hostname or IP address the remote actor
          #      should connect to
          #    - port should be the port for the remote server on the other node
          # The number of actor instances to be spawned is still taken from the
          # nr-of-instances setting as for local routers; the instances will be
          # distributed round-robin among the given nodes.
          nodes = []

        }
      }
    }
  }

  remote {
    ### Settings shared by classic remoting and Artery (the new implementation of remoting)

    # Using remoting directly is typically not desirable, so a warning will
    # be shown to make this clear. Set this setting to 'off' to suppress that
    # warning.
    warn-about-direct-use = on


    # If Cluster is not used, remote watch and deployment are disabled.
    # To optionally use them while not using Cluster, set to 'on'.
    use-unsafe-remote-features-outside-cluster = off

    # A warning will be logged on remote watch attempts if Cluster
    # is not in use and 'use-unsafe-remote-features-outside-cluster'
    # is 'off'. Set this to 'off' to suppress these.
    warn-unsafe-watch-outside-cluster = on

    # Settings for the Phi accrual failure detector (http://www.jaist.ac.jp/~defago/files/pdf/IS_RR_2004_010.pdf
    # [Hayashibara et al]) used for remote death watch.
    # The default PhiAccrualFailureDetector will trigger if there are no heartbeats within
    # the duration heartbeat-interval + acceptable-heartbeat-pause + threshold_adjustment,
    # i.e. around 12.5 seconds with default settings.
    watch-failure-detector {

      # FQCN of the failure detector implementation.
      # It must implement akka.remote.FailureDetector and have
      # a public constructor with a com.typesafe.config.Config and
      # akka.actor.EventStream parameter.
      implementation-class = "akka.remote.PhiAccrualFailureDetector"

      # How often keep-alive heartbeat messages should be sent to each connection.
      heartbeat-interval = 1 s

      # Defines the failure detector threshold.
      # A low threshold is prone to generate many wrong suspicions but ensures
      # a quick detection in the event of a real crash. Conversely, a high
      # threshold generates fewer mistakes but needs more time to detect
      # actual crashes.
      threshold = 10.0

      # Number of the samples of inter-heartbeat arrival times to adaptively
      # calculate the failure timeout for connections.
      max-sample-size = 200

      # Minimum standard deviation to use for the normal distribution in
      # AccrualFailureDetector. Too low standard deviation might result in
      # too much sensitivity for sudden, but normal, deviations in heartbeat
      # inter arrival times.
      min-std-deviation = 100 ms

      # Number of potentially lost/delayed heartbeats that will be
      # accepted before considering it to be an anomaly.
      # This margin is important to be able to survive sudden, occasional,
      # pauses in heartbeat arrivals, due to for example garbage collect or
      # network drop.
      acceptable-heartbeat-pause = 10 s


      # How often to check for nodes marked as unreachable by the failure
      # detector
      unreachable-nodes-reaper-interval = 1s

      # After the heartbeat request has been sent the first failure detection
      # will start after this period, even though no heartbeat mesage has
      # been received.
      expected-response-after = 1 s

    }

    # remote deployment configuration section
    deployment {
      # deprecated, use `enable-allow-list`
      enable-whitelist = off

      # If true, will only allow specific classes listed in `allowed-actor-classes` to be instanciated on this
      # system via remote deployment
      enable-allow-list = ${akka.remote.deployment.enable-whitelist}


      # deprecated, use `allowed-actor-classes`
      whitelist = []

      allowed-actor-classes = ${akka.remote.deployment.whitelist}
    }

    ### Default dispatcher for the remoting subsystem
    default-remote-dispatcher {
      type = Dispatcher
      executor = "fork-join-executor"
      fork-join-executor {
        parallelism-min = 2
        parallelism-factor = 0.5
        parallelism-max = 16
      }
      throughput = 10
    }
akka {

  remote {

    ### Configuration for Artery, the new implementation of remoting
    artery {

      # Disable artery with this flag
      enabled = on

      # Select the underlying transport implementation.
      #
      # Possible values: aeron-udp, tcp, tls-tcp
      # See https://doc.akka.io/docs/akka/current/remoting-artery.html#selecting-a-transport for the tradeoffs
      # for each transport
      transport = tcp

      # Canonical address is the address other clients should connect to.
      # Artery transport will expect messages to this address.
      canonical {

        # The default remote server port clients should connect to.
        # Default is 25520, use 0 if you want a random available port
        # This port needs to be unique for each actor system on the same machine.
        port = 25520

        # Hostname clients should connect to. Can be set to an ip, hostname
        # or one of the following special values:
        #   "<getHostAddress>"   InetAddress.getLocalHost.getHostAddress
        #   "<getHostName>"      InetAddress.getLocalHost.getHostName
        #
        hostname = "<getHostAddress>"
      }

      # Use these settings to bind a network interface to a different address
      # than artery expects messages at. This may be used when running Akka
      # nodes in a separated networks (under NATs or in containers). If canonical
      # and bind addresses are different, then network configuration that relays
      # communications from canonical to bind addresses is expected.
      bind {

        # Port to bind a network interface to. Can be set to a port number
        # of one of the following special values:
        #   0    random available port
        #   ""   akka.remote.artery.canonical.port
        #
        port = ""

        # Hostname to bind a network interface to. Can be set to an ip, hostname
        # or one of the following special values:
        #   "0.0.0.0"            all interfaces
        #   ""                   akka.remote.artery.canonical.hostname
        #   "<getHostAddress>"   InetAddress.getLocalHost.getHostAddress
        #   "<getHostName>"      InetAddress.getLocalHost.getHostName
        #
        hostname = ""

        # Time to wait for Aeron/TCP to bind
        bind-timeout = 3s
      }


      # Actor paths to use the large message stream for when a message
      # is sent to them over remoting. The large message stream dedicated
      # is separate from "normal" and system messages so that sending a
      # large message does not interfere with them.
      # Entries should be the full path to the actor. Wildcards in the form of "*"
      # can be supplied at any place and matches any name at that segment -
      # "/user/supervisor/actor/*" will match any direct child to actor,
      # while "/supervisor/*/child" will match any grandchild to "supervisor" that
      # has the name "child"
      # Entries have to be specified on both the sending and receiving side.
      # Messages sent to ActorSelections will not be passed through the large message
      # stream, to pass such messages through the large message stream the selections
      # but must be resolved to ActorRefs first.
      large-message-destinations = []

      # Enable untrusted mode, which discards inbound system messages, PossiblyHarmful and
      # ActorSelection messages. E.g. remote watch and remote deployment will not work.
      # ActorSelection messages can be enabled for specific paths with the trusted-selection-paths
      untrusted-mode = off

      # When 'untrusted-mode=on' inbound actor selections are by default discarded.
      # Actors with paths defined in this list are granted permission to receive actor
      # selections messages.
      # E.g. trusted-selection-paths = ["/user/receptionist", "/user/namingService"]
      trusted-selection-paths = []

      # If this is "on", all inbound remote messages will be logged at DEBUG level,
      # if off then they are not logged
      log-received-messages = off

      # If this is "on", all outbound remote messages will be logged at DEBUG level,
      # if off then they are not logged
      log-sent-messages = off

      # Logging of message types with payload size in bytes larger than
      # this value. Maximum detected size per message type is logged once,
      # with an increase threshold of 10%.
      # By default this feature is turned off. Activate it by setting the property to
      # a value in bytes, such as 1000b. Note that for all messages larger than this
      # limit there will be extra performance and scalability cost.
      log-frame-size-exceeding = off

      advanced {

        # Maximum serialized message size, including header data.
        maximum-frame-size = 256 KiB

        # Direct byte buffers are reused in a pool with this maximum size.
        # Each buffer has the size of 'maximum-frame-size'.
        # This is not a hard upper limit on number of created buffers. Additional
        # buffers will be created if needed, e.g. when using many outbound
        # associations at the same time. Such additional buffers will be garbage
        # collected, which is not as efficient as reusing buffers in the pool.
        buffer-pool-size = 128

        # Maximum serialized message size for the large messages, including header data.
        # It is currently restricted to 1/8th the size of a term buffer that can be
        # configured by setting the 'aeron.term.buffer.length' system property.
        # See 'large-message-destinations'.
        maximum-large-frame-size = 2 MiB

        # Direct byte buffers for the large messages are reused in a pool with this maximum size.
        # Each buffer has the size of 'maximum-large-frame-size'.
        # See 'large-message-destinations'.
        # This is not a hard upper limit on number of created buffers. Additional
        # buffers will be created if needed, e.g. when using many outbound
        # associations at the same time. Such additional buffers will be garbage
        # collected, which is not as efficient as reusing buffers in the pool.
        large-buffer-pool-size = 32

        # For enabling testing features, such as blackhole in akka-remote-testkit.
        test-mode = off

        # Settings for the materializer that is used for the remote streams.
        materializer = ${akka.stream.materializer}

        # Remoting will use the given dispatcher for the ordinary and large message
        # streams.
        use-dispatcher = "akka.remote.default-remote-dispatcher"

        # Remoting will use the given dispatcher for the control stream.
        # It can be good to not use the same dispatcher for the control stream as
        # the dispatcher for the ordinary message stream so that heartbeat messages
        # are not disturbed.
        use-control-stream-dispatcher = "akka.actor.internal-dispatcher"


        # Total number of inbound lanes, shared among all inbound associations. A value
        # greater than 1 means that deserialization can be performed in parallel for
        # different destination actors. The selection of lane is based on consistent
        # hashing of the recipient ActorRef to preserve message ordering per receiver.
        # Lowest latency can be achieved with inbound-lanes=1 because of one less
        # asynchronous boundary.
        inbound-lanes = 4

        # Number of outbound lanes for each outbound association. A value greater than 1
        # means that serialization and other work can be performed in parallel for different
        # destination actors. The selection of lane is based on consistent hashing of the
        # recipient ActorRef to preserve message ordering per receiver. Note that messages
        # for different destination systems (hosts) are handled by different streams also
        # when outbound-lanes=1. Lowest latency can be achieved with outbound-lanes=1
        # because of one less asynchronous boundary.
        outbound-lanes = 1

        # Size of the send queue for outgoing messages. Messages will be dropped if
        # the queue becomes full. This may happen if you send a burst of many messages
        # without end-to-end flow control. Note that there is one such queue per
        # outbound association. The trade-off of using a larger queue size is that
        # it consumes more memory, since the queue is based on preallocated array with
        # fixed size.
        outbound-message-queue-size = 3072

        # Size of the send queue for outgoing control messages, such as system messages.
        # If this limit is reached the remote system is declared to be dead and its UID
        # marked as quarantined. Note that there is one such queue per outbound association.
        # It is a linked queue so it will not use more memory than needed but by increasing
        # too much you may risk OutOfMemoryError in the worst case.
        outbound-control-queue-size = 20000

        # Size of the send queue for outgoing large messages. Messages will be dropped if
        # the queue becomes full. This may happen if you send a burst of many messages
        # without end-to-end flow control. Note that there is one such queue per
        # outbound association.
        # It is a linked queue so it will not use more memory than needed but by increasing
        # too much you may risk OutOfMemoryError, especially since the message payload
        # of these messages may be large.
        outbound-large-message-queue-size = 256

        # This setting defines the maximum number of unacknowledged system messages
        # allowed for a remote system. If this limit is reached the remote system is
        # declared to be dead and its UID marked as quarantined.
        system-message-buffer-size = 20000

        # unacknowledged system messages are re-delivered with this interval
        system-message-resend-interval = 1 second



        # The timeout for outbound associations to perform the initial handshake.
        # This timeout must be greater than the 'image-liveness-timeout' when
        # transport is aeron-udp.
        handshake-timeout = 20 seconds

        # incomplete initial handshake attempt is retried with this interval
        handshake-retry-interval = 1 second

        # Handshake requests are performed periodically with this interval,
        # also after the handshake has been completed to be able to establish
        # a new session with a restarted destination system.
        inject-handshake-interval = 1 second


        # System messages that are not acknowledged after re-sending for this period are
        # dropped and will trigger quarantine. The value should be longer than the length
        # of a network partition that you need to survive.
        give-up-system-message-after = 6 hours

        # Outbound streams are stopped when they haven't been used for this duration.
        # They are started again when new messages are sent.
        stop-idle-outbound-after = 5 minutes

        # Outbound streams are quarantined when they haven't been used for this duration
        # to cleanup resources used by the association, such as compression tables.
        # This will cleanup association to crashed systems that didn't announce their
        # termination.
        # The value should be longer than the length of a network partition that you
        # need to survive.
        # The value must also be greater than stop-idle-outbound-after.
        # Once every 1/10 of this duration an extra handshake message will be sent.
        # Therfore it's also recommended to use a value that is greater than 10 times
        # the stop-idle-outbound-after, since otherwise the idle streams will not be
        # stopped.
        quarantine-idle-outbound-after = 6 hours

        # Stop outbound stream of a quarantined association after this idle timeout, i.e.
        # when not used any more.
        stop-quarantined-after-idle = 3 seconds

        # After catastrophic communication failures that could result in the loss of system
        # messages or after the remote DeathWatch triggers the remote system gets
        # quarantined to prevent inconsistent behavior.
        # This setting controls how long the quarantined association will be kept around
        # before being removed to avoid long-term memory leaks. It must be quarantined
        # and also unused for this duration before it's removed. When removed the historical
        # information about which UIDs that were quarantined for that hostname:port is
        # gone which could result in communication with a previously quarantined node
        # if it wakes up again. Therfore this shouldn't be set too low.
        remove-quarantined-association-after = 1 h

        # during ActorSystem termination the remoting will wait this long for
        # an acknowledgment by the destination system that flushing of outstanding
        # remote messages has been completed
        shutdown-flush-timeout = 1 second

        # Before sending notificaiton of terminated actor (DeathWatchNotification) other messages
        # will be flushed to make sure that the Terminated message arrives after other messages.
        # It will wait this long for the flush acknowledgement before continuing.
        # The flushing can be disabled by setting this to `off`.
        death-watch-notification-flush-timeout = 3 seconds

        # See 'inbound-max-restarts'
        inbound-restart-timeout = 5 seconds

        # Max number of restarts within 'inbound-restart-timeout' for the inbound streams.
        # If more restarts occurs the ActorSystem will be terminated.
        inbound-max-restarts = 5

        # Retry outbound connection after this backoff.
        # Only used when transport is tcp or tls-tcp.
        outbound-restart-backoff = 1 second

        # See 'outbound-max-restarts'
        outbound-restart-timeout = 5 seconds

        # Max number of restarts within 'outbound-restart-timeout' for the outbound streams.
        # If more restarts occurs the ActorSystem will be terminated.
        outbound-max-restarts = 5

        # compression of common strings in remoting messages, like actor destinations, serializers etc
        compression {

          actor-refs {
            # Max number of compressed actor-refs
            # Note that compression tables are "rolling" (i.e. a new table replaces the old
            # compression table once in a while), and this setting is only about the total number
            # of compressions within a single such table.
            # Must be a positive natural number. Can be disabled with "off".
            max = 256

            # interval between new table compression advertisements.
            # this means the time during which we collect heavy-hitter data and then turn it into a compression table.
            advertisement-interval = 1 minute
          }
          manifests {
            # Max number of compressed manifests
            # Note that compression tables are "rolling" (i.e. a new table replaces the old
            # compression table once in a while), and this setting is only about the total number
            # of compressions within a single such table.
            # Must be a positive natural number. Can be disabled with "off".
            max = 256

            # interval between new table compression advertisements.
            # this means the time during which we collect heavy-hitter data and then turn it into a compression table.
            advertisement-interval = 1 minute
          }
        }

        # List of fully qualified class names of remote instruments which should
        # be initialized and used for monitoring of remote messages.
        # The class must extend akka.remote.artery.RemoteInstrument and
        # have a public constructor with empty parameters or one ExtendedActorSystem
        # parameter.
        # A new instance of RemoteInstrument will be created for each encoder and decoder.
        # It's only called from the stage, so if it dosn't delegate to any shared instance
        # it doesn't have to be thread-safe.
        # Refer to `akka.remote.artery.RemoteInstrument` for more information.
        instruments = ${?akka.remote.artery.advanced.instruments} []

        # Only used when transport is aeron-udp
        aeron {
          # Only used when transport is aeron-udp.
          log-aeron-counters = false

          # Controls whether to start the Aeron media driver in the same JVM or use external
          # process. Set to 'off' when using external media driver, and then also set the
          # 'aeron-dir'.
          # Only used when transport is aeron-udp.
          embedded-media-driver = on

          # Directory used by the Aeron media driver. It's mandatory to define the 'aeron-dir'
          # if using external media driver, i.e. when 'embedded-media-driver = off'.
          # Embedded media driver will use a this directory, or a temporary directory if this
          # property is not defined (empty).
          # Only used when transport is aeron-udp.
          aeron-dir = ""

          # Whether to delete aeron embedded driver directory upon driver stop.
          # Only used when transport is aeron-udp.
          delete-aeron-dir = yes

          # Level of CPU time used, on a scale between 1 and 10, during backoff/idle.
          # The tradeoff is that to have low latency more CPU time must be used to be
          # able to react quickly on incoming messages or send as fast as possible after
          # backoff backpressure.
          # Level 1 strongly prefer low CPU consumption over low latency.
          # Level 10 strongly prefer low latency over low CPU consumption.
          # Only used when transport is aeron-udp.
          idle-cpu-level = 5

          # messages that are not accepted by Aeron are dropped after retrying for this period
          # Only used when transport is aeron-udp.
          give-up-message-after = 60 seconds

          # Timeout after which aeron driver has not had keepalive messages
          # from a client before it considers the client dead.
          # Only used when transport is aeron-udp.
          client-liveness-timeout = 20 seconds

          # Timout after after which an uncommitted publication will be unblocked
          # Only used when transport is aeron-udp.
          publication-unblock-timeout = 40 seconds

          # Timeout for each the INACTIVE and LINGER stages an aeron image
          # will be retained for when it is no longer referenced.
          # This timeout must be less than the 'handshake-timeout'.
          # Only used when transport is aeron-udp.
          image-liveness-timeout = 10 seconds

          # Timeout after which the aeron driver is considered dead
          # if it does not update its C'n'C timestamp.
          # Only used when transport is aeron-udp.
          driver-timeout = 20 seconds
        }

        # Only used when transport is tcp or tls-tcp.
        tcp {
          # Timeout of establishing outbound connections.
          connection-timeout = 5 seconds

          # The local address that is used for the client side of the TCP connection.
          outbound-client-hostname = ""
        }

      }

      # SSL configuration that is used when transport=tls-tcp.
      ssl {
        # Factory of SSLEngine.
        # Must implement akka.remote.artery.tcp.SSLEngineProvider and have a public
        # constructor with an ActorSystem parameter.
        # The default ConfigSSLEngineProvider is configured by properties in section
        # akka.remote.artery.ssl.config-ssl-engine
        ssl-engine-provider = akka.remote.artery.tcp.ConfigSSLEngineProvider

        # Config of akka.remote.artery.tcp.ConfigSSLEngineProvider
        config-ssl-engine {

          # This is the Java Key Store used by the server connection
          key-store = "keystore"

          # This password is used for decrypting the key store
          # Use substitution from environment variables for passwords. Don't define
          # real passwords in config files. key-store-password=${SSL_KEY_STORE_PASSWORD}
          key-store-password = "changeme"

          # This password is used for decrypting the key
          # Use substitution from environment variables for passwords. Don't define
          # real passwords in config files. key-password=${SSL_KEY_PASSWORD}
          key-password = "changeme"

          # This is the Java Key Store used by the client connection
          trust-store = "truststore"

          # This password is used for decrypting the trust store
          # Use substitution from environment variables for passwords. Don't define
          # real passwords in config files. trust-store-password=${SSL_TRUST_STORE_PASSWORD}
          trust-store-password = "changeme"

          # Protocol to use for SSL encryption.
          protocol = "TLSv1.2"

          # Example: ["TLS_DHE_RSA_WITH_AES_128_GCM_SHA256", 
          #   "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256",
          #   "TLS_DHE_RSA_WITH_AES_256_GCM_SHA384",
          #   "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384"]
          # When doing rolling upgrades, make sure to include both the algorithm used 
          # by old nodes and the preferred algorithm.
          # If you use a JDK 8 prior to 8u161 you need to install
          # the JCE Unlimited Strength Jurisdiction Policy Files to use AES 256.
          # More info here:
          # https://www.oracle.com/java/technologies/javase-jce-all-downloads.html
          enabled-algorithms = ["TLS_DHE_RSA_WITH_AES_256_GCM_SHA384",
            "TLS_RSA_WITH_AES_128_CBC_SHA"]

          # There are two options, and the default SecureRandom is recommended:
          # "" or "SecureRandom" => (default)
          # "SHA1PRNG" => Can be slow because of blocking issues on Linux
          #
          # Setting a value here may require you to supply the appropriate cipher
          # suite (see enabled-algorithms section above)
          random-number-generator = ""

          # Require mutual authentication between TLS peers
          #
          # Without mutual authentication only the peer that actively establishes a connection (TLS client side)
          # checks if the passive side (TLS server side) sends over a trusted certificate. With the flag turned on,
          # the passive side will also request and verify a certificate from the connecting peer.
          #
          # To prevent man-in-the-middle attacks this setting is enabled by default.
          require-mutual-authentication = on

          # Set this to `on` to verify hostnames with sun.security.util.HostnameChecker
          # If possible it is recommended to have this enabled. Hostname verification is designed for
          # situations where things locate each other by hostname, in scenarios where host names are dynamic
          # and not known up front it can make sense to have this disabled.
          hostname-verification = off
        }

        # Config of akka.remote.artery.tcp.ssl.RotatingKeysSSLEngineProvider
        # This engine provider reads PEM files from a mount point shared with the secret
        # manager. The constructed SSLContext is cached some time (configurable) so when
        # the credentials rotate the new credentials are eventually picked up.
        # By default mTLS is enabled.
        # This provider also includes a verification phase that runs after the TLS handshake
        # phase. In this verification, both peers run an authorization and verify they are
        # part of the same akka cluster. The verification happens via comparing the subject
        # names in the peer's certificate with the name on the own certificate so if you
        # use this SSLEngineProvider you should make sure all nodes on the cluster include
        # at least one common subject name (CN or SAN).
        # The Key setup this implementation supports has some limitations:
        #   1. the private key must be provided on a PKCS#1 or a non-encrypted PKCS#8 PEM-formatted file
        #   2. the private key must be be of an algorythm supported by `akka-pki` tools (e.g. "RSA", not "EC")
        #   3. the node certificate must be issued by a root CA (not an intermediate CA)
        #   4. both the node and the CA certificates must be provided in PEM-formatted files
        rotating-keys-engine {

          # This is a convention that people may follow if they wish to save themselves some configuration
          secret-mount-point = /var/run/secrets/akka-tls/rotating-keys-engine

          # The absolute path the PEM file with the private key.
          key-file = ${akka.remote.artery.ssl.rotating-keys-engine.secret-mount-point}/tls.key
          # The absolute path to the PEM file of the certificate for the private key above.
          cert-file = ${akka.remote.artery.ssl.rotating-keys-engine.secret-mount-point}/tls.crt
          # The absolute path to the PEM file of the certificate of the CA that emited
          # the node certificate above.
          ca-cert-file = ${akka.remote.artery.ssl.rotating-keys-engine.secret-mount-point}/ca.crt

          # There are two options, and the default SecureRandom is recommended:
          # "" or "SecureRandom" => (default)
          # "SHA1PRNG" => Can be slow because of blocking issues on Linux
          #
          # Setting a value here may require you to supply the appropriate cipher
          # suite (see enabled-algorithms section)
          random-number-generator = ""

          # Example: ["TLS_DHE_RSA_WITH_AES_128_GCM_SHA256",
          #   "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256",
          #   "TLS_DHE_RSA_WITH_AES_256_GCM_SHA384",
          #   "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384"]
          # If you use a JDK 8 prior to 8u161 you need to install
          # the JCE Unlimited Strength Jurisdiction Policy Files to use AES 256.
          # More info here:
          # https://www.oracle.com/java/technologies/javase-jce-all-downloads.html
          enabled-algorithms = ["TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384"]

          # Protocol to use for SSL encryption.
          protocol = "TLSv1.2"

          # How long should an SSLContext instance be cached. When rotating keys and certificates,
          # there must a time overlap between the old certificate/key and the new ones. The
          # value of this setting should be lower than duration of that overlap.
          ssl-context-cache-ttl = 5m
        }
      }
    }
  }

}
```



### akka-remote classic（已弃用）

```

#####################################
# Akka Remote Reference Config File #
#####################################

# This is the reference config file that contains all the default settings.
# Make your edits/overrides in your application.conf.

# comments about akka.actor settings left out where they are already in akka-
# actor.jar, because otherwise they would be repeated in config rendering.
#
# For the configuration of the new remoting implementation (Artery) please look
# at the bottom section of this file as it is listed separately.

akka {

  actor {

    serializers {
      akka-containers = "akka.remote.serialization.MessageContainerSerializer"
      akka-misc = "akka.remote.serialization.MiscMessageSerializer"
      artery = "akka.remote.serialization.ArteryMessageSerializer"
      proto = "akka.remote.serialization.ProtobufSerializer"
      daemon-create = "akka.remote.serialization.DaemonMsgCreateSerializer"
      akka-system-msg = "akka.remote.serialization.SystemMessageSerializer"
    }

    serialization-bindings {
      "akka.actor.ActorSelectionMessage" = akka-containers

      "akka.remote.DaemonMsgCreate" = daemon-create

      "akka.remote.artery.ArteryMessage" = artery

      # Since akka.protobuf.Message does not extend Serializable but
      # GeneratedMessage does, need to use the more specific one here in order
      # to avoid ambiguity.
      # This is only loaded if akka-protobuf is on the classpath
      # It should not be used and users should migrate to using the protobuf classes
      # directly
      # Remove in 2.7
      "akka.protobuf.GeneratedMessage" = proto

      "akka.protobufv3.internal.GeneratedMessageV3" = proto

      # Since com.google.protobuf.Message does not extend Serializable but
      # GeneratedMessage does, need to use the more specific one here in order
      # to avoid ambiguity.
      # This com.google.protobuf serialization binding is only used if the class can be loaded,
      # i.e. com.google.protobuf dependency has been added in the application project.
      "com.google.protobuf.GeneratedMessage" = proto
      "com.google.protobuf.GeneratedMessageV3" = proto

      "akka.actor.Identify" = akka-misc
      "akka.actor.ActorIdentity" = akka-misc
      "scala.Some" = akka-misc
      "scala.None$" = akka-misc
      "java.util.Optional" = akka-misc
      "akka.actor.Status$Success" = akka-misc
      "akka.actor.Status$Failure" = akka-misc
      "akka.actor.ActorRef" = akka-misc
      "akka.actor.PoisonPill$" = akka-misc
      "akka.actor.Kill$" = akka-misc
      "akka.remote.RemoteWatcher$Heartbeat$" = akka-misc
      "akka.remote.RemoteWatcher$HeartbeatRsp" = akka-misc
      "akka.Done" = akka-misc
      "akka.NotUsed" = akka-misc
      "akka.actor.Address" = akka-misc
      "akka.remote.UniqueAddress" = akka-misc

      "akka.actor.ActorInitializationException" = akka-misc
      "akka.actor.IllegalActorStateException" = akka-misc
      "akka.actor.ActorKilledException" = akka-misc
      "akka.actor.InvalidActorNameException" = akka-misc
      "akka.actor.InvalidMessageException" = akka-misc
      "java.util.concurrent.TimeoutException" = akka-misc
      "akka.remote.serialization.ThrowableNotSerializableException" = akka-misc

      "akka.actor.LocalScope$" = akka-misc
      "akka.remote.RemoteScope" = akka-misc

      "com.typesafe.config.impl.SimpleConfig" = akka-misc
      "com.typesafe.config.Config" = akka-misc

      "akka.routing.FromConfig" = akka-misc
      "akka.routing.DefaultResizer" = akka-misc
      "akka.routing.BalancingPool" = akka-misc
      "akka.routing.BroadcastGroup" = akka-misc
      "akka.routing.BroadcastPool" = akka-misc
      "akka.routing.RandomGroup" = akka-misc
      "akka.routing.RandomPool" = akka-misc
      "akka.routing.RoundRobinGroup" = akka-misc
      "akka.routing.RoundRobinPool" = akka-misc
      "akka.routing.ScatterGatherFirstCompletedGroup" = akka-misc
      "akka.routing.ScatterGatherFirstCompletedPool" = akka-misc
      "akka.routing.SmallestMailboxPool" = akka-misc
      "akka.routing.TailChoppingGroup" = akka-misc
      "akka.routing.TailChoppingPool" = akka-misc
      "akka.remote.routing.RemoteRouterConfig" = akka-misc

      "akka.pattern.StatusReply" = akka-misc

      "akka.dispatch.sysmsg.SystemMessage" = akka-system-msg

      # Java Serializer is by default used for exceptions and will by default
      # not be allowed to be serialized, but in certain cases they are replaced
      # by `akka.remote.serialization.ThrowableNotSerializableException` if
      # no specific serializer has been defined:
      # - when wrapped in `akka.actor.Status.Failure` for ask replies
      # - when wrapped in system messages for exceptions from remote deployed child actors
      #
      # It's recommended that you implement custom serializer for exceptions that are
      # sent remotely, You can add binding to akka-misc (MiscMessageSerializer) for the
      # exceptions that have a constructor with single message String or constructor with
      # message String as first parameter and cause Throwable as second parameter. Note that it's not
      # safe to add this binding for general exceptions such as IllegalArgumentException
      # because it may have a subclass without required constructor.
      "java.lang.Throwable" = java
    }

    serialization-identifiers {
      "akka.remote.serialization.ProtobufSerializer" = 2
      "akka.remote.serialization.DaemonMsgCreateSerializer" = 3
      "akka.remote.serialization.MessageContainerSerializer" = 6
      "akka.remote.serialization.MiscMessageSerializer" = 16
      "akka.remote.serialization.ArteryMessageSerializer" = 17

      "akka.remote.serialization.SystemMessageSerializer" = 22

      # deprecated in 2.6.0, moved to akka-actor
      "akka.remote.serialization.LongSerializer" = 18
      # deprecated in 2.6.0, moved to akka-actor
      "akka.remote.serialization.IntSerializer" = 19
      # deprecated in 2.6.0, moved to akka-actor
      "akka.remote.serialization.StringSerializer" = 20
      # deprecated in 2.6.0, moved to akka-actor
      "akka.remote.serialization.ByteStringSerializer" = 21
    }

    deployment {

      default {

        # if this is set to a valid remote address, the named actor will be
        # deployed at that node e.g. "akka://sys@host:port"
        remote = ""

        target {

          # A list of hostnames and ports for instantiating the children of a
          # router
          #   The format should be on "akka://sys@host:port", where:
          #    - sys is the remote actor system name
          #    - hostname can be either hostname or IP address the remote actor
          #      should connect to
          #    - port should be the port for the remote server on the other node
          # The number of actor instances to be spawned is still taken from the
          # nr-of-instances setting as for local routers; the instances will be
          # distributed round-robin among the given nodes.
          nodes = []

        }
      }
    }
  }

  remote {
    ### Settings shared by classic remoting and Artery (the new implementation of remoting)

    # Using remoting directly is typically not desirable, so a warning will
    # be shown to make this clear. Set this setting to 'off' to suppress that
    # warning.
    warn-about-direct-use = on


    # If Cluster is not used, remote watch and deployment are disabled.
    # To optionally use them while not using Cluster, set to 'on'.
    use-unsafe-remote-features-outside-cluster = off

    # A warning will be logged on remote watch attempts if Cluster
    # is not in use and 'use-unsafe-remote-features-outside-cluster'
    # is 'off'. Set this to 'off' to suppress these.
    warn-unsafe-watch-outside-cluster = on

    # Settings for the Phi accrual failure detector (http://www.jaist.ac.jp/~defago/files/pdf/IS_RR_2004_010.pdf
    # [Hayashibara et al]) used for remote death watch.
    # The default PhiAccrualFailureDetector will trigger if there are no heartbeats within
    # the duration heartbeat-interval + acceptable-heartbeat-pause + threshold_adjustment,
    # i.e. around 12.5 seconds with default settings.
    watch-failure-detector {

      # FQCN of the failure detector implementation.
      # It must implement akka.remote.FailureDetector and have
      # a public constructor with a com.typesafe.config.Config and
      # akka.actor.EventStream parameter.
      implementation-class = "akka.remote.PhiAccrualFailureDetector"

      # How often keep-alive heartbeat messages should be sent to each connection.
      heartbeat-interval = 1 s

      # Defines the failure detector threshold.
      # A low threshold is prone to generate many wrong suspicions but ensures
      # a quick detection in the event of a real crash. Conversely, a high
      # threshold generates fewer mistakes but needs more time to detect
      # actual crashes.
      threshold = 10.0

      # Number of the samples of inter-heartbeat arrival times to adaptively
      # calculate the failure timeout for connections.
      max-sample-size = 200

      # Minimum standard deviation to use for the normal distribution in
      # AccrualFailureDetector. Too low standard deviation might result in
      # too much sensitivity for sudden, but normal, deviations in heartbeat
      # inter arrival times.
      min-std-deviation = 100 ms

      # Number of potentially lost/delayed heartbeats that will be
      # accepted before considering it to be an anomaly.
      # This margin is important to be able to survive sudden, occasional,
      # pauses in heartbeat arrivals, due to for example garbage collect or
      # network drop.
      acceptable-heartbeat-pause = 10 s


      # How often to check for nodes marked as unreachable by the failure
      # detector
      unreachable-nodes-reaper-interval = 1s

      # After the heartbeat request has been sent the first failure detection
      # will start after this period, even though no heartbeat mesage has
      # been received.
      expected-response-after = 1 s

    }

    # remote deployment configuration section
    deployment {
      # deprecated, use `enable-allow-list`
      enable-whitelist = off

      # If true, will only allow specific classes listed in `allowed-actor-classes` to be instanciated on this
      # system via remote deployment
      enable-allow-list = ${akka.remote.deployment.enable-whitelist}


      # deprecated, use `allowed-actor-classes`
      whitelist = []

      allowed-actor-classes = ${akka.remote.deployment.whitelist}
    }

    ### Default dispatcher for the remoting subsystem
    default-remote-dispatcher {
      type = Dispatcher
      executor = "fork-join-executor"
      fork-join-executor {
        parallelism-min = 2
        parallelism-factor = 0.5
        parallelism-max = 16
      }
      throughput = 10
    }

    ### Configuration for classic remoting. Classic remoting is deprecated, use artery.


    # If set to a nonempty string remoting will use the given dispatcher for
    # its internal actors otherwise the default dispatcher is used. Please note
    # that since remoting can load arbitrary 3rd party drivers (see
    # "enabled-transport" and "adapters" entries) it is not guaranteed that
    # every module will respect this setting.
    use-dispatcher = "akka.remote.default-remote-dispatcher"

    # Settings for the failure detector to monitor connections.
    # For TCP it is not important to have fast failure detection, since
    # most connection failures are captured by TCP itself.
    # The default DeadlineFailureDetector will trigger if there are no heartbeats within
    # the duration heartbeat-interval + acceptable-heartbeat-pause, i.e. 124 seconds
    # with the default settings.
    transport-failure-detector {

      # FQCN of the failure detector implementation.
      # It must implement akka.remote.FailureDetector and have
      # a public constructor with a com.typesafe.config.Config and
      # akka.actor.EventStream parameter.
      implementation-class = "akka.remote.DeadlineFailureDetector"

      # How often keep-alive heartbeat messages should be sent to each connection.
      heartbeat-interval = 4 s

      # Number of potentially lost/delayed heartbeats that will be
      # accepted before considering it to be an anomaly.
      # A margin to the `heartbeat-interval` is important to be able to survive sudden,
      # occasional, pauses in heartbeat arrivals, due to for example garbage collect or
      # network drop.
      acceptable-heartbeat-pause = 120 s
    }


    # Timeout after which the startup of the remoting subsystem is considered
    # to be failed. Increase this value if your transport drivers (see the
    # enabled-transports section) need longer time to be loaded.
    startup-timeout = 10 s

    # Timout after which the graceful shutdown of the remoting subsystem is
    # considered to be failed. After the timeout the remoting system is
    # forcefully shut down. Increase this value if your transport drivers
    # (see the enabled-transports section) need longer time to stop properly.
    shutdown-timeout = 10 s

    # Before shutting down the drivers, the remoting subsystem attempts to flush
    # all pending writes. This setting controls the maximum time the remoting is
    # willing to wait before moving on to shut down the drivers.
    flush-wait-on-shutdown = 2 s

    # Reuse inbound connections for outbound messages
    use-passive-connections = on

    # Controls the backoff interval after a refused write is reattempted.
    # (Transports may refuse writes if their internal buffer is full)
    backoff-interval = 5 ms

    # Acknowledgment timeout of management commands sent to the transport stack.
    command-ack-timeout = 30 s

    # The timeout for outbound associations to perform the handshake.
    # If the transport is akka.remote.classic.netty.tcp or akka.remote.classic.netty.ssl
    # the configured connection-timeout for the transport will be used instead.
    handshake-timeout = 15 s

    ### Security settings

    # Enable untrusted mode for full security of server managed actors, prevents
    # system messages to be send by clients, e.g. messages like 'Create',
    # 'Suspend', 'Resume', 'Terminate', 'Supervise', 'Link' etc.
    untrusted-mode = off

    # When 'untrusted-mode=on' inbound actor selections are by default discarded.
    # Actors with paths defined in this list are granted permission to receive actor
    # selections messages.
    # E.g. trusted-selection-paths = ["/user/receptionist", "/user/namingService"]
    trusted-selection-paths = []

    ### Logging

    # If this is "on", Akka will log all inbound messages at DEBUG level,
    # if off then they are not logged
    log-received-messages = off

    # If this is "on", Akka will log all outbound messages at DEBUG level,
    # if off then they are not logged
    log-sent-messages = off

    # Sets the log granularity level at which Akka logs remoting events. This setting
    # can take the values OFF, ERROR, WARNING, INFO, DEBUG, or ON. For compatibility
    # reasons the setting "on" will default to "debug" level. Please note that the effective
    # logging level is still determined by the global logging level of the actor system:
    # for example debug level remoting events will be only logged if the system
    # is running with debug level logging.
    # Failures to deserialize received messages also fall under this flag.
    log-remote-lifecycle-events = on

    # Logging of message types with payload size in bytes larger than
    # this value. Maximum detected size per message type is logged once,
    # with an increase threshold of 10%.
    # By default this feature is turned off. Activate it by setting the property to
    # a value in bytes, such as 1000b. Note that for all messages larger than this
    # limit there will be extra performance and scalability cost.
    log-frame-size-exceeding = off

    # Log warning if the number of messages in the backoff buffer in the endpoint
    # writer exceeds this limit. It can be disabled by setting the value to off.
    log-buffer-size-exceeding = 50000

    # After failed to establish an outbound connection, the remoting will mark the
    # address as failed. This configuration option controls how much time should
    # be elapsed before reattempting a new connection. While the address is
    # gated, all messages sent to the address are delivered to dead-letters.
    # Since this setting limits the rate of reconnects setting it to a
    # very short interval (i.e. less than a second) may result in a storm of
    # reconnect attempts.
    retry-gate-closed-for = 5 s

    # After catastrophic communication failures that result in the loss of system
    # messages or after the remote DeathWatch triggers the remote system gets
    # quarantined to prevent inconsistent behavior.
    # This setting controls how long the Quarantine marker will be kept around
    # before being removed to avoid long-term memory leaks.
    # WARNING: DO NOT change this to a small value to re-enable communication with
    # quarantined nodes. Such feature is not supported and any behavior between
    # the affected systems after lifting the quarantine is undefined.
    prune-quarantine-marker-after = 5 d

    # If system messages have been exchanged between two systems (i.e. remote death
    # watch or remote deployment has been used) a remote system will be marked as
    # quarantined after the two system has no active association, and no
    # communication happens during the time configured here.
    # The only purpose of this setting is to avoid storing system message redelivery
    # data (sequence number state, etc.) for an undefined amount of time leading to long
    # term memory leak. Instead, if a system has been gone for this period,
    # or more exactly
    # - there is no association between the two systems (TCP connection, if TCP transport is used)
    # - neither side has been attempting to communicate with the other
    # - there are no pending system messages to deliver
    # for the amount of time configured here, the remote system will be quarantined and all state
    # associated with it will be dropped.
    #
    # Maximum value depends on the scheduler's max limit (default 248 days) and if configured
    # to a longer duration this feature will effectively be disabled. Setting the value to
    # 'off' will also disable the feature. Note that if disabled there is a risk of a long
    # term memory leak.
    quarantine-after-silence = 2 d

    # This setting defines the maximum number of unacknowledged system messages
    # allowed for a remote system. If this limit is reached the remote system is
    # declared to be dead and its UID marked as tainted.
    system-message-buffer-size = 20000

    # This setting defines the maximum idle time after an individual
    # acknowledgement for system messages is sent. System message delivery
    # is guaranteed by explicit acknowledgement messages. These acks are
    # piggybacked on ordinary traffic messages. If no traffic is detected
    # during the time period configured here, the remoting will send out
    # an individual ack.
    system-message-ack-piggyback-timeout = 0.3 s

    # This setting defines the time after internal management signals
    # between actors (used for DeathWatch and supervision) that have not been
    # explicitly acknowledged or negatively acknowledged are resent.
    # Messages that were negatively acknowledged are always immediately
    # resent.
    resend-interval = 2 s

    # Maximum number of unacknowledged system messages that will be resent
    # each 'resend-interval'. If you watch many (> 1000) remote actors you can
    # increase this value to for example 600, but a too large limit (e.g. 10000)
    # may flood the connection and might cause false failure detection to trigger.
    # Test such a configuration by watching all actors at the same time and stop
    # all watched actors at the same time.
    resend-limit = 200

    # WARNING: this setting should not be not changed unless all of its consequences
    # are properly understood which assumes experience with remoting internals
    # or expert advice.
    # This setting defines the time after redelivery attempts of internal management
    # signals are stopped to a remote system that has been not confirmed to be alive by
    # this system before.
    initial-system-message-delivery-timeout = 3 m

    ### Transports and adapters

    # List of the transport drivers that will be loaded by the remoting.
    # A list of fully qualified config paths must be provided where
    # the given configuration path contains a transport-class key
    # pointing to an implementation class of the Transport interface.
    # If multiple transports are provided, the address of the first
    # one will be used as a default address.
    enabled-transports = ["akka.remote.classic.netty.tcp"]

    # Transport drivers can be augmented with adapters by adding their
    # name to the applied-adapters setting in the configuration of a
    # transport. The available adapters should be configured in this
    # section by providing a name, and the fully qualified name of
    # their corresponding implementation. The class given here
    # must implement akka.akka.remote.transport.TransportAdapterProvider
    # and have public constructor without parameters.
    adapters {
      gremlin = "akka.remote.transport.FailureInjectorProvider"
      trttl = "akka.remote.transport.ThrottlerProvider"
    }

    ### Default configuration for the Netty based transport drivers

    netty.tcp {
      # The class given here must implement the akka.remote.transport.Transport
      # interface and offer a public constructor which takes two arguments:
      #  1) akka.actor.ExtendedActorSystem
      #  2) com.typesafe.config.Config
      transport-class = "akka.remote.transport.netty.NettyTransport"

      # Transport drivers can be augmented with adapters by adding their
      # name to the applied-adapters list. The last adapter in the
      # list is the adapter immediately above the driver, while
      # the first one is the top of the stack below the standard
      # Akka protocol
      applied-adapters = []

      # The default remote server port clients should connect to.
      # Default is 2552 (AKKA), use 0 if you want a random available port
      # This port needs to be unique for each actor system on the same machine.
      port = 2552

      # The hostname or ip clients should connect to.
      # InetAddress.getLocalHost.getHostAddress is used if empty
      hostname = ""

      # Use this setting to bind a network interface to a different port
      # than remoting protocol expects messages at. This may be used
      # when running akka nodes in a separated networks (under NATs or docker containers).
      # Use 0 if you want a random available port. Examples:
      #
      # akka.remote.classic.netty.tcp.port = 2552
      # akka.remote.classic.netty.tcp.bind-port = 2553
      # Network interface will be bound to the 2553 port, but remoting protocol will
      # expect messages sent to port 2552.
      #
      # akka.remote.classic.netty.tcp.port = 0
      # akka.remote.classic.netty.tcp.bind-port = 0
      # Network interface will be bound to a random port, and remoting protocol will
      # expect messages sent to the bound port.
      #
      # akka.remote.classic.netty.tcp.port = 2552
      # akka.remote.classic.netty.tcp.bind-port = 0
      # Network interface will be bound to a random port, but remoting protocol will
      # expect messages sent to port 2552.
      #
      # akka.remote.classic.netty.tcp.port = 0
      # akka.remote.classic.netty.tcp.bind-port = 2553
      # Network interface will be bound to the 2553 port, and remoting protocol will
      # expect messages sent to the bound port.
      #
      # akka.remote.classic.netty.tcp.port = 2552
      # akka.remote.classic.netty.tcp.bind-port = ""
      # Network interface will be bound to the 2552 port, and remoting protocol will
      # expect messages sent to the bound port.
      #
      # akka.remote.classic.netty.tcp.port if empty
      bind-port = ""

      # Use this setting to bind a network interface to a different hostname or ip
      # than remoting protocol expects messages at.
      # Use "0.0.0.0" to bind to all interfaces.
      # akka.remote.classic.netty.tcp.hostname if empty
      bind-hostname = ""

      # Enables SSL support on this transport
      enable-ssl = false

      # Sets the connectTimeoutMillis of all outbound connections,
      # i.e. how long a connect may take until it is timed out
      connection-timeout = 15 s

      # If set to "<id.of.dispatcher>" then the specified dispatcher
      # will be used to accept inbound connections, and perform IO. If "" then
      # dedicated threads will be used.
      # Please note that the Netty driver only uses this configuration and does
      # not read the "akka.remote.use-dispatcher" entry. Instead it has to be
      # configured manually to point to the same dispatcher if needed.
      use-dispatcher-for-io = ""

      # Sets the high water mark for the in and outbound sockets,
      # set to 0b for platform default
      write-buffer-high-water-mark = 0b

      # Sets the low water mark for the in and outbound sockets,
      # set to 0b for platform default
      write-buffer-low-water-mark = 0b

      # Sets the send buffer size of the Sockets,
      # set to 0b for platform default
      send-buffer-size = 256000b

      # Sets the receive buffer size of the Sockets,
      # set to 0b for platform default
      receive-buffer-size = 256000b

      # Maximum message size the transport will accept, but at least
      # 32000 bytes.
      # Please note that UDP does not support arbitrary large datagrams,
      # so this setting has to be chosen carefully when using UDP.
      # Both send-buffer-size and receive-buffer-size settings has to
      # be adjusted to be able to buffer messages of maximum size.
      maximum-frame-size = 128000b

      # Sets the size of the connection backlog
      backlog = 4096

      # Enables the TCP_NODELAY flag, i.e. disables Nagle’s algorithm
      tcp-nodelay = on

      # Enables TCP Keepalive, subject to the O/S kernel’s configuration
      tcp-keepalive = on

      # Enables SO_REUSEADDR, which determines when an ActorSystem can open
      # the specified listen port (the meaning differs between *nix and Windows)
      # Valid values are "on", "off" and "off-for-windows"
      # due to the following Windows bug: http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=4476378
      # "off-for-windows" of course means that it's "on" for all other platforms
      tcp-reuse-addr = off-for-windows

      # Used to configure the number of I/O worker threads on server sockets
      server-socket-worker-pool {
        # Min number of threads to cap factor-based number to
        pool-size-min = 2

        # The pool size factor is used to determine thread pool size
        # using the following formula: ceil(available processors * factor).
        # Resulting size is then bounded by the pool-size-min and
        # pool-size-max values.
        pool-size-factor = 1.0

        # Max number of threads to cap factor-based number to
        pool-size-max = 2
      }

      # Used to configure the number of I/O worker threads on client sockets
      client-socket-worker-pool {
        # Min number of threads to cap factor-based number to
        pool-size-min = 2

        # The pool size factor is used to determine thread pool size
        # using the following formula: ceil(available processors * factor).
        # Resulting size is then bounded by the pool-size-min and
        # pool-size-max values.
        pool-size-factor = 1.0

        # Max number of threads to cap factor-based number to
        pool-size-max = 2
      }


    }

    netty.ssl = ${akka.remote.classic.netty.tcp}
    netty.ssl = {
      # Enable SSL/TLS encryption.
      # This must be enabled on both the client and server to work.
      enable-ssl = true

      # Factory of SSLEngine.
      # Must implement akka.remote.transport.netty.SSLEngineProvider and have a public
      # constructor with an ActorSystem parameter.
      # The default ConfigSSLEngineProvider is configured by properties in section
      # akka.remote.classic.netty.ssl.security
      #
      # The SSLEngineProvider can also be defined via ActorSystemSetup with
      # SSLEngineProviderSetup  when starting the ActorSystem. That is useful when
      # the SSLEngineProvider implementation requires other external constructor
      # parameters or is created before the ActorSystem is created.
      # If such SSLEngineProviderSetup is defined this config property is not used.
      ssl-engine-provider = akka.remote.transport.netty.ConfigSSLEngineProvider

      security {
        # This is the Java Key Store used by the server connection
        key-store = "keystore"

        # This password is used for decrypting the key store
        key-store-password = "changeme"

        # This password is used for decrypting the key
        key-password = "changeme"

        # This is the Java Key Store used by the client connection
        trust-store = "truststore"

        # This password is used for decrypting the trust store
        trust-store-password = "changeme"

        # Protocol to use for SSL encryption.
        protocol = "TLSv1.2"

        # Example: ["TLS_DHE_RSA_WITH_AES_128_GCM_SHA256", 
        #   "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256",
        #   "TLS_DHE_RSA_WITH_AES_256_GCM_SHA384",
        #   "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384"]
        # When doing rolling upgrades, make sure to include both the algorithm used 
        # by old nodes and the preferred algorithm.
        # If you use a JDK 8 prior to 8u161 you need to install
        # the JCE Unlimited Strength Jurisdiction Policy Files to use AES 256.
        # More info here:
        # https://www.oracle.com/java/technologies/javase-jce-all-downloads.html
        enabled-algorithms = ["TLS_DHE_RSA_WITH_AES_256_GCM_SHA384",
          "TLS_RSA_WITH_AES_128_CBC_SHA"]

        # There are two options, and the default SecureRandom is recommended:
        # "" or "SecureRandom" => (default)
        # "SHA1PRNG" => Can be slow because of blocking issues on Linux
        #
        # Setting a value here may require you to supply the appropriate cipher
        # suite (see enabled-algorithms section above)
        random-number-generator = ""

        # Require mutual authentication between TLS peers
        #
        # Without mutual authentication only the peer that actively establishes a connection (TLS client side)
        # checks if the passive side (TLS server side) sends over a trusted certificate. With the flag turned on,
        # the passive side will also request and verify a certificate from the connecting peer.
        #
        # To prevent man-in-the-middle attacks this setting is enabled by default.
        #
        # Note: Nodes that are configured with this setting to 'on' might not be able to receive messages from nodes that
        # run on older versions of akka-remote. This is because in versions of Akka < 2.4.12 the active side of the remoting
        # connection will not send over certificates even if asked.
        #
        # However, starting with Akka 2.4.12, even with this setting "off", the active side (TLS client side)
        # will use the given key-store to send over a certificate if asked. A rolling upgrade from versions of
        # Akka < 2.4.12 can therefore work like this:
        #   - upgrade all nodes to an Akka version >= 2.4.12, in the best case the latest version, but keep this setting at "off"
        #   - then switch this flag to "on" and do again a rolling upgrade of all nodes
        # The first step ensures that all nodes will send over a certificate when asked to. The second
        # step will ensure that all nodes finally enforce the secure checking of client certificates.
        require-mutual-authentication = on
      }
    }

    ### Default configuration for the failure injector transport adapter

    gremlin {
      # Enable debug logging of the failure injector transport adapter
      debug = off
    }

    backoff-remote-dispatcher {
      type = Dispatcher
      executor = "fork-join-executor"
      fork-join-executor {
        # Min number of threads to cap factor-based parallelism number to
        parallelism-min = 2
        parallelism-max = 2
      }
    }
  }
}
```



### akka-testkit

```
######################################
# Akka Testkit Reference Config File #
######################################

# This is the reference config file that contains all the default settings.
# Make your edits/overrides in your application.conf.

akka {
  test {
    # factor by which to scale timeouts during tests, e.g. to account for shared
    # build system load
    timefactor =  1.0

    # duration of EventFilter.intercept waits after the block is finished until
    # all required messages are received
    filter-leeway = 3s

    # duration to wait in expectMsg and friends outside of within() block
    # by default, will be dilated by the timefactor.
    single-expect-default = 3s

    # duration to wait in expectNoMessage by default,
    # will be dilated by the timefactor.
    expect-no-message-default = 100ms

    # The timeout that is added as an implicit by DefaultTimeout trait
    default-timeout = 5s

    calling-thread-dispatcher {
      type = akka.testkit.CallingThreadDispatcherConfigurator
    }
  }

  actor {

    serializers {
      java-test = "akka.testkit.TestJavaSerializer"
    }

    serialization-identifiers {
      "akka.testkit.TestJavaSerializer" = 23
    }

    serialization-bindings {
      "akka.testkit.JavaSerializable" = java-test
    }
  }
}
```



### akka-cluster-metrics

```

##############################################
# Akka Cluster Metrics Reference Config File #
##############################################

# This is the reference config file that contains all the default settings.
# Make your edits in your application.conf in order to override these settings.

# Sigar provisioning:
#
#  User can provision sigar classes and native library in one of the following ways:
# 
#  1) Use https://github.com/kamon-io/sigar-loader Kamon sigar-loader as a project dependency for the user project.
#  Metrics extension will extract and load sigar library on demand with help of Kamon sigar provisioner.
# 
#  2) Use https://github.com/kamon-io/sigar-loader Kamon sigar-loader as java agent: `java -javaagent:/path/to/sigar-loader.jar`
#  Kamon sigar loader agent will extract and load sigar library during JVM start.
# 
#  3) Place `sigar.jar` on the `classpath` and sigar native library for the o/s on the `java.library.path`
#  User is required to manage both project dependency and library deployment manually.

# Cluster metrics extension.
# Provides periodic statistics collection and publication throughout the cluster.
akka.cluster.metrics {
  # Full path of dispatcher configuration key.
  dispatcher = "akka.actor.default-dispatcher"
  # How long should any actor wait before starting the periodic tasks.
  periodic-tasks-initial-delay = 1s
  # Sigar native library extract location.
  # Use per-application-instance scoped location, such as program working directory.
  native-library-extract-folder = ${user.dir}"/native"
  # Metrics supervisor actor.
  supervisor {
    # Actor name. Example name space: /system/cluster-metrics
    name = "cluster-metrics"
    # Supervision strategy.
    strategy {
      #
      # FQCN of class providing `akka.actor.SupervisorStrategy`.
      # Must have a constructor with signature `<init>(com.typesafe.config.Config)`.
      # Default metrics strategy provider is a configurable extension of `OneForOneStrategy`.
      provider = "akka.cluster.metrics.ClusterMetricsStrategy"
      #
      # Configuration of the default strategy provider.
      # Replace with custom settings when overriding the provider.
      configuration = {
        # Log restart attempts.
        loggingEnabled = true
        # Child actor restart-on-failure window.
        withinTimeRange = 3s
        # Maximum number of restart attempts before child actor is stopped.
        maxNrOfRetries = 3
      }
    }
  }
  # Metrics collector actor.
  collector {
    # Enable or disable metrics collector for load-balancing nodes.
    # Metrics collection can also be controlled at runtime by sending control messages
    # to /system/cluster-metrics actor: `akka.cluster.metrics.{CollectionStartMessage,CollectionStopMessage}`
    enabled = on
    # FQCN of the metrics collector implementation.
    # It must implement `akka.cluster.metrics.MetricsCollector` and
    # have public constructor with akka.actor.ActorSystem parameter.
    # Will try to load in the following order of priority:
    # 1) configured custom collector 2) internal `SigarMetricsCollector` 3) internal `JmxMetricsCollector`
    provider = ""
    # Try all 3 available collector providers, or else fail on the configured custom collector provider.
    fallback = true
    # How often metrics are sampled on a node.
    # Shorter interval will collect the metrics more often.
    # Also controls frequency of the metrics publication to the node system event bus.
    sample-interval = 3s
    # How often a node publishes metrics information to the other nodes in the cluster.
    # Shorter interval will publish the metrics gossip more often.
    gossip-interval = 3s
    # How quickly the exponential weighting of past data is decayed compared to
    # new data. Set lower to increase the bias toward newer values.
    # The relevance of each data sample is halved for every passing half-life
    # duration, i.e. after 4 times the half-life, a data sample’s relevance is
    # reduced to 6% of its original relevance. The initial relevance of a data
    # sample is given by 1 – 0.5 ^ (collect-interval / half-life).
    # See http://en.wikipedia.org/wiki/Moving_average#Exponential_moving_average
    moving-average-half-life = 12s
  }
}

# Cluster metrics extension serializers and routers.
akka.actor {
  # Protobuf serializer for remote cluster metrics messages.
  serializers {
    akka-cluster-metrics = "akka.cluster.metrics.protobuf.MessageSerializer"
  }
  # Interface binding for remote cluster metrics messages.
  serialization-bindings {
    "akka.cluster.metrics.ClusterMetricsMessage" = akka-cluster-metrics
    "akka.cluster.metrics.AdaptiveLoadBalancingPool" = akka-cluster-metrics
    "akka.cluster.metrics.MixMetricsSelector" = akka-cluster-metrics
    "akka.cluster.metrics.CpuMetricsSelector$" = akka-cluster-metrics
    "akka.cluster.metrics.HeapMetricsSelector$" = akka-cluster-metrics
    "akka.cluster.metrics.SystemLoadAverageMetricsSelector$" = akka-cluster-metrics
  }
  # Globally unique metrics extension serializer identifier.
  serialization-identifiers {
    "akka.cluster.metrics.protobuf.MessageSerializer" = 10
  }
  #  Provide routing of messages based on cluster metrics.
  router.type-mapping {
    cluster-metrics-adaptive-pool  = "akka.cluster.metrics.AdaptiveLoadBalancingPool"
    cluster-metrics-adaptive-group = "akka.cluster.metrics.AdaptiveLoadBalancingGroup"
  }
}
```



### akka-cluster-tools

```

############################################
# Akka Cluster Tools Reference Config File #
############################################

# This is the reference config file that contains all the default settings.
# Make your edits/overrides in your application.conf.

# //#pub-sub-ext-config
# Settings for the DistributedPubSub extension
akka.cluster.pub-sub {
  # Actor name of the mediator actor, /system/distributedPubSubMediator
  name = distributedPubSubMediator

  # Start the mediator on members tagged with this role.
  # All members are used if undefined or empty.
  role = ""

  # The routing logic to use for 'Send'
  # Possible values: random, round-robin, broadcast
  routing-logic = random

  # How often the DistributedPubSubMediator should send out gossip information
  gossip-interval = 1s

  # Removed entries are pruned after this duration
  removed-time-to-live = 120s

  # Maximum number of elements to transfer in one message when synchronizing the registries.
  # Next chunk will be transferred in next round of gossip.
  max-delta-elements = 3000

  # When a message is published to a topic with no subscribers send it to the dead letters.
  send-to-dead-letters-when-no-subscribers = on
  
  # The id of the dispatcher to use for DistributedPubSubMediator actors. 
  # If specified you need to define the settings of the actual dispatcher.
  use-dispatcher = "akka.actor.internal-dispatcher"
}
# //#pub-sub-ext-config

# Protobuf serializer for cluster DistributedPubSubMeditor messages
akka.actor {
  serializers {
    akka-pubsub = "akka.cluster.pubsub.protobuf.DistributedPubSubMessageSerializer"
  }
  serialization-bindings {
    "akka.cluster.pubsub.DistributedPubSubMessage" = akka-pubsub
    "akka.cluster.pubsub.DistributedPubSubMediator$Internal$SendToOneSubscriber" = akka-pubsub
  }
  serialization-identifiers {
    "akka.cluster.pubsub.protobuf.DistributedPubSubMessageSerializer" = 9
  }
}


# //#receptionist-ext-config
# Settings for the ClusterClientReceptionist extension
akka.cluster.client.receptionist {
  # Actor name of the ClusterReceptionist actor, /system/receptionist
  name = receptionist

  # Start the receptionist on members tagged with this role.
  # All members are used if undefined or empty.
  role = ""

  # The receptionist will send this number of contact points to the client
  number-of-contacts = 3

  # The actor that tunnel response messages to the client will be stopped
  # after this time of inactivity.
  response-tunnel-receive-timeout = 30s
  
  # The id of the dispatcher to use for ClusterReceptionist actors.
  # If specified you need to define the settings of the actual dispatcher.
  use-dispatcher = "akka.actor.internal-dispatcher"

  # How often failure detection heartbeat messages should be received for
  # each ClusterClient
  heartbeat-interval = 2s

  # Number of potentially lost/delayed heartbeats that will be
  # accepted before considering it to be an anomaly.
  # The ClusterReceptionist is using the akka.remote.DeadlineFailureDetector, which
  # will trigger if there are no heartbeats within the duration
  # heartbeat-interval + acceptable-heartbeat-pause, i.e. 15 seconds with
  # the default settings.
  acceptable-heartbeat-pause = 13s

  # Failure detection checking interval for checking all ClusterClients
  failure-detection-interval = 2s
}
# //#receptionist-ext-config

# //#cluster-client-config
# Settings for the ClusterClient
akka.cluster.client {
  # Actor paths of the ClusterReceptionist actors on the servers (cluster nodes)
  # that the client will try to contact initially. It is mandatory to specify
  # at least one initial contact. 
  # Comma separated full actor paths defined by a string on the form of
  # "akka://system@hostname:port/system/receptionist"
  initial-contacts = []
  
  # Interval at which the client retries to establish contact with one of 
  # ClusterReceptionist on the servers (cluster nodes)
  establishing-get-contacts-interval = 3s
  
  # Interval at which the client will ask the ClusterReceptionist for
  # new contact points to be used for next reconnect.
  refresh-contacts-interval = 60s
  
  # How often failure detection heartbeat messages should be sent
  heartbeat-interval = 2s
  
  # Number of potentially lost/delayed heartbeats that will be
  # accepted before considering it to be an anomaly.
  # The ClusterClient is using the akka.remote.DeadlineFailureDetector, which
  # will trigger if there are no heartbeats within the duration 
  # heartbeat-interval + acceptable-heartbeat-pause, i.e. 15 seconds with
  # the default settings.
  acceptable-heartbeat-pause = 13s
  
  # If connection to the receptionist is not established the client will buffer
  # this number of messages and deliver them the connection is established.
  # When the buffer is full old messages will be dropped when new messages are sent
  # via the client. Use 0 to disable buffering, i.e. messages will be dropped
  # immediately if the location of the singleton is unknown.
  # Maximum allowed buffer size is 10000.
  buffer-size = 1000

  # If connection to the receiptionist is lost and the client has not been
  # able to acquire a new connection for this long the client will stop itself.
  # This duration makes it possible to watch the cluster client and react on a more permanent
  # loss of connection with the cluster, for example by accessing some kind of
  # service registry for an updated set of initial contacts to start a new cluster client with.
  # If this is not wanted it can be set to "off" to disable the timeout and retry
  # forever.
  reconnect-timeout = off
}
# //#cluster-client-config

# Protobuf serializer for ClusterClient messages
akka.actor {
  serializers {
    akka-cluster-client = "akka.cluster.client.protobuf.ClusterClientMessageSerializer"
  }
  serialization-bindings {
    "akka.cluster.client.ClusterClientMessage" = akka-cluster-client
  }
  serialization-identifiers {
    "akka.cluster.client.protobuf.ClusterClientMessageSerializer" = 15
  }
}

# //#singleton-config
akka.cluster.singleton {
  # The actor name of the child singleton actor.
  singleton-name = "singleton"
  
  # Singleton among the nodes tagged with specified role.
  # If the role is not specified it's a singleton among all nodes in the cluster.
  role = ""
  
  # When a node is becoming oldest it sends hand-over request to previous oldest, 
  # that might be leaving the cluster. This is retried with this interval until 
  # the previous oldest confirms that the hand over has started or the previous 
  # oldest member is removed from the cluster (+ akka.cluster.down-removal-margin).
  hand-over-retry-interval = 1s
  
  # The number of retries are derived from hand-over-retry-interval and
  # akka.cluster.down-removal-margin (or ClusterSingletonManagerSettings.removalMargin),
  # but it will never be less than this property.
  # After the hand over retries and it's still not able to exchange the hand over messages
  # with the previous oldest it will restart itself by throwing ClusterSingletonManagerIsStuck,
  # to start from a clean state. After that it will still not start the singleton instance
  # until the previous oldest node has been removed from the cluster.
  # On the other side, on the previous oldest node, the same number of retries - 3 are used
  # and after that the singleton instance is stopped.
  # For large clusters it might be necessary to increase this to avoid too early timeouts while
  # gossip dissemination of the Leaving to Exiting phase occurs. For normal leaving scenarios
  # it will not be a quicker hand over by reducing this value, but in extreme failure scenarios
  # the recovery might be faster.
  min-number-of-hand-over-retries = 15

  # Config path of the lease to be taken before creating the singleton actor
  # if the lease is lost then the actor is restarted and it will need to re-acquire the lease
  # the default is no lease
  use-lease = ""

  # The interval between retries for acquiring the lease
  lease-retry-interval = 5s
}
# //#singleton-config

# //#singleton-proxy-config
akka.cluster.singleton-proxy {
  # The actor name of the singleton actor that is started by the ClusterSingletonManager
  singleton-name = ${akka.cluster.singleton.singleton-name}
  
  # The role of the cluster nodes where the singleton can be deployed.
  # Corresponding to the role used by the `ClusterSingletonManager`. If the role is not
  # specified it's a singleton among all nodes in the cluster, and the `ClusterSingletonManager`
  # must then also be configured in same way.
  role = ""
  
  # Interval at which the proxy will try to resolve the singleton instance.
  singleton-identification-interval = 1s
  
  # If the location of the singleton is unknown the proxy will buffer this
  # number of messages and deliver them when the singleton is identified. 
  # When the buffer is full old messages will be dropped when new messages are
  # sent via the proxy.
  # Use 0 to disable buffering, i.e. messages will be dropped immediately if
  # the location of the singleton is unknown.
  # Maximum allowed buffer size is 10000.
  buffer-size = 1000 
}
# //#singleton-proxy-config

# Serializer for cluster ClusterSingleton messages
akka.actor {
  serializers {
    akka-singleton = "akka.cluster.singleton.protobuf.ClusterSingletonMessageSerializer"
  }
  serialization-bindings {
    "akka.cluster.singleton.ClusterSingletonMessage" = akka-singleton
  }
  serialization-identifiers {
    "akka.cluster.singleton.protobuf.ClusterSingletonMessageSerializer" = 14
  }
}
```



### akka-cluster-sharding-typed

```

# //#sharding-ext-config
# //#number-of-shards
akka.cluster.sharding {
  # Number of shards used by the default HashCodeMessageExtractor
  # when no other message extractor is defined. This value must be
  # the same for all nodes in the cluster and that is verified by
  # configuration check when joining. Changing the value requires
  # stopping all nodes in the cluster.
  number-of-shards = 1000
}
# //#number-of-shards
# //#sharding-ext-config


akka.cluster.sharded-daemon-process {
  # Settings for the sharded dameon process internal usage of sharding are using the akka.cluste.sharding defaults.
  # Some of the settings can be overriden specifically for the sharded daemon process here. For example can the
  # `role` setting limit what nodes the daemon processes and the keep alive pingers will run on.
  # Some settings can not be changed (remember-entitites and related settings, passivation, number-of-shards),
  # overriding those settings will be ignored.
  sharding = ${akka.cluster.sharding}

  # Each entity is pinged at this interval from each node in the
  # cluster to trigger a start if it has stopped, for example during
  # rebalancing.
  # Note: How the set of actors is kept alive may change in the future meaning this setting may go away.
  keep-alive-interval = 10s
}

akka.cluster.configuration-compatibility-check.checkers {
  akka-cluster-sharding-hash-extractor = "akka.cluster.sharding.typed.internal.JoinConfigCompatCheckerClusterSharding"
}

akka.actor {
  serializers {
    typed-sharding = "akka.cluster.sharding.typed.internal.ShardingSerializer"
  }
  serialization-identifiers {
    "akka.cluster.sharding.typed.internal.ShardingSerializer" = 25
  }
  serialization-bindings {
    "akka.cluster.sharding.typed.ShardingEnvelope" = typed-sharding
  }
}

akka.reliable-delivery {
  sharding {
    producer-controller = ${akka.reliable-delivery.producer-controller}
    producer-controller {
      # Limit of how many messages that can be buffered when there
      # is no demand from the consumer side.
      buffer-size = 1000

      # Ask timeout for sending message to worker until receiving Ack from worker
      internal-ask-timeout = 60s

      # If no messages are sent to an entity within this duration the
      # ProducerController for that entity will be removed.
      cleanup-unused-after = 120s

      # In case ShardingConsumerController is stopped and there are pending
      # unconfirmed messages the ShardingConsumerController has to "wake up"
      # the consumer again by resending the first unconfirmed message.
      resend-first-unconfirmed-idle-timeout = 10s

      # Chunked messages not implemented for sharding yet. Override to not
      # propagate property from akka.reliable-delivery.producer-controller.
      chunk-large-messages = off
    }

    consumer-controller = ${akka.reliable-delivery.consumer-controller}
    consumer-controller {
      # Limit of how many messages that can be buffered before the
      # ShardingConsumerController is initialized by the Start message.
      buffer-size = 1000
    }
  }
}
```



### akka-cluster-sharding

```
###############################################
# Akka Cluster Sharding Reference Config File #
###############################################

# This is the reference config file that contains all the default settings.
# Make your edits/overrides in your application.conf.


# //#sharding-ext-config
# Settings for the ClusterShardingExtension
akka.cluster.sharding {

  # The extension creates a top level actor with this name in top level system scope,
  # e.g. '/system/sharding'
  guardian-name = sharding

  # Specifies that entities runs on cluster nodes with a specific role.
  # If the role is not specified (or empty) all nodes in the cluster are used.
  role = ""

  # When this is set to 'on' the active entity actors will automatically be restarted
  # upon Shard restart. i.e. if the Shard is started on a different ShardRegion
  # due to rebalance or crash.
  remember-entities = off

  # When 'remember-entities' is enabled and the state store mode is ddata this controls
  # how the remembered entities and shards are stored. Possible values are "eventsourced" and "ddata"
  # Default is ddata for backwards compatibility.
  remember-entities-store = "ddata"

  # Set this to a time duration to have sharding passivate entities when they have not
  # received any message in this length of time. Set to 'off' to disable.
  # It is always disabled if `remember-entities` is enabled.
  passivate-idle-entity-after = 120s

  # If the coordinator can't store state changes it will be stopped
  # and started again after this duration, with an exponential back-off
  # of up to 5 times this duration.
  coordinator-failure-backoff = 5 s

  # The ShardRegion retries registration and shard location requests to the
  # ShardCoordinator with this interval if it does not reply.
  retry-interval = 2 s

  # Maximum number of messages that are buffered by a ShardRegion actor.
  buffer-size = 100000

  # Timeout of the shard rebalancing process.
  # Additionally, if an entity doesn't handle the stopMessage
  # after (handoff-timeout - 5.seconds).max(1.second) it will be stopped forcefully
  handoff-timeout = 60 s

  # Time given to a region to acknowledge it's hosting a shard.
  shard-start-timeout = 10 s

  # If the shard is remembering entities and can't store state changes
  # will be stopped and then started again after this duration. Any messages
  # sent to an affected entity may be lost in this process.
  shard-failure-backoff = 10 s

  # If the shard is remembering entities and an entity stops itself without
  # using passivate. The entity will be restarted after this duration or when
  # the next message for it is received, which ever occurs first.
  entity-restart-backoff = 10 s

  # Rebalance check is performed periodically with this interval.
  rebalance-interval = 10 s

  # Absolute path to the journal plugin configuration entity that is to be
  # used for the internal persistence of ClusterSharding. If not defined
  # the default journal plugin is used. Note that this is not related to
  # persistence used by the entity actors.
  # Only used when state-store-mode=persistence
  journal-plugin-id = ""

  # Absolute path to the snapshot plugin configuration entity that is to be
  # used for the internal persistence of ClusterSharding. If not defined
  # the default snapshot plugin is used. Note that this is not related to
  # persistence used by the entity actors.
  # Only used when state-store-mode=persistence
  snapshot-plugin-id = ""

  # Defines how the coordinator stores its state. Same is also used by the
  # shards for rememberEntities.
  # Valid values are "ddata" or "persistence".
  # "persistence" mode is deprecated
  state-store-mode = "ddata"

  # The shard saves persistent snapshots after this number of persistent
  # events. Snapshots are used to reduce recovery times. A snapshot trigger might be delayed if a batch of updates is processed.
  # Only used when state-store-mode=persistence
  snapshot-after = 1000

  # The shard deletes persistent events (messages and snapshots) after doing snapshot
  # keeping this number of old persistent batches.
  # Batch is of size `snapshot-after`.
  # When set to 0 after snapshot is successfully done all events with equal or lower sequence number will be deleted.
  # Default value of 2 leaves last maximum 2*`snapshot-after` events and 3 snapshots (2 old ones + latest snapshot).
  # If larger than 0, one additional batch of journal messages is kept when state-store-mode=persistence to include messages from delayed snapshots.
  keep-nr-of-batches = 2

  # Settings for LeastShardAllocationStrategy.
  #
  # A new rebalance algorithm was included in Akka 2.6.10. It can reach optimal balance in
  # less rebalance rounds (typically 1 or 2 rounds). The amount of shards to rebalance in each
  # round can still be limited to make it progress slower. For backwards compatibility
  # the new algorithm is not enabled by default. Enable the new algorithm by setting
  # `rebalance-absolute-limit` > 0, for example:
  # akka.cluster.sharding.least-shard-allocation-strategy.rebalance-absolute-limit=20
  # The new algorithm is recommended and will become the default in future versions of Akka.
  least-shard-allocation-strategy {
    # Maximum number of shards that will be rebalanced in one rebalance round.
    # The lower of this and `rebalance-relative-limit` will be used.
    rebalance-absolute-limit = 0

    # Maximum number of shards that will be rebalanced in one rebalance round.
    # Fraction of total number of (known) shards.
    # The lower of this and `rebalance-absolute-limit` will be used.
    rebalance-relative-limit = 0.1

    # Deprecated: Use rebalance-absolute-limit and rebalance-relative-limit instead. This property is not used
    # when rebalance-absolute-limit > 0.
    #
    # Threshold of how large the difference between most and least number of
    # allocated shards must be to begin the rebalancing.
    # The difference between number of shards in the region with most shards and
    # the region with least shards must be greater than (>) the `rebalanceThreshold`
    # for the rebalance to occur.
    # It is also the maximum number of shards that will start rebalancing per rebalance-interval
    # 1 gives the best distribution and therefore typically the best choice.
    # Increasing the threshold can result in quicker rebalance but has the
    # drawback of increased difference between number of shards (and therefore load)
    # on different nodes before rebalance will occur.
    rebalance-threshold = 1

    # Deprecated: Use rebalance-absolute-limit and rebalance-relative-limit instead. This property is not used
    # when rebalance-absolute-limit > 0.
    #
    # The number of ongoing rebalancing processes is limited to this number.
    max-simultaneous-rebalance = 3
  }

  external-shard-allocation-strategy {
    # How long to wait for the client to persist an allocation to ddata or get a all shard locations
    client-timeout = 5s
  }

  # Timeout of waiting the initial distributed state for the shard coordinator (an initial state will be queried again if the timeout happened)
  # and for a shard to get its state when remembered entities is enabled
  # The read from ddata is a ReadMajority, for small clusters (< majority-min-cap) every node needs to respond
  # so is more likely to time out if there are nodes restarting e.g. when there is a rolling re-deploy happening
  waiting-for-state-timeout = 2 s

  # Timeout of waiting for update the distributed state (update will be retried if the timeout happened)
  # Also used as timeout for writes of remember entities when that is enabled
  updating-state-timeout = 5 s

  # Timeout to wait for querying all shards for a given `ShardRegion`.
  shard-region-query-timeout = 3 s

  # The shard uses this strategy to determines how to recover the underlying entity actors. The strategy is only used
  # by the persistent shard when rebalancing or restarting and is applied per remembered shard starting up (not for
  # entire shard region). The value can either be "all" or "constant". The "all"
  # strategy start all the underlying entity actors at the same time. The constant strategy will start the underlying
  # entity actors at a fix rate. The default strategy "all".
  entity-recovery-strategy = "all"

  # Default settings for the constant rate entity recovery strategy
  entity-recovery-constant-rate-strategy {
    # Sets the frequency at which a batch of entity actors is started.
    frequency = 100 ms
    # Sets the number of entity actors to be restart at a particular interval
    number-of-entities = 5
  }

  event-sourced-remember-entities-store {
    # When using remember entities and the event sourced remember entities store the batches
    # written to the store are limited by this number to avoid getting a too large event for
    # the journal to handle. If using long persistence ids you may have to increase this.
    max-updates-per-write = 100
  }

  # Settings for the coordinator singleton. Same layout as akka.cluster.singleton.
  # The "role" of the singleton configuration is not used. The singleton role will
  # be the same as "akka.cluster.sharding.role".
  # A lease can be configured in these settings for the coordinator singleton
  coordinator-singleton = ${akka.cluster.singleton}

  coordinator-state {
    # State updates are required to be written to a majority of nodes plus this
    # number of additional nodes. Can also be set to "all" to require
    # writes to all nodes. The reason for write/read to more than majority
    # is to have more tolerance for membership changes between write and read.
    # The tradeoff of increasing this is that updates will be slower.
    # It is more important to increase the `read-majority-plus`.
    write-majority-plus = 5
    # State retrieval when ShardCoordinator is started is required to be read
    # from a majority of nodes plus this number of additional nodes. Can also
    # be set to "all" to require reads from all nodes. The reason for write/read
    # to more than majority is to have more tolerance for membership changes between
    # write and read.
    # The tradeoff of increasing this is that coordinator startup will be slower.
    read-majority-plus = 5
  }
  
  # Settings for the Distributed Data replicator. 
  # Same layout as akka.cluster.distributed-data.
  # The "role" of the distributed-data configuration is not used. The distributed-data
  # role will be the same as "akka.cluster.sharding.role".
  # Note that there is one Replicator per role and it's not possible
  # to have different distributed-data settings for different sharding entity types.
  # Only used when state-store-mode=ddata
  distributed-data = ${akka.cluster.distributed-data}
  distributed-data {
    # minCap parameter to MajorityWrite and MajorityRead consistency level.
    majority-min-cap = 5
    durable.keys = ["shard-*"]
    
    # When using many entities with "remember entities" the Gossip message
    # can become to large if including to many in same message. Limit to
    # the same number as the number of ORSet per shard.
    max-delta-elements = 5

    # ShardCoordinator is singleton running on oldest
    prefer-oldest = on
  }

  # The id of the dispatcher to use for ClusterSharding actors.
  # If specified you need to define the settings of the actual dispatcher.
  # This dispatcher for the entity actors is defined by the user provided
  # Props, i.e. this dispatcher is not used for the entity actors.
  use-dispatcher = "akka.actor.internal-dispatcher"

  # Config path of the lease that each shard must acquire before starting entity actors
  # default is no lease
  # A lease can also be used for the singleton coordinator by settings it in the coordinator-singleton properties
  use-lease = ""

  # The interval between retries for acquiring the lease
  lease-retry-interval = 5s

  # Provide a higher level of details in the debug logs, often per routed message. Be careful about enabling
  # in production systems.
  verbose-debug-logging = off

  # Throw an exception if the internal state machine in the Shard actor does an invalid state transition.
  # Mostly for the Akka test suite, if off the invalid transition is logged as a warning instead of throwing and
  # crashing the shard.
  fail-on-invalid-entity-state-transition = off

  # Healthcheck that can be used with Akka management health checks: https://doc.akka.io/docs/akka-management/current/healthchecks.html
  healthcheck {
    # sharding names to check have registered with the coordinator for the health check to pass
    # once initial registration has taken place the health check always returns true to prevent the coordinator
    # moving making the health check of all nodes fail
    # by default no sharding instances are monitored
    names = []

    # Timeout for the local shard region to respond. This should be lower than your monitoring system's
    # timeout for health checks
    timeout = 5s
  }
}
# //#sharding-ext-config

# Enable health check by default for when Akka management is on the classpath
akka.management.health-checks.readiness-checks {
  sharding = "akka.cluster.sharding.ClusterShardingHealthCheck"
}

akka.cluster {
  configuration-compatibility-check {
    checkers {
      akka-cluster-sharding = "akka.cluster.sharding.JoinConfigCompatCheckSharding"
    }
  }
}

# Protobuf serializer for Cluster Sharding messages
akka.actor {
  serializers {
    akka-sharding = "akka.cluster.sharding.protobuf.ClusterShardingMessageSerializer"
  }
  serialization-bindings {
    "akka.cluster.sharding.ClusterShardingSerializable" = akka-sharding
  }
  serialization-identifiers {
    "akka.cluster.sharding.protobuf.ClusterShardingMessageSerializer" = 13
  }
}
```



### akka-distributed-data

```

##############################################
# Akka Distributed DataReference Config File #
##############################################

# This is the reference config file that contains all the default settings.
# Make your edits/overrides in your application.conf.


#//#distributed-data
# Settings for the DistributedData extension
akka.cluster.distributed-data {
  # Actor name of the Replicator actor, /system/ddataReplicator
  name = ddataReplicator

  # Replicas are running on members tagged with this role.
  # All members are used if undefined or empty.
  role = ""

  # How often the Replicator should send out gossip information
  gossip-interval = 2 s
  
  # How often the subscribers will be notified of changes, if any
  notify-subscribers-interval = 500 ms

  # Logging of data with payload size in bytes larger than
  # this value. Maximum detected size per key is logged once,
  # with an increase threshold of 10%.
  # It can be disabled by setting the property to off.
  log-data-size-exceeding = 10 KiB

  # Maximum number of entries to transfer in one round of gossip exchange when
  # synchronizing the replicas. Next chunk will be transferred in next round of gossip.
  # The actual number of data entries in each Gossip message is dynamically
  # adjusted to not exceed the maximum remote message size (maximum-frame-size).
  max-delta-elements = 500
  
  # The id of the dispatcher to use for Replicator actors.
  # If specified you need to define the settings of the actual dispatcher.
  use-dispatcher = "akka.actor.internal-dispatcher"

  # How often the Replicator checks for pruning of data associated with
  # removed cluster nodes. If this is set to 'off' the pruning feature will
  # be completely disabled.
  pruning-interval = 120 s
  
  # How long time it takes to spread the data to all other replica nodes.
  # This is used when initiating and completing the pruning process of data associated
  # with removed cluster nodes. The time measurement is stopped when any replica is 
  # unreachable, but it's still recommended to configure this with certain margin.
  # It should be in the magnitude of minutes even though typical dissemination time
  # is shorter (grows logarithmic with number of nodes). There is no advantage of 
  # setting this too low. Setting it to large value will delay the pruning process.
  max-pruning-dissemination = 300 s
  
  # The markers of that pruning has been performed for a removed node are kept for this
  # time and thereafter removed. If and old data entry that was never pruned is somehow
  # injected and merged with existing data after this time the value will not be correct.
  # This would be possible (although unlikely) in the case of a long network partition.
  # It should be in the magnitude of hours. For durable data it is configured by 
  # 'akka.cluster.distributed-data.durable.pruning-marker-time-to-live'.
 pruning-marker-time-to-live = 6 h
  
  # Serialized Write and Read messages are cached when they are sent to 
  # several nodes. If no further activity they are removed from the cache
  # after this duration.
  serializer-cache-time-to-live = 10s

  # Update and Get operations are sent to oldest nodes first.
  # This is useful together with Cluster Singleton, which is running on oldest nodes.
  prefer-oldest = off
  
  # Settings for delta-CRDT
  delta-crdt {
    # enable or disable delta-CRDT replication
    enabled = on
    
    # Some complex deltas grow in size for each update and above this
    # threshold such deltas are discarded and sent as full state instead.
    # This is number of elements or similar size hint, not size in bytes.
    max-delta-size = 50
  }
  
  durable {
    # List of keys that are durable. Prefix matching is supported by using * at the
    # end of a key.  
    keys = []
    
    # The markers of that pruning has been performed for a removed node are kept for this
    # time and thereafter removed. If and old data entry that was never pruned is
    # injected and merged with existing data after this time the value will not be correct.
    # This would be possible if replica with durable data didn't participate in the pruning
    # (e.g. it was shutdown) and later started after this time. A durable replica should not 
    # be stopped for longer time than this duration and if it is joining again after this
    # duration its data should first be manually removed (from the lmdb directory).
    # It should be in the magnitude of days. Note that there is a corresponding setting
    # for non-durable data: 'akka.cluster.distributed-data.pruning-marker-time-to-live'.
    pruning-marker-time-to-live = 10 d
    
    # Fully qualified class name of the durable store actor. It must be a subclass
    # of akka.actor.Actor and handle the protocol defined in 
    # akka.cluster.ddata.DurableStore. The class must have a constructor with 
    # com.typesafe.config.Config parameter.
    store-actor-class = akka.cluster.ddata.LmdbDurableStore
    
    use-dispatcher = akka.cluster.distributed-data.durable.pinned-store
    
    pinned-store {
      executor = thread-pool-executor
      type = PinnedDispatcher
    }
    
    # Config for the LmdbDurableStore
    lmdb {
      # Directory of LMDB file. There are two options:
      # 1. A relative or absolute path to a directory that ends with 'ddata'
      #    the full name of the directory will contain name of the ActorSystem
      #    and its remote port.
      # 2. Otherwise the path is used as is, as a relative or absolute path to
      #    a directory.
      #
      # When running in production you may want to configure this to a specific
      # path (alt 2), since the default directory contains the remote port of the
      # actor system to make the name unique. If using a dynamically assigned 
      # port (0) it will be different each time and the previously stored data 
      # will not be loaded.
      dir = "ddata"
      
      # Size in bytes of the memory mapped file.
      map-size = 100 MiB
      
      # Accumulate changes before storing improves performance with the
      # risk of losing the last writes if the JVM crashes.
      # The interval is by default set to 'off' to write each update immediately.
      # Enabling write behind by specifying a duration, e.g. 200ms, is especially 
      # efficient when performing many writes to the same key, because it is only 
      # the last value for each key that will be serialized and stored.  
      # write-behind-interval = 200 ms
      write-behind-interval = off
    }
  }
  
}
#//#distributed-data

# Protobuf serializer for cluster DistributedData messages
akka.actor {
  serializers {
    akka-data-replication = "akka.cluster.ddata.protobuf.ReplicatorMessageSerializer"
    akka-replicated-data = "akka.cluster.ddata.protobuf.ReplicatedDataSerializer"
  }
  serialization-bindings {
    "akka.cluster.ddata.Replicator$ReplicatorMessage" = akka-data-replication
    "akka.cluster.ddata.ReplicatedDataSerialization" = akka-replicated-data
  }
  serialization-identifiers {
    "akka.cluster.ddata.protobuf.ReplicatedDataSerializer" = 11
    "akka.cluster.ddata.protobuf.ReplicatorMessageSerializer" = 12
  }
}
```



### akka-stream

```

#####################################
# Akka Stream Reference Config File #
#####################################

# eager creation of the system wide materializer
akka.library-extensions += "akka.stream.SystemMaterializer$"
akka {
  stream {

    # Default materializer settings
    materializer {

      # Initial size of buffers used in stream elements
      initial-input-buffer-size = 4
      # Maximum size of buffers used in stream elements
      max-input-buffer-size = 16

      # Fully qualified config path which holds the dispatcher configuration
      # or full dispatcher configuration to be used by ActorMaterializer when creating Actors.
      dispatcher = "akka.actor.default-dispatcher"

      # Fully qualified config path which holds the dispatcher configuration
      # or full dispatcher configuration to be used by stream operators that
      # perform blocking operations
      blocking-io-dispatcher = "akka.actor.default-blocking-io-dispatcher"

      # Cleanup leaked publishers and subscribers when they are not used within a given
      # deadline
      subscription-timeout {
        # when the subscription timeout is reached one of the following strategies on
        # the "stale" publisher:
        # cancel - cancel it (via `onError` or subscribing to the publisher and
        #          `cancel()`ing the subscription right away
        # warn   - log a warning statement about the stale element (then drop the
        #          reference to it)
        # noop   - do nothing (not recommended)
        mode = cancel

        # time after which a subscriber / publisher is considered stale and eligible
        # for cancelation (see `akka.stream.subscription-timeout.mode`)
        timeout = 5s
      }

      # Enable additional troubleshooting logging at DEBUG log level
      debug-logging = off

      # Maximum number of elements emitted in batch if downstream signals large demand
      output-burst-limit = 1000

      # Enable automatic fusing of all graphs that are run. For short-lived streams
      # this may cause an initial runtime overhead, but most of the time fusing is
      # desirable since it reduces the number of Actors that are created.
      # Deprecated, since Akka 2.5.0, setting does not have any effect.
      auto-fusing = on

      # Those stream elements which have explicit buffers (like mapAsync, mapAsyncUnordered,
      # buffer, flatMapMerge, Source.actorRef, Source.queue, etc.) will preallocate a fixed
      # buffer upon stream materialization if the requested buffer size is less than this
      # configuration parameter. The default is very high because failing early is better
      # than failing under load.
      #
      # Buffers sized larger than this will dynamically grow/shrink and consume more memory
      # per element than the fixed size buffers.
      max-fixed-buffer-size = 1000000000

      # Maximum number of sync messages that actor can process for stream to substream communication.
      # Parameter allows to interrupt synchronous processing to get upstream/downstream messages.
      # Allows to accelerate message processing that happening within same actor but keep system responsive.
      sync-processing-limit = 1000

      debug {
        # Enables the fuzzing mode which increases the chance of race conditions
        # by aggressively reordering events and making certain operations more
        # concurrent than usual.
        # This setting is for testing purposes, NEVER enable this in a production
        # environment!
        # To get the best results, try combining this setting with a throughput
        # of 1 on the corresponding dispatchers.
        fuzzing-mode = off
      }

      io.tcp {
        # The outgoing bytes are accumulated in a buffer while waiting for acknowledgment
        # of pending write. This improves throughput for small messages (frames) without
        # sacrificing latency. While waiting for the ack the stage will eagerly pull
        # from upstream until the buffer exceeds this size. That means that the buffer may hold
        # slightly more bytes than this limit (at most one element more). It can be set to 0
        # to disable the usage of the buffer.
        write-buffer-size = 16 KiB
      }

      # Time to wait for async materializer creation before throwing an exception
      creation-timeout = 20 seconds

      //#stream-ref
      # configure defaults for SourceRef and SinkRef
      stream-ref {
        # Buffer of a SinkRef that is used to batch Request elements from the other side of the stream ref
        #
        # The buffer will be attempted to be filled eagerly even while the local stage did not request elements,
        # because the delay of requesting over network boundaries is much higher.
        buffer-capacity = 32

        # Demand is signalled by sending a cumulative demand message ("requesting messages until the n-th sequence number)
        # Using a cumulative demand model allows us to re-deliver the demand message in case of message loss (which should
        # be very rare in any case, yet possible -- mostly under connection break-down and re-establishment).
        #
        # The semantics of handling and updating the demand however are in-line with what Reactive Streams dictates.
        #
        # In normal operation, demand is signalled in response to arriving elements, however if no new elements arrive
        # within `demand-redelivery-interval` a re-delivery of the demand will be triggered, assuming that it may have gotten lost.
        demand-redelivery-interval = 1 second

        # Subscription timeout, during which the "remote side" MUST subscribe (materialize) the handed out stream ref.
        # This timeout does not have to be very low in normal situations, since the remote side may also need to
        # prepare things before it is ready to materialize the reference. However the timeout is needed to avoid leaking
        # in-active streams which are never subscribed to.
        subscription-timeout = 30 seconds

        # In order to guard the receiving end of a stream ref from never terminating (since awaiting a Completion or Failed
        # message) after / before a Terminated is seen, a special timeout is applied once Terminated is received by it.
        # This allows us to terminate stream refs that have been targeted to other nodes which are Downed, and as such the
        # other side of the stream ref would never send the "final" terminal message.
        #
        # The timeout specifically means the time between the Terminated signal being received and when the local SourceRef
        # determines to fail itself, assuming there was message loss or a complete partition of the completion signal.
        final-termination-signal-deadline = 2 seconds
      }
      //#stream-ref
    }

    # Deprecated, left here to not break Akka HTTP which refers to it
    blocking-io-dispatcher = "akka.actor.default-blocking-io-dispatcher"

    # Deprecated, will not be used unless user code refer to it, use 'akka.stream.materializer.blocking-io-dispatcher'
    # instead, or if from code, prefer the 'ActorAttributes.IODispatcher' attribute
    default-blocking-io-dispatcher = "akka.actor.default-blocking-io-dispatcher"
  }

  # configure overrides to ssl-configuration here (to be used by akka-streams, and akka-http – i.e. when serving https connections)
  ssl-config {
    protocol = "TLSv1.2"
  }

  actor {

    serializers {
      akka-stream-ref = "akka.stream.serialization.StreamRefSerializer"
    }

    serialization-bindings {
      "akka.stream.SinkRef"                           = akka-stream-ref
      "akka.stream.SourceRef"                         = akka-stream-ref
      "akka.stream.impl.streamref.StreamRefsProtocol" = akka-stream-ref
    }

    serialization-identifiers {
      "akka.stream.serialization.StreamRefSerializer" = 30
    }
  }
}

# ssl configuration
# folded in from former ssl-config-akka module
ssl-config {
  logger = "com.typesafe.sslconfig.akka.util.AkkaLoggerBridge"
}
```



### akka-stream-testkit

```
akka.stream.testkit {
  all-stages-stopped-timeout = 5 s
}
```

----

[Actor](../typed/index.md)

----

[英文原文](https://doc.akka.io/docs/akka/current/typed/index.html): 

