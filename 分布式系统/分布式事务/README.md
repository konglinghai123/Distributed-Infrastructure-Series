# 分布式事务

在[关系型数据库导论]()中我们讨论了数据库中著名的 ACID 特性，以及事务与并发控制的相关内容。随着互联网快速发展，微服务，SOA 等服务架构模式正在被大规模的使用，现在分布式系统一般由多个独立的子系统组成，多个子系统通过网络通信互相协作配合完成各个功能。而且这个过程中会涉及到事务的概念，即保证交易系统和支付系统的数据一致性，此处我们称这种跨系统的事务为分布式事务。具体一点而言，分布式事务是指事务的参与者、支持事务的服务器、资源服务器以及事务管理器分别位于不同的分布式系统的不同节点之上。分布式事务处理的关键是必须有一种方法可以知道事务在任何地方所做的所有动作，提交或回滚事务的决定必须产生统一的结果(全部提交或全部回滚)。

有很多用例会跨多个子系统才能完成，比较典型的是电子商务网站的下单支付流程，至少会涉及交易系统和支付系统；在用户下单之后，需要在库存子系统减少库存，然后在订单子系统添加订单。如果多个数据库之间的数据更新没有保证事务，将会导致出现子系统数据不一致，业务出现问题。

# 柔性事务与补偿

分布式事务同样需要满足原子性，一致性，隔离性等特征：

- 事务的原子性：事务操作跨不同节点，当多个节点某一节点操作失败时，需要保证多节点操作的要么不做，要么全做（All or Nothing）的原子性。

- 事务的一致性：当发生网络传输故障或者节点故障，节点间数据复制通道中断，在进行事务操作时需要保证数据一致性，保证事务的任何操作都不会使得数据违反数据库定义的约束、触发器等规则。

- 事务的隔离性：事务隔离性的本质就是如何正确处理多个并发事务的读写冲突和写写冲突，因为在分布式事务控制中，可能会出现提交不同步的现象，这个时候就有可能出现“部分已经提交”的事务。此时并发应用访问数据如果没有加以控制，有可能出现“脏读”问题。

事务是为了保证数据的一致性，在[分布式系统理论]()中我们也讨论了 CAP 理论，对于分布式的应用而言，不可能同时满足 C（一致性），A（可用性），P（分区容错性），由于网络分区是分布式应用的基本要素，因此开发者需要在 C 和 A 上做出平衡。由于 C 和 A 互斥性，其权衡的结果就是 BASE 理论。对于大部分的分布式应用而言，只要数据在规定的时间内达到最终一致性即可。我们可以把符合传统的 ACID 叫做刚性事务，把满足 BASE 理论的最终一致性事务叫做柔性事务。

在电商等互联网场景下，传统的事务在数据库性能和处理能力上都暴露出了瓶颈。在分布式领域基于 CAP 理论以及 BASE 理论，有人就提出了柔性事务的概念。基于 BASE 理论的设计思想，柔性事务下，在不影响系统整体可用性的情况下(Basically Available 基本可用)，允许系统存在数据不一致的中间状态(Soft State 软状态)，在经过数据同步的延时之后，最终数据能够达到一致。并不是完全放弃了 ACID，而是通过放宽一致性要求，借助本地事务来实现最终分布式事务一致性的同时也保证系统的吞吐。

柔性事务往往需要具备以下特性：

- 可见性(对外可查询) ：在分布式事务执行过程中，如果某一个步骤执行出错，就需要明确的知道其他几个操作的处理情况，这就需要其他的服务都能够提供查询接口，保证可以通过查询来判断操作的处理情况。为了保证操作的可查询，需要对于每一个服务的每一次调用都有一个全局唯一的标识，可以是业务单据号（如订单号）、也可以是系统分配的操作流水号（如支付记录流水号）。除此之外，操作的时间信息也要有完整的记录。

- 操作幂等性：幂等性，其实是一个数学概念。幂等函数，或幂等方法，是指可以使用相同参数重复执行，并能获得相同结果的函数。幂等操作的特点是其任意多次执行所产生的影响均与一次执行的影响相同。也就是说，同一个方法，使用同样的参数，调用多次产生的业务结果与调用一次产生的业务结果相同。

之所以需要操作幂等性，是因为为了保证数据的最终一致性，很多事务协议都会有很多重试的操作，如果一个方法不保证幂等，那么将无法被重试。幂等操作的实现方式有多种，如在系统中缓存所有的请求与处理结果、检测到重复操作后，直接返回上一次的处理结果等。

事务补偿符合我们最终一致性的理念。补偿事务不一定会将系统中的数据返回到原始操作开始时其所处的状态。 相反，它补偿操作失败前由已成功完成的步骤所执行的工作。补偿事务中步骤的顺序不一定与原始操作中步骤的顺序完全相反。 例如，一个数据存储可能比另一个数据存储对不一致性更加敏感，因而补偿事务中撤销对此存储的更改的步骤应该会首先发生。对完成操作所需的每个资源采用短期的基于超时的锁并预先获取这些资源，这样有助于增加总体活动成功的可能性。 仅在获取所有资源后才应执行工作。 锁过期之前必须完成所有操作。

# 分布式事务方案

![](https://ww1.sinaimg.cn/large/007rAy9hgy1g29eo85f8sj30u00ek75x.jpg)

![](https://ww1.sinaimg.cn/large/007rAy9hgy1g29eo837q8j30pk0b9wez.jpg)

![](http://dbaplus.cn/uploadfile/2018/0801/20180801105319120.jpg)

- 2PC/3PC：依赖于数据库，能够很好的提供强一致性和强事务性，但相对来说延迟比较高，比较适合传统的单体应用，在同一个方法中存在跨库操作的情况，不适合高并发和高性能要求的场景。

- TCC：适用于执行时间确定且较短，实时性要求高，对数据一致性要求高，比如互联网金融企业最核心的三个服务：交易、支付、账务。

- 消息驱动：都适用于事务中参与方支持操作幂等，对一致性要求不高，业务上能容忍数据不一致到一个人工检查周期，事务涉及的参与方、参与环节较少，业务上有对账/校验系统兜底。

- Saga 事务：由于 Saga 事务不能保证隔离性，需要在业务层控制并发，适合于业务场景事务并发操作同一资源较少的情况。Saga 相比缺少预提交动作，导致补偿动作的实现比较麻烦，例如业务是发送短信，补偿动作则得再发送一次短信说明撤销，用户体验比较差。Saga 事务较适用于补偿动作容易处理的场景。

# 链接

- https://mp.weixin.qq.com/s/vgohXl1LxYk3CyDI8WHxwA
- https://dbaplus.cn/news-159-2152-1.html?utm_source=oschina-app
- [分布式事务笔记](http://www.yangguo.info/2016/05/23/%E5%88%86%E5%B8%83%E5%BC%8F%E4%BA%8B%E5%8A%A1%E7%AC%94%E8%AE%B0/)
- https://www.atatech.org/articles/139627
