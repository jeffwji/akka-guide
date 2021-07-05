# Actors

- [Actor 介绍](actors.md)
  - [模块信息](actors.md#模块信息)
  - [Akka Actors](actors.md#akka-actors)
  - [第一个例子](actors.md#第一个例子)
  - [一个更复杂的例子](actors.md#一个更复杂的例子)
- [Actor 生命周期](actor-lifecycle.md)
  - [依赖](actor-lifecycle.md#依赖)
  - [介绍](actor-lifecycle.md#介绍)
  - [新建](actor-lifecycle.md#新建)
  - [停止](actor-lifecycle.md#停止)
  - [Watching Actors](actor-lifecycle.md#watching-actors)
- [交互模式](interaction-patterns.md)
  - [依赖](interaction-patterns.md#依赖)
  - [介绍](interaction-patterns.md#介绍)
  - [执行并遗忘](interaction-patterns.md#执行并遗忘)
  - [请求-响应](interaction-patterns.md#请求-响应)
  - [适应性反应](interaction-patterns.md#适应性反应)
  - [两个Actor之间的询问式请求响应](interaction-patterns.md#两个Actor之间的询问式请求响应)
  - [来自 actor 之外的询问式请求响应](interaction-patterns.md#来自actor之外的询问式请求响应)
  - [通用响应包装器](interaction-patterns.md#通用响应包装器)
  - [忽略回复](interaction-patterns.md#忽略回复)
  - [将 Future 结果发送给自己](interaction-patterns.md#将Future结果发送给自己)
  - [每个会话子 Actor](interaction-patterns.md#每个会话子Actor)
  - [通用响应聚合器](interaction-patterns.md#通用响应聚合器)
  - [延迟尾部斩断](interaction-patterns.md#延迟尾部斩断)
  - [给自己发送计划消息](interaction-patterns.md#给自己发送计划消息)
  - [响应分片Actor](interaction-patterns.md#响应分片Actor)
- [容错](fault-tolerance.md)
  - [监督](fault-tolerance.md#监督)
  - [父级重新启动时停止子Actor](fault-tolerance.md#父级重新启动时停止子Actor)
  - [PreRestart 信号](fault-tolerance.md#PreRestart信号)
  - [通过层次结构向上冒升失败](fault-tolerance.md#通过层次结构向上冒升失败)
- [Actor发现](actor-discovery.md)
  - [依赖](actor-discovery.md#dependency)
  - [获取 Actor 引用](actor-discovery.md#获取Actor引用)
  - [Receptionist](actor-discovery.md#receptionist)
  - [集群 Receptionist](actor-discovery.md#cluster-receptionist)
  - [Receptionist 可扩展性](actor-discovery.md#receptionist可扩展性)
- [路由](routers.md)
  - [依赖](routers.md#依赖)
  - [介绍](routers.md#介绍)
  - [池路由器](routers.md#池路由器)
  - [组路由器](routers.md#组路由器)
  - [路由策略](routers.md#路由策略)
  - [路由器及其性能](routers.md#路由器及其性能)
- [Stash](stash.md)
  - [依赖](stash.md#依赖)
  - [介绍](stash.md#介绍)
- [将 Behaviors 做为有限状态机](fsm.md)
  - [案例项目](fsm.md#案例项目)
- [协同关闭](coordinated-shutdown.md)
- [调度器](dispatchers.md)
  - [依赖](dispatchers.md#依赖)
  - [介绍](dispatchers.md#介绍)
  - [缺省调度器](dispatchers.md#缺省调度器)
  - [内部调度器](dispatchers.md#内部调度器)
  - [查找一个调度器](dispatchers.md#查找一个调度器)
  - [选择一个调度器](dispatchers.md#选择一个调度器)
  - [调度器的类型](dispatchers.md#调度器的类型)
  - [调度器别名](dispatchers.md#调度器别名)
  - [谨慎管理阻塞](dispatchers.md#需要谨慎管理阻塞)
  - [更多调度器配置示例](dispatchers.md#更多调度器配置示例)
- [邮箱](mailboxes.md)
  - [依赖](mailboxes.md#依赖)
  - [介绍](mailboxes.md#介绍)
  - [如何选择使用邮箱](mailboxes.md#如何选择使用邮箱)
  - [邮箱的实现](mailboxes.md#邮箱实现)
  - [自定义邮箱类型](mailboxes.md#自定义邮箱类型)
- [测试](testing.md)
  - [模块信息](testing.md#模块信息)
  - [介绍](testing.md#介绍)
  - [异步测试](testing-async.md)
  - [同步 behavior 测试](testing-sync.md)
- [共存](coexisting.md)
  - [依赖](coexisting.md#依赖)
  - [介绍](coexisting.md#介绍)
  - [从 Classic 到 typed](coexisting.md#从classic到typed)
  - [从 Typed 到 classic](coexisting.md#从typed到classic)
  - [监督](coexisting.md#监督)
- [编程风格指南](style-guide.md)
  - [函数式与面向对象的风格](style-guide.md#函数式与面向对象的风格)
  - [传递太多参数](style-guide.md#传递太多参数)
  - [行为工厂方法](style-guide.md#行为工厂方法)
  - [在哪里定义消息](style-guide.md#在哪里定义消息)
  - [公共消息与私人消息](style-guide.md#公共消息与私人消息)
  - [偏函数与完全函数](style-guide.md#偏函数与完全函数)
  - [如何组合偏函数](style-guide.md#如何组合偏函数)
  - ["ask" versus "?"](style-guide.md#ask-versus-)
  - [嵌套设置](style-guide.md#嵌套设置)
  - [其它命名约定](style-guide.md#其它命名约定)
- [Learning Akka Typed from Classic](from-classic.md)
  - [依赖](from-classic.md#依赖)
  - [包名](from-classic.md#包名)
  - [Actor的定义](from-classic.md#Actor的定义)
  - [actorOf 和 Props](from-classic.md#actorOf和Props)
  - [ActorRef](from-classic.md#actorref)
  - [ActorSystem](from-classic.md#actorsystem)
  - [become](from-classic.md#become)
  - [sender](from-classic.md#sender)
  - [父亲](from-classic.md#父亲)
  - [监督](from-classic.md#监督)
  - [生命周期钩子](from-classic.md#生命周期钩子)
  - [观测](from-classic.md#观测)
  - [停止](from-classic.md#停止)
  - [ActorSelection](from-classic.md#actorselection)
  - [ask](from-classic.md#ask)
  - [pipeTo](from-classic.md#pipeto)
  - [ActorContext.children](from-classic.md#actorcontext-children)
  - [Remote deployment](from-classic.md#remote-deployment)
  - [路由器](from-classic.md#路由器)
  - [FSM](from-classic.md#fsm)
  - [定时器](from-classic.md#定时器)
  - [Stash](from-classic.md#stash)
  - [PersistentActor](from-classic.md#persistentactor)
  - [异步测试](from-classic.md#异步测试)
  - [同步测试](from-classic.md#同步测试)