# Kafka
[toc]

# Kafka中的重要基本概念
1. Topic：主题，对信息源的分类。
2. Partition：分区，Topic在物理上的分区。每个Partition都是一个有序的队列。Partition中的每条消息都会被分配一个有序的id(offset)。
3. Message：消息，消息队列传递的基本单位。
4. Producer：消息和数据的生产者。
5. Consumer：消息和数据消费者。
6. Broker：缓冲代理，Kafka集群中的一台或多态服务器统称为broker。
7. Controller：在Kafka集群中，通过Zookeeper选举某个Broker作为Controller，来进行leader election以及各种fail-over。
8. Zookeeper：Kafka通过Zookeeper来存储集群的Topic、Partition等元信息等。

# Kafka是什么？
Kafka是一个消息队列，可以实现**发布订阅模式**，在**异步通信**或者**生产者和消费者需要解耦**的场景中经常使用，可以对数据流进行处理等。

## Kafka的主要特点
1. 同时为发布和订阅提供高吞吐量。
2. 可进行持久化操作。
3. 分布式系统，易于向外扩展。
4. 消息被处理的状态实在Consumer端维护，而不是由Broker端维护。当失败时，能够自平衡。
5. 支持online和offline的场景。

## Kafka的特性
- 支持消息的**快速**持久化。
- 支持**批量**读写信息。
- 支持信息**分区**，并且支持**在线增加分区**，提高了并发能力。
- 支持为每个分区创建多个副本。

## Kafka的设计要点
1. 吞吐量
	- 数据磁盘持久化：数据直接**顺序写入**磁盘。
	- 零拷贝：减少IO操作。
	- 数据批量发送：每次发送一个batch。
	- 数据压缩。
	- 每个Topic中可以有多个分区。
2. 负载均衡
	- Producer根据指定的算法，将消息发送到指定的Partition中。
	- 每个Partition在不同的broker上都有replica。多个Partition需要选取出Leader partition，Leader partition负责读写，并由Zookeeper负责fail over。
3. 拉取系统
	- 简化Kafka设计。
	- Consumer根据消费能力自主控制消息拉取速度。
	- Consumer根据自身情况自主选择消费模式。
4. 可扩展性
	- 通过Zookeeper管理Broker与Consumer的动态加入与离开。

# Kafka为什么快？
## 顺序IO
Kafka经常被用来最为大数据系统的一个组件，这就意味着它需要跟大量的数据打交道，使得它只能将数据存储在磁盘而不是内存中。而对磁盘而言，顺序IO的速度远远快于随机IO的速度。
## 内存映射文件
Kafka使用内存映射文件将内存与磁盘关联起来。存储数据的时候，Kafka并不是立马将数据写入磁盘，而是先将数据写入内存，稍后再将内存上的数据冲刷到磁盘上。
## 零拷贝
非零拷贝：
1. 操作系统将数据从磁盘读入到**内核空间**的读缓冲区。
2. 应用程序从内核空间的读缓冲区将数据拷贝到用户空间的缓冲区中。
3. 应用程序将数据从用户空间的缓冲区再写回到内核空间的socket缓冲区中。
4. 操作系统将socket缓冲区中的数据拷贝到NIC缓冲区中，然后通过网络发送给客户端。

零拷贝：
直接将数据从内核空间的读缓冲区拷贝到内核空间的socket缓冲区，然后再写入到NIC缓冲区，避免了再内核空间和用户空间之间穿梭。

# Kafka中的核心概念
- 生产者(Producer)：生产信息，并且按照一定的规则(分区分配规则)推送到Topic的分区中。
- 消费者(Consumer)：从Topic中拉取消息并且进行消费，消费者自行维护消费信息的位置(offset)。
- 主题(Topic)：存储消息的逻辑概念，是一个消息集合，Kafka根据Topic对消息进行归类，发布到Kafka集群的每条消息都需要指定一个Topic。
- 分区(Partition)：每个Topic可以划分为多个分区，每个消息在分区中都会有一个唯一编号offset，Kafka通过offset保证消息在分区中的顺序，同一个Topic的不同分区可以分配在不同的Broker上，Partition以文件的形式存储在文件系统中。
- 副本(Replica)：Kafka对消息进行了**冗余备份**，每个分区有多个副本，每个副本中包含的消息是“一样”的。每个副本中都会**选举出一个Leader副本**，其余为Follower副本，Follower副本仅仅是将数据从Leader副本拉到本地，然后同步到自己的Log中。
- 消费者组(Consumer Group)：每个Consumer都属于一个Consumer Group，每条信息只能被Consumer Group中的一个Consumer消费，但可以被多个Consumer Group消费。
- Broker：一个单独的Server就是一个Broker，主要用来接收生产者发过来的消息，分配Offerset，并且保存在磁盘中。
- Cluster & Controller：多个Broker可以组成一个Cluster集群，每个集群选举一个Broker来作为Controller，充当指挥中心。Controller负责管理分区的状态，管理每个分区的副本状态，监听ZooKeeper中数据的变化等工作。
- 日志压缩与保留策略：不管消费者是否已经消费了信息，Kafka都会保存这些信息，通过配置相应的保留策略，定时删除陈旧的消息。所谓日志压缩，就是定时进行相同Key值的合并，只保留最新的Key-Value值。

# Kafka的副本机制
## 同步复制
当所有的Follower副本都将消息复制完成，这条消息才会被认为是提交完成，一旦有一个Follower副本出现故障，就会导致信息无法提交，极大的影响到了系统的性能。
## 异步复制
当Leader副本接收到生产者发送的消息后就认为当前信息提交成功。Follower副本异步的从Leader副本同步信息，但是不可以保证同步速度，当Leader副本突然宕机的时候，可能Follower副本中的消息落后太多，导致消息的丢失。
## ISR(In-Sync-Replica)集合
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

# Topic和Partition
Kafka中的一个Topic可以认为是一类信息，每个Topic被分成多个Partition，每个Partition在存储层面时Append Log文件。发布到此Partition的消息都会被追加到Log文件的尾部，每条消息在文件中的位置称为offset(偏移量)，offset为一个Long型的数字，它唯一标记一条消息。每条消息都被append到Partition中，是一种顺序写磁盘的方法。

**分区的水平扩展**
每一条消息被发送到Broker中，会根据Partition规则选择被存储到哪一个Partition。如果Partition规则设置的合理，所有消息可以均匀分布到不用的Partition里。
**为什么Kafka中的分区只支持增长，不支持减小分区个数的操作？**
- 删除掉的分区的消息不好处理，若丢弃则可靠性得不到保证。
- 如果插入现有分区的尾部，则一些带时间戳的消息会对消费者有影响。
- 如果消息量大的话，复制到其它分区也会很耗费资源。

# Kafka的网络模型
Kafka的网络框架是基于Java NIO封装的。
1. KafkaClient，单线程Selector模型。
2. KafkaServer，即Kafka Broker