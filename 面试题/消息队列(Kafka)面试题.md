# 消息队列(Kafka)面试题

[toc]

# Kafka如何实现消息不重复消费(有且仅被消费一次)？
Kafka在consumer消费了一个信息之后就会将offset给提交(提交到ZooKeeper中)，这样就能够避免正常情况下的消息重复消费问题。
但是如果Kafka如果在offset提交前就宕机的话，那么offset就没被提交。那么这时候就需要用其它方法来保证消息的幂等性。在Kafka中是通过给Producer加入ID(即**PID**)，然后每条消息都加上Sequence Number(简称**Seq**)，当broker发现传送过来的消息的Seq小于自身记录的，那就将该消息丢弃。

1. 幂等producer：保证单个分区只会发送一次。
2. 事务：保证原子性的写入多个分区。
3. 流式EOS：流处理(读取-处理-写入)。

# Kafka如何实现消息的不丢失
1. ACK机制：
	- ack = 0：不关心是否投递成功。
	- ack = 1：收到broker发送的确认消息即可。
	- ack = all：必须要有一半节点确认才可以。
2. 生产者(消息发送端)：生产者能够异步或者同步发送消息，如果条件允许可以使用同步发送，避免消息缓存带来的丢失。
3. 消费者：消息一旦被消费就提交offset。

# Kafka为什么快？
1. 零拷贝。
2. 顺序IO。
3. 内存映射文件。

# Kafka如何保证消息的有序
1. topic中只有一个partition。
2. producer将消息发送到同一个partition(指定partition)。

# ZooKeeper在Kafka中的作用
1. Kafka的正常运行少不了ZooKeeper，一些关键模块依赖于ZooKeeper。
2. ZooKeeper存储原信息：consumerGroup/consumer、broker、Topic等。
3. 在旧版本中，consumer和broker依赖于ZooKeeper，新版本中就broker依赖于ZooKeeper。