# 入门指南

- [Akka介绍](introduction-to-akka.md)
  - [如何开始](introduction-to-akka.md#如何开始？)
- [为什么现代系统需要新的编程模型](actors-motivation.md)
  - [封装的挑战](actors-motivation.md#封装的挑战)
  - [现代计算机体系结构上共享内存的错觉](actors-motivation.md#现代计算机体系结构上共享内存的错觉)
  - [调用堆栈的错觉](actors-motivation.md#调用堆栈的错觉)
- [Actor 模型如何满足现代分布式系统的需求](actors-intro.md)
  - [使用消息传递避免锁和阻塞](actors-intro.md#使用消息传递避免锁和阻塞)
  - [Actor如何优雅地处理错误情况](actors-intro.md#Actor如何优雅地处理错误情况)
- [Akka 库和模块概述](modules.md)
  - [Actor](modules.md#Actor库)
  - [远程处理](modules.md#Remoting)
  - [簇](modules.md#cluster)
  - [集群分片](modules.md#cluster-sharding)
  - [集群单例](modules.md#cluster-singleton)
  - [持久化](modules.md#persistence)
  - [分支投射](modules.md#projections)
  - [分布式数据](modules.md#distributed-data)
  - [流](modules.md#streams)
  - [阿尔帕卡](modules.md#alpakka)
  - [HTTP](modules.md#http)
  - [gRPC](modules.md#grpc)
  - [模块使用示例](modules.md#模块使用示例)
- [示例介绍](tutorial.md)
  - [先决条件](tutorial.md#先决条件)
  - [物联网示例用例](tutorial.md#IoT示例)
  - [您将在本教程中学到什么](tutorial.md#在本教程中你将学到什么？)
- [第 1 部分：Actor 架构](tutorial_1.md)
  - [依赖](tutorial_1.md#依赖)
  - [简介](tutorial_1.md#简介)
  - [Akka actor 层次结构](tutorial_1.md#Akka的Actor层级)
  - [总结](tutorial_1.md#总结)
- [第 2 部分：创建第一个 Actor](tutorial_2.md)
  - [介绍](tutorial_2.md#简介)
  - [下一步是什么？](tutorial_2.md#下一步是什么？)
- [第 3 部分：使用设备 Actor](tutorial_3.md)
  - [介绍](tutorial_3.md#简介)
  - [识别设备消息](tutorial_3.md#识别设备的消息)
  - [增加设备消息的灵活性](tutorial_3.md#增加设备消息的灵活性)
  - [实现设备actor及其读取协议](tutorial_3.md#实现设备Actor及其读取协议)
  - [测试](tutorial_3.md#测试)
  - [添加写协议](tutorial_3.md#添加写入协议)
  - [具有读写消息的Actor](tutorial_3.md#具有读写消息的Actor)
  - [下一步是什么？](tutorial_3.md#下一步是什么？)
- [第 4 部分：使用设备组](tutorial_4.md)
  - [介绍](tutorial_4.md#简介)
  - [设备管理器层次结构](tutorial_4.md#设备管理器层次结构)
  - [注册协议](https://doc-akka-io.translate.goog/docs/akka/current/typed/guide/tutorial_4.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=ajax,elem#the-registration-protocol)
  - [向设备Actor添加注册支持](tutorial_4.md#向设备Actor添加注册支持)
  - [创建设备管理器角色](tutorial_4.md#创建设备管理器角色)
  - [下一步是什么？](tutorial_4.md#下一步是什么？)
- [第 5 部分：查询设备组](tutorial_5.md)
  - [介绍](tutorial_5.md#简介)
  - [处理可能的场景](tutorial_5.md#处理可能的情况)
  - [实现查询](tutorial_5.md#实现查询)
  - [向组添加查询功能](tutorial_5.md#向设备组添加查询功能)
  - [总结](tutorial_5.md#总结)
  - [下一步是什么？](tutorial_5.md#下一步是什么？)