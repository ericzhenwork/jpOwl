<h2>项目源码将于2019年9月开源公布</font></h2>

## 设计架构
jpOwl客户端是java语言编写而成，要求做到API简单、高可靠性能、无论在任何场景下客户端都不能影响各业务服务的性能。旨在为各业务线提供丰富的埋点功能与数据采集。

在收集数据方面使用ThreadLocal，为每一个使用该变量的线程都提供一个变量值的副本，是Java中一种较为特殊的线程绑定机制，是每一个线程都可以独立地改变自己的副本，而不会和其它线程的副本冲突。

![](https://user-gold-cdn.xitu.io/2019/7/24/16c233271510a63f?w=1428&h=784&f=png&s=466162)
如图，执行业务逻辑的时候，就会把此次请求对应的监控存放于ThreadContext中，ThreadContext其实是一个监控树的结构。最后业务线程执行结束时，将监控对象**异步**存入一个内存队列中，jpOwl有个消费线程将队列内的数据**异步**发送到第三方存储引擎。
## 价值与优势
**价值**
* 减少故障发现时间
* 降低故障定位成本
* 辅助应用程序优化
* 监控数据是全量统计，jpOwl支持预计算数据
* 链路数据是采样计算

**优势**
* 实时处理：信息的价值会随时间锐减，所以收集信息需要迅速
* 全量数据：全量采集指标数据，便于深度分析故障案例
* 故障容忍：故障不影响业务正常运转、对业务透明
* 高吞吐：海量监控数据的收集，需要高吞吐能力做保证

## 业务模型监控
jpOwl主要支持以下四种监控模型：

* **Transaction**	适合记录跨越系统边界的程序访问行为,比如远程调用，数据库调用，也适合执行时间较长的业务逻辑监控，Transaction用来记录一段代码的执行时间和次数。
* **Event**	用来记录一件事发生的次数，比如记录系统异常，它和transaction相比缺少了时间的统计，开销比transaction要小。
* **Heartbeat**	表示程序内定期产生的统计信息, 如CPU利用率, 内存利用率, 连接池状态, 系统负载等。
* **Metric** 用于记录业务指标、指标可能包含对一个指标记录次数、记录平均值、记录总和，业务指标最低统计粒度为1分钟。

### 消息树
jpOwl监控系统将每次URL、Service的请求内部执行情况都封装为一个完整的消息树、消息树可能包括`Transaction`、`Event`、`Heartbeat`、`Metric`等信息。
**完整的消息树**
![](https://user-gold-cdn.xitu.io/2019/7/24/16c233f58bf00c91?w=1047&h=333&f=png&s=248125)
**可视化消息树**
![](https://user-gold-cdn.xitu.io/2019/7/24/16c233f3411080ce?w=976&h=469&f=png&s=560622)
**分布式消息树【一台机器调用另外一台机器】**
![](https://user-gold-cdn.xitu.io/2019/7/24/16c234002943d50e?w=769&h=609&f=png&s=447817)
## API功能设计
* 可记录一段代码的URL执行耗时，也可以是SQL的执行耗时
* 可记录一段代码的抛出异常记录次数，或者一段逻辑的执行次数
* 定期执行上报一些核心指标，jvm内存、gc等指标
* 业务指标监控，比如监控订单数、交易额、支付成功率等
* 声明式或者编程式的监控注入
* 可随意指定第三方数据存储引擎
* 支持日志级别的自动升降级
* 可动态指定监控日志前缀，用于分类

## 性能设计
序列化和通信是整个客户端包括服务端性能里面很关键的一个环节
* 异步序列化，jpOwl序列化协议protobuf序列化协议
* 异步通信，jpOwl通信是基于Netty来实现的NIO的数据传输
* 异步化IO的操作
* 异步化传输
* 异步数据采集，基于NIO管道记录日志
* 基于内存级别的数据缓冲
