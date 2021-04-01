
# 分布式事务产生的背景
## 分布式架构演进之 - 数据库的水平拆分
蚂蚁金服的业务数据库起初是单库单表，但随着业务数据规模的快速发展，数据量越来越大，单库单表逐渐成为瓶颈。所以我们对数据库进行了水平拆分，将原单库单表拆分成数据库分片。
如下图所示，分库分表之后，原来在一个数据库上就能完成的写操作，可能就会跨多个数据库，这就产生了跨数据库事务问题。
![](http://showdoc.chinahrt.com/server/index.php?s=/api/attachment/visitFile/sign/3111d8ddebcb167a83a6b59f08eca8e9&showdoc=.jpg)

## 分布式架构演进之 - 业务服务化拆分
在业务发展初期，“一块大饼”的单业务系统架构，能满足基本的业务需求。但是随着业务的快速发展，系统的访问量和业务复杂程度都在快速增长，单系统架构逐渐成为业务发展瓶颈，解决业务系统的高耦合、可伸缩问题的需求越来越强烈。

如下图所示，蚂蚁金服按照面向服务架构（SOA）的设计原则，将单业务系统拆分成多个业务系统，降低了各系统之间的耦合度，使不同的业务系统专注于自身业务，更有利于业务的发展和系统容量的伸缩。业务系统按照服务拆分之后，一个完整的业务往往需要调用多个服务，如何保证多个服务间的数据一致性成为一个难题。
![](http://showdoc.chinahrt.com/server/index.php?s=/api/attachment/visitFile/sign/d8d059ad998bb22f7e020bea89551b28&showdoc=.jpg)

# 分布式事务理论基础
###  两阶段提交协议
   ![](http://showdoc.chinahrt.com/server/index.php?s=/api/attachment/visitFile/sign/e59aeca55a61622580075fc828e11b18&showdoc=.jpg)
   事务管理器分两个阶段来协调资源管理器，第一阶段准备资源，也就是预留事务所需的资源，如果每个资源管理器都资源预留成功，则进行第二阶段资源提交，否则协调资源管理器回滚资源。
###   TCC
  ![](http://showdoc.chinahrt.com/server/index.php?s=/api/attachment/visitFile/sign/3fb89590e39d79a3ade7b70a7a35985f&showdoc=.jpg)
   TCC（Try-Confirm-Cancel） 实际上是服务化的两阶段提交协议，业务开发者需要实现这三个服务接口，第一阶段服务由业务代码编排来调用 Try 接口进行资源预留，所有参与者的 Try 接口都成功了，事务管理器会提交事务，并调用每个参与者的 Confirm 接口真正提交业务操作，否则调用每个参与者的 Cancel 接口回滚事务。
###   Saga
Saga 是一种补偿协议，在 Saga 模式下，分布式事务内有多个参与者，每一个参与者都是一个冲正补偿服务，需要用户根据业务场景实现其正向操作和逆向回滚操作。
分布式事务执行过程中，依次执行各参与者的正向操作，如果所有正向操作均执行成功，那么分布式事务提交。如果任何一个正向操作执行失败，那么分布式事务会退回去执行前面各参与者的逆向回滚操作，回滚已提交的参与者，使分布式事务回到初始状态。
Saga 理论出自 Hector & Kenneth 1987发表的论文 Sagas。
Saga 正向服务与补偿服务也需要业务开发者实现。
![](http://showdoc.chinahrt.com/server/index.php?s=/api/attachment/visitFile/sign/c8b8221b125c2f2bddab71410034b235&showdoc=.jpg)



------------

------------

------------




为了便于大家之间参考基于Seata集成测试已经相关文档的快速查看,内容摘要及相关项目的集成,开发,测试将会在该文档中体现.
关于更详细的信息,大家可以参考http://seata.io/zh-cn/docs/overview/what-is-seata.html 官方文档进行查阅.
# Seata 是什么?

- Seata 是一款开源的分布式事务解决方案，致力于在微服务架构下提供高性能和简单易用的分布式事务服务。在 Seata 开源之前，Seata 对应的内部版本在阿里经济体内部一直扮演着分布式一致性中间件的角色，帮助经济体平稳的度过历年的双11，对各BU业务进行了有力的支撑。经过多年沉淀与积累，商业化产品先后在阿里云、金融云进行售卖。2019.1 为了打造更加完善的技术生态和普惠技术成果，Seata 正式宣布对外开源，未来 Seata 将以社区共建的形式帮助其技术更加可靠与完备。

![](http://showdoc.chinahrt.com/server/index.php?s=/api/attachment/visitFile/sign/d696739e828a6024dbc5b952e1c06809&showdoc=.jpg)

Seata 是一款开源的分布式事务解决方案，致力于提供高性能和简单易用的分布式事务服务。Seata 将为用户提供了 AT、TCC、SAGA 和 XA 事务模式，为用户打造一站式的分布式解决方案。(阿里开源项目).

Seata 会有 4 种分布式事务解决方案，分别是 AT 模式、TCC 模式、Saga 模式和 XA 模式。
![](http://showdoc.chinahrt.com/server/index.php?s=/api/attachment/visitFile/sign/f1b44169fa4d0fd8bfee955864777c31&showdoc=.jpg)

![](http://showdoc.chinahrt.com/server/index.php?s=/api/attachment/visitFile/sign/3df1ebad96ca57da12ea47fa5f8cc7b0&showdoc=.jpg)

# Seata术语
### TC (Transaction Coordinator) - 事务协调者
维护全局和分支事务的状态，驱动全局事务提交或回滚。

### TM (Transaction Manager) - 事务管理器
定义全局事务的范围：开始全局事务、提交或回滚全局事务。

### RM (Resource Manager) - 资源管理器
管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。

![](http://showdoc.chinahrt.com/server/index.php?s=/api/attachment/visitFile/sign/b1f090009d953e23303867eb27591b3d&showdoc=.jpg)

Seata管理的分布式事务的典型生命周期：

1. TM要求TC开始新的全球交易。TC生成代表全局交易的XID。
2. XID通过微服务的调用链传播。
3. RM将本地事务注册为XID到TC的相应全局事务的分支。
4. TM要求TC提交或回退相应的XID全局事务。
5. TC驱动XID的相应全局事务下的所有分支事务，以完成分支提交或回滚。

# AT 模式
  2019年 1 月份，Seata 开源了 AT 模式。AT 模式是一种无侵入的分布式事务解决方案。在 AT 模式下，用户只需关注自己的“业务 SQL”，用户的 “业务 SQL” 作为一阶段，Seata 框架会自动生成事务的二阶段提交和回滚操作。
  

## 前提 
  **基于支持本地 ACID 事务的关系型数据库。**
  **Java 应用，通过 JDBC 访问数据库。**
##  整体机制
两阶段提交协议的演变：

  一阶段：业务数据和回滚日志记录在同一个本地事务中提交，释放本地锁和连接资源。

  二阶段：

- 提交异步化，非常快速地完成。
- 回滚通过一阶段的回滚日志进行反向补偿。
![](http://showdoc.chinahrt.com/server/index.php?s=/api/attachment/visitFile/sign/4b55fd14fc70d8c3f37f76ff33e52e81&showdoc=.jpg)
##  写隔离
- 一阶段本地事务提交前，需要确保先拿到 **全局锁** 。
- 拿不到 **全局锁** ，不能提交本地事务。
- 拿 **全局锁** 的尝试被限制在一定范围内，超出范围将放弃，并回滚本地事务，释放本地锁。

#####     以一个示例来说明：
两个全局事务 tx1 和 tx2，分别对 a 表的 m 字段进行更新操作，m 的初始值 1000。

tx1 先开始，开启本地事务，拿到本地锁，更新操作 m = 1000 - 100 = 900。本地事务提交前，先拿到该记录的 **全局锁** ，本地提交释放**本地锁**。 tx2 后开始，开启本地事务，拿到**本地锁**，更新操作 m = 900 - 100 = 800。本地事务提交前，尝试拿该记录的**全局锁** ，tx1 全局提交前，该记录的全局锁被 tx1 持有，tx2 需要重试等待 全局锁 。
   
   
![](http://showdoc.chinahrt.com/server/index.php?s=/api/attachment/visitFile/sign/3b2e4b4fa0f7c4915008de89dc6792ab&showdoc=.jpg)


tx1 二阶段全局提交，释放 全局锁 。tx2 拿到 全局锁 提交本地事务。

![](http://showdoc.chinahrt.com/server/index.php?s=/api/attachment/visitFile/sign/3e8aa53ea7738149fae859fc107ec541&showdoc=.jpg)

## 读隔离
 在数据库本地事务隔离级别 读已提交（Read Committed） 或以上的基础上，Seata（AT 模式）的默认全局隔离级别是 读未提交（Read Uncommitted） 。

如果应用在特定场景下，必需要求全局的 读已提交 ，目前 Seata 的方式是通过 SELECT FOR UPDATE 语句的代理。
![](http://showdoc.chinahrt.com/server/index.php?s=/api/attachment/visitFile/sign/b95e50f9fd44bda68c25e917da031067&showdoc=.jpg)
SELECT FOR UPDATE 语句的执行会申请 全局锁 ，如果 全局锁 被其他事务持有，则释放本地锁（回滚 SELECT FOR UPDATE 语句的本地执行）并重试。这个过程中，查询是被 block 住的，直到 全局锁 拿到，即读取的相关数据是 已提交 的，才返回。

出于总体性能上的考虑，Seata 目前的方案并没有对所有 SELECT 语句都进行代理，仅针对 FOR UPDATE 的 SELECT 语句。


