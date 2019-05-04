### 分布式事务

#### 1. CAP理论

- C (Consistency): 一致性： 分布式系统中所有数据备份在同一时刻是否有相同的值
- A (Availability): 可用性： 集群中部分节点故障时，是否还能响应客户端请求
- P (Partition tolerance): 分区容错性：系统能否在一定的时限内达到数据一致性

一个分布式应用最多只能同时满足上述两点, 程序员应当在一致性与可用性之间做出选择

互联网大多数场景都需要牺牲**强一致性**来换取系统的**高可用性**，系统只需要保证**最终可用性**

#### 2. BASE理论
- Basically Available 基本可用，响应事件运行延长，运行返回降级页面
- Soft state 软状态：同一数据的不同副本不要求实时一致
- Eventually consistent 最终一致性： 同一数据不同副本经过一段时间后要求得到一致性

核心思想是，如果做不到强一致性，可以通过适当方式达到最终一致性

#### 3. 分布式事务协议

- XA规范
 
XA是X/Open DTP定义的交易中间件与数据库之间的接口规范，交易中间件用它来通知数据库事务的开始、结束、提交、回滚等。XA接口函数由数据库厂商提供。

WebLogic、WebSphere提供了JTA的实现和支持。Tomcat没有实现，这时需要借助第三方框架Jotm，Automikos等来实现。

- 两阶段提交协议 2PC
   - 第一阶段：准备阶段(投票阶段)
   - 第二阶段：提交阶段(执行阶段)

阶段1中，协调者发起提议，分别询问各参与者是否接受

![](https://github.com/zybotian/distributed-transaction-solutions/blob/master/imgs/2pc-1.png)

阶段2中，协调者根据参与者的反馈提交事务，全部同意则提交，一个不同意则回滚。

![](https://github.com/zybotian/distributed-transaction-solutions/blob/master/imgs/2pc-2.png)

- 2PC存在的问题
   - 同步阻塞：所有参与节点都是阻塞型的
   - 单点故障：协调者发生故障，参与者都会阻塞下去
   - 数据不一致：协调者向参与者发送commit请求时由于网络原因，某些参与者收到了commit请求执行commit，某些参与者没有收到commit请求，数据不一致
   - 事务状态不确定：协调者发送commit之后宕机，通过选举选出新的协调者，但之前事务的状态就无法确定了

- 三阶段提交协议 3PC