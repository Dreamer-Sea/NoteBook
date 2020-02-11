# 消息队列 - Kafka
[toc]
## 消息队列Kafka是什么？
Kafka是一个消息队列，可以实现**发布订阅模式**，在**异步通信**或者**生产者和消费者需要解耦**的场景中经常使用，可以对数据流进行处理等。
### Kafka的特性
- 支持消息的**快速**持久化。
- 支持**批量**读写信息。
- 支持信息**分区**，并且支持**在线增加分区**，提高了并发能力。
- 支持为每个分区创建多个副本。
### Kafka可以实现消息的快速持久化的原因
- Kafka将信息保存在磁盘中，并且读写磁盘的方式时**顺序读写**，避免了随机读写磁盘(寻道时间过长)导致的性能瓶颈。
- 磁盘的顺序读写速度超过内存随机读写。
### 为什么具有高性能的特点
- 顺序读写磁盘：磁盘的顺序读写速度超过内存随机读写。
- 页缓存：一种主要的磁盘缓存，以此用来减少对磁盘I/O的操作。
- 零拷贝(Zero-Copy)：将数据直接从磁盘文件复制到网卡设备，而不需要经由应用程序。减少了内核和用户模式之间的上下文切换。

## Kafka中的核心概念
- 生产者(Producer)：生产信息，并且按照一定的规则(分区分配规则)推送到Topic的分区中。
- 消费者(Consumer)：从Topic中拉取消息并且进行消费，消费者自行维护消费信息的位置(offset)。
- 主题(Topic)：存储消息的逻辑概念，是一个消息集合，Kafka根据Topic对消息进行归类，发布到Kafka集群的每条消息都需要指定一个Topic。
- 分区(Partition)：每个Topic可以划分为多个分区，每个消息在分区中都会有一个唯一编号offset，Kafka通过offset保证消息在分区中的顺序，同一个Topic的不同分区可以分配在不同的Broker上，Partition以文件的形式存储在文件系统中。
- 副本(Replica)：Kafka对消息进行了**冗余备份**，每个分区有多个副本，每个副本中包含的消息是“一样”的。每个副本中都会**选举出一个Leader副本**，其余为Follower副本，Follower副本仅仅是将数据从Leader副本拉到本地，然后同步到自己的Log中。
- 消费者组(Consumer Group)：每个Consumer都属于一个Consumer Group，每条信息只能被Consumer Group中的一个Consumer消费，但可以被多个Consumer Group消费。
- Broker：一个单独的Server就是一个Broker，主要用来接收生产者发过来的消息，分配Offerset，并且保存在磁盘中。
- Cluster & Controller：多个Broker可以组成一个Cluster集群，每个集群选举一个Broker来作为Controller，充当指挥中心。Controller负责管理分区的状态，管理每个分区的副本状态，监听ZooKeeper中数据的变化等工作。
- 日志压缩与保留策略：不管消费者是否已经消费了信息，Kafka都会保存这些信息，通过配置相应的保留策略，定时删除陈旧的消息。所谓日志压缩，就是定时进行相同Key值的合并，只保留最新的Key-Value值。

## Kafka的副本机制
### 同步复制
当所有的Follower副本都将消息复制完成，这条消息才会被认为是提交完成，一旦有一个Follower副本出现故障，就会导致信息无法提交，极大的影响到了系统的性能。
### 异步复制
当Leader副本接收到生产者发送的消息后就认为当前信息提交成功。Follower副本异步的从Leader副本同步信息，但是不可以保证同步速度，当Leader副本突然宕机的时候，可能Follower副本中的消息落后太多，导致消息的丢失。
### ISR(In-Sync-Replica)集合
可用副本集合，ISR集合表示**当前“可用”**且消息量与Leader相差不多的副本集合，需要满足如下条件：
- 副本所在节点必须维持着与ZooKeeper的连接。
- 副本最后一条消息的offset与Leader副本的最后一条消息的offset之间的差值不能超过指定的阈值。

**HW和LEO标志**
- HW(HighWatermark)表示高水位，标记了一个特殊的offset，当消费者处理消息的时候，只能拉取到HW之前的消息。HW也是由Leader副本管理的。
- LEO(Log End Offset)是所有副本都会有的一个offset标记。

**ISR、HW和LEO的工作配合机制**
- Producer向此分区中推送消息。
- Leader副本将消息追加到Log中，并且递增其LEO。
- Follower副本从Leader副本中拉取消息进行同步。
- Follower副本将消息更新到本地Log中，并且递增其LEO。
- 当ISR集合中的所有副本都完成了对offset的消息同步，Leader副本就会递增其HW。

**优势**
- 同步复制会导致**高延迟**，异步复制可能会**造成消息的丢失**，ISR解决了这两个缺点。
- 当Follower副本**延迟过高**时，就会被踢出ISR集合，避免了高延迟的Follower副本影响整个Kafka集群性能。
- 当Leader副本所在的Broker宕机，会优先将ISR集合中的Follower副本选举为Leader。

## Topic和Partition
Kafka中的一个Topic可以认为是一类信息，每个Topic被分成多个Partition，每个Partition在存储层面时Append Log文件。发布到此Partition的消息都会被追加到Log文件的尾部，每条消息在文件中的位置称为offset(偏移量)，offset为一个Long型的数字，它唯一标记一条消息。每条消息都被append到Partition中，是一种顺序写磁盘的方法。

**分区的水平扩展**
每一条消息被发送到Broker中，会根据Partition规则选择被存储到哪一个Partition。如果Partition规则设置的合理，所有消息可以均匀分布到不用的Partition里。
**为什么Kafka中的分区只支持增长，不支持减小分区个数的操作？**
- 删除掉的分区的消息不好处理，若丢弃则可靠性得不到保证。
- 如果插入现有分区的尾部，则一些带时间戳的消息会对消费者有影响。
- 如果消息量大的话，复制到其它分区也会很耗费资源。