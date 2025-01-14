# Distributed System

随着大型网站的各种高并发访问、海量数据处理等场景越来越多，如何实现网站的高可用、易伸缩、可扩展、安全等目标就显得越来越重要。为了解决这样一系列问题，大型网站的架构也在不断发展。提高大型网站的高可用架构，不得不提的就是分布式。本文主要简单介绍了分布式系统的概念、分布式系统的特点、常用的分布式方案以及分布式和集群的区别等。

分布与并发一直是高度重叠又有一定差异的概念，笔者理解的所谓分布式，并不是一定要在多台物理设备或者虚拟设备上实现的系统，见微知著，即使是由多个进程或者多个线程组成的也可以看做一个分布式系统。在这种思量之下，可以把并发领域的一系列待解决的问题归拢到整个分布式体系之内。不过这边还有一个小的矛盾点，并发并不一定是多进程 / 多线程的，譬如 IO 多路复用这一解决方案，毕竟并发这个概念本身只是为了应对而不是动手做。不过无论它底层是如何实现的，只要对上层提供了同时处理多个请求的能力，那也可以看过一个分布式系统。基于以上理解，笔者在这里描述的分布式系统即是能够由逻辑上多个线性处理器构成的能够同时响应多个请求与进行多个操作的系统。

> 此种分类方式全凭主观，没有啥科学有效的分类依据，只是为了方便排布笔记

分布式(distributed )是指在多台不同的服务器中部署不同的服务模块，通过远程调用协同工作，对外提供服务。集群(cluster )是指在多台不同的服务器中部署相同应用或服务模块，构成一个集群，通过负载均衡设备对外提供服务。笔者觉得，集群是分布式的一个重要组成部分。

Consistency: 一致性 ConsensusProtocol: 共识协议

# 分布式系统定义

## 集中式系统

集中式系统用一句话概括就是：一个主机带多个终端。终端没有数据处理能力，仅负责数据的录入和输出。而运算、存储等全部在主机上进行。现在的银行系统，大部分都是这种集中式的系统，此外，在大型企业、科研单位、军队、政府等也有分布。集中式系统，主要流行于上个世纪。集中式系统的最大的特点就是部署结构非常简单，底层一般采用从 IBM、HP 等厂商购买到的昂贵的大型主机。因此无需考虑如何对服务进行多节点的部署，也就不用考虑各节点之间的分布式协作问题。但是，由于采用单机部署。很可能带来系统大而复杂、难于维护、发生单点故障(单个点发生故障的时候会波及到整个系统或者网络，从而导致整个系统或者网络的瘫痪)、扩展性差等问题。

## 分布式系统定义

在《分布式系统概念与设计》一书中，对分布式系统做了如下定义：

> 分布式系统是一个硬件或软件组件分布在不同的网络计算机上，彼此之间仅仅通过消息传递进行通信和协调的系统

简单来说就是一群独立计算机集合共同对外提供服务，但是对于系统的用户来说，就像是一台计算机在提供服务一样。分布式意味着可以采用更多的普通计算机(相对于昂贵的大型机)组成分布式集群对外提供服务。计算机越多，CPU 、内存、存储资源等也就越多，能够处理的并发访问量也就越大。

从分布式系统的概念中我们知道，各个主机之间通信和协调主要通过网络进行，所以，分布式系统中的计算机在空间上几乎没有任何限制，这些计算机可能被放在不同的机柜上，也可能被部署在不同的机房中，还可能在不同的城市中，对于大型的网站甚至可能分布在不同的国家和地区。但是，无论空间上如何分布，一个标准的分布式系统应该具有以下几个主要特征：

#### 分布性

> 分布式系统中的多台计算机之间在空间位置上可以随意分布，系统中的多台计算机之间没有主、从之分，即没有控制整个系统的主机，也没有受控的从机。

#### 透明性

> 系统资源被所有计算机共享。每台计算机的用户不仅可以使用本机的资源，还可以使用本分布式系统中其他计算机的资源 ( 包括 CPU、文件、打印机等 )。

#### 同一性

> 系统中的若干台计算机可以互相协作来完成一个共同的任务，或者说一个程序可以分布在几台计算机上并行地运行。

#### 通信性

> 系统中任意两台计算机都可以通过通信来交换信息。

和集中式系统相比，分布式系统的性价比更高、处理能力更强、可靠性更高、也有很好的扩展性。但是，分布式在解决了网站的高并发问题的同时也带来了一些其他问题。首先，分布式的必要条件就是网络，这可能对性能甚至服务能力造成一定的影响。其次，一个集群中的服务器数量越多，服务器宕机的概率也就越大。另外，由于服务在集群中分布是部署，用户的请求只会落到其中一台机器上，所以，一旦处理不好就很容易产生数据一致性问题。

以请求 - 响应的方式对服务进行解耦，通常使用 REST 的服务实现。它很适合于 UI 以及提问的场景。事件驱动，这种模式的特征是异步的，或是 “fire and forget” 的消息传递。它非常适合于设计跨服务的复杂依赖。这两种模式可以结合使用，例如使用请求 - 响应模式实现一个 REST 接口，随后以事件的方式进行后台处理。

# 常用分布式方案

#### 分布式应用和服务

> 将应用和服务进行分层和分割，然后将应用和服务模块进行分布式部署。这样做不仅可以提高并发访问能力、减少数据库连接和资源消耗，还能使不同应用复用共同的服务，使业务易于扩展。

#### 分布式静态资源

> 对网站的静态资源如 JS、CSS 、图片等资源进行分布式部署可以减轻应用服务器的负载压力，提高访问速度。

#### 分布式数据和存储

> 大型网站常常需要处理海量数据，单台计算机往往无法提供足够的内存空间，可以对这些数据进行分布式存储。

#### 分布式计算

> 随着计算技术的发展，有些应用需要非常巨大的计算能力才能完成，如果采用集中式计算，需要耗费相当长的时间来完成。分布式计算将该应用分解成许多小的部分，分配给多台计算机进行处理。这样可以节约整体计算时间，大大提高计算效率。

# OLTP&OLAP

目前分布式数据库主要分为两种场景 ——OLTP(联机事务处理)和 OLAP(联机分析处理)。随着大数据技术发展，数据库选择越来越多，其主要差别包括：面向事务还是面向分析；数据内容是当前的、详细的数据还是历史的、汇总的数据；数据库设计是实体联系模型 ER 和面向应用的数据库设计，还是星型、雪花模型和面向主题的数据库设计等。前者指的是 OLTP 场景，后者指的是 OLAP 场景。

**表 1 OLTP 和 OLAP 对比**

![](http://mmbiz.qpic.cn/mmbiz/Pn4Sm0RsAujF1Uh53H2CzRNHKIzAkSZbRBpKL8rcgq5CPia1egwsKVgWV8ORz7H8oLt4De7CPUWOSYZ89bjLv3w/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1)

基于分布式数据库的理论，不管是数据库的优化还是设计、使用，遇到的问题非常多。举例说，现在硬件发展很好，特别 SSD，如果其设备性能远远没有达到，那么使用 SSD 的数据库性能该如何提高。如果只是为了满足业务当前的简单需求，可以把现在很多数据库的传输引擎存储直接换成 SSD，可以快速地解决很大的问题。另外还有一个很经典的问题，怎么保证在高可靠的前提下提高数据库插入和查询性能。刚才说的是单机模式，多机的分布式模式下又该怎么提高数据调用性能，也有许多挑战。总之，一定要根据业务的需求来选择最合适自己的数据库系统。

多关注数据库的三大顶级会议 , 还有 OSDI 等 . 将 motivation 和 workload 作为最根本的方法论。彻底搞明白存储精髓 WAL+RSM. 好好阅读经典论文 , 搞清楚 workload 和 motivation. 牢记两个基本观点 : 没有抽象的技术 , 凡技术都有自己适用的应用场景 . 没有广谱的系统 , 凡系统都有自己典型的 workload. 建议看先阅读下面两本书 : 1. Stonebraker 的 Architecture of a database system. 2. Jim Gray 的 Transaction. 学习分布式的 Roadmap 1. WAL + RSM 2. 研究分布式共识算法 , 因为这个是日志复制协议的基石 : TOB 协议 , Quorum-based 协议 (mpaxos, raft), Primary-backup 协议 .3. partitioning & replication + local storage engine. 4. local storage engine: bdb, innodb, leveldb( 可以参考我写的源码阅读 ). 5. 磁盘和网络优化技术 . 6. 分布式一致性 ( 隔离性 ). 7. 分布式事务 : stm, mvcc, 乐观锁 . 8. 分布式查询优化 . 工程实现超高的编程技巧 1. 能够像 haskell 那样 impure 和 pure 分离 .2. 幂等的设计 , 让部分有状态的模块成为 metal unit, 同时幂等设计和 failstop 能够将有效地避 bf. 3. 元编程能力 , 降低代码的冗余和耦合 , 代码更适合扩展和组合 .4. 超高的系统编程能力 .5. 单测 , mock 测试设计能力 .6. 漂亮的日志输出 . 7. 会做性能分析 , 参考大牛博客 .8. 会使用 docker 加速自己的开发效率 . 周边涉及 1. 精通 MySQL sharding, 不知道它的痛，就不知道为什么要坚定不移地搞分布式关系数据库 .2. 精通大数据的 BI 解决方案 , 不知道它的痛，就不知道为什么要搞全新的分布式列式数据库 .3. 区块链，分布式计算引擎也了解一点 .

1. 多线程的最大副作用 : `并发`. 如果多个逻辑控制流在时间上发生了重叠 , 就会产生并发 . 逻辑控制流是指一次程序操作 . 如读取或者更新内存变量的值 .`更新的并发性`: 多线程同时更新内存值而产生的并发 .
2. 分布式一致性
   - 目标 :
     - 增加系统可用性 , 防止因单点故障引起的系统不可用 .
     - 提高系统的整体性能 , 通过负载均衡 , 让分布在不同地方的数据副本都能为用户提供服务 .
   - 缺陷 :
     - 为了解决复制延迟 , 阻塞写入动作直到所有的复制完成 , 会严重影响写入的性能 .
     - 在不影响系统运行性能前提下 , 保证数据一致性是不可能的 .
     - 一致性的级别 :
       - `强一致性`. 最好的用户体验 , 但是会影响性能 .
       - `弱一致性`. 尽可能快地但不保证数据一致状态 . 分为会话一致性和用户一致性 .
       - 最终一致性 . 保证一定时间内 , 达到数据一致状态 .
         - 属于弱一致性的特例 . 是目前最被广泛接收的 .
3. 分布式架构
   - 常见问题
     - 通信异常 , 节点故障 .
     - `网络分区`: 部分节点的网络延迟过大 , 导致只有部分节点间能够正常通信的现象 .
     - 请求与响应的`三态`: 成功 , 失败 , 超时 .
   - ACID.
     - Atomicity: 事务中的所有操作只能全部执行或者全部不执行 .
     - Consistency: 事务的执行不能破坏数据库中数据的完整性和一致性 .
       - 当数据库在一些事务尚未完成时发生故障 , 会导致部分修改写入 , 而处于不一致状态 .
     - Isolation: 并发环境中 , 事物相互隔离 .
       - Read Uncommitted. 允许脏读 .
       - Read Committed. 允许不可重复读 ( 一次事物多次读取相同值时 , 原始读取不可重复 ).
       - Repeatable Read( 锁定行 ). 事务过程中多次读取同一数据 , 其值和开始时是一致的 . 可能有幻影数据 ( 锁定行范围 ).
       - Serializable. 所有事务串行执行 , 不支持并发 .
     - Durability: 事务成功结束后 , 其对数据库的更新要被永久保存下来 .
   - 分布式事务
     - 由多个分布式的操作 ( 子事务 ) 序列组成 , 嵌套型的事务 . 需要兼顾可用性和一致性 .
     - CAP. 最多只能同时满足其中的两项 . 分区容错性是必须的 , 平衡的是 Consistency 和 Availability.
       - Consistency. 数据在多个副本间的一致性 .
       - Availability. 用户的每个请求总能在有限的时间内返回结果 .
       - Partition tolerance. 遇到任何网络分区 ( 子网络 ) 故障时 , 对外提供满足一致性和可用性的服务 . 除非整个网络故障 .
     - BASE 理论 . `既然无法达到强一致性 , 那么采用适当的方式来达到最终一致性 .`Basically Available. 出现不可预知的故障时 , 允许损失部分可用性 . 造成响应时间上的损失 , 功能上的损失 .Soft state. 允许系统中的数据存在中间状态 . 即数据副本间的同步存在延迟 .Eventually Consistent. 它包含五种变种 :Causal consistency. 更新进程完成后通知进程 B, 进程 B 拿到的是更新后的数据 .Read your writes. 进程更新数据后 , 自己总是能读取到更新过的新值 .Session consistency. 在同一会话中实现`read your writes`的一致性 .Monotonic read consistency. 进程读取某数据项后 , 后续的数据访问不能返回旧值 .Monotonic write consistency. 保证来自同一进程的写操作顺序地执行 . 核心 : `牺牲强一致性来获得可用性 .`
4. 一致性协议
   - 跨越多个分布式节点的事务操作 , 需引入协调器 (Coordinator) 来调度 , 节点称为参与者 (Participant).
     - Coordinator 调度 Participan t 的行为 , 并最终决定是否要把事务真正进行提交 .
   - 两阶段提交协议 (2PC, Two-Phase Commit).
     - 关系型数据库的通用做法 , 属于强一致性算法 .
     - 阶段 : 提交事务请求 ( 投票 ); 执行事务提交 ( 执行 ).
       - 采用先尝试后提交的方式 .
     - 缺点 :
       1. 同步阻塞 (participant 会一直等待它人的响应 );
       2. 单点问题 (coordinator);
       3. 数据不一致性 ( 阶段二如果有节点没有收到提交消息时 );
       4. 太过保守 (coordinate 只能依靠超时来处理长时间无响应的 participant).
   - 三阶段提交协议 (3PC)
     - 阶段 :
       - CanCommit, PreCommit, Do Commit.
       - participant 在 PreCommit 阶段执行事务操作 , 并将 Undo 和 Redo 信息记录到事务 log 中 .
     - 缺点 :
       - 进入 Do commit 阶段 , 参与者在等待 ( 协调者的提交 / 中断指令 ) 超时后 , 会继续进行事务提交 . 可能导致数据的不一致性 .
     - 优势 :
       - 降低了参与者的阻塞范围 , 且在单点故障后继续达成一致性 .

# 链接

- https://www.atatech.org/articles/143093
- https://www.atatech.org/articles/140173
