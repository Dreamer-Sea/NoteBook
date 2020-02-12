# ZooKeeper

# 什么是ZooKeeper？
这是一个典型的分布式数据一致性解决方案，分布式应用程序可以基于ZooKeeper实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master选举、分布式锁和分布式队列等功能。

# ZooKeeper的重要概念
1. 分布式程序(只要半数以上节点存活，ZooKeeper就能正常服务)。
2. 为了保证高可用，最好是以集群形态来部署ZooKeeper。
3. 数据存储在内存中，保证了高吞吐和低延迟。
4. 高性能的，尤其是在“读”多于“写”的应用场景下。
5. 支持临时节点，当创建临时节点的客户端会话终结时，临时节点也被删除。而持久节点除非主动删除，不然一直存在。
6. ZooKeeper只负责：管理用户程序提交的数据；为用户程序提供数据节点监听服务。

# ZooKeeper中涉及的概念
## 会话(Session)
会话在客户端与ZooKeeper建立连接后就开始存在了。在创建会话前，ZooKeeper会为每个会话创建一个sessionID，这个ID必须是全局唯一的。

## Znode
ZooKeeper中的节点有2种：
1. 机器节点，表示构成集群的机器。
2. 数据节点，数据模型中的数据单元。

节点根据生命周期能被分为2种：
1. 持久节点。
2. 临时节点。

还能够给每个节点添加一个特殊属性：SEQUENTIAL。即在节点名后追加上一个整型数字。

## 版本
ZooKeeper会为每个ZNode维护一个Stat的数据结构。Stat记录了该ZNode的三个数据版本：version(当前ZNode版本)、cversion(当前ZNode子节点的版本)和aversion(当前ZNode的ACL版本)。

## 事件监听器(Watcher)
ZooKeeper允许用户在指定节点上注册一些Watcher，并且在一些特定事件触发的时候，ZooKeeper服务端会将事件通知到感兴趣的客户端上去。

## ACL(AccessControlLists)
类似于UNIX文件系统的权限控制。其中定义了以下5种权限：
1. CREATE
2. READ
3. WRITE
4. DELETE
5. ADMIN

# ZooKeeper的特点
1. 顺序一致性。
2. 原子性。
3. 单一系统映像。
4. 可靠性。

# ZooKeeper的设计目标
## 简单的数据模型
允许分布式进程通过共享的层次结构命名空间进行相互协调。
名称空间有数据寄存器组成。
数据存储在内存种。

## 可构建集群
以集群形态部署ZooKeeper来保证高可用性。

## 顺序访问
为每个更新请求分配一个全局唯一的递增编号，该编号反应了所有事务操作的先后顺序。

## 高性能
在“读”多于“写”的应用程序中尤为明显。

# ZooKeeper中的集群角色
典型的Master/Slave模式。

## Leader
由所有Follower机器选举产生。为客户端提供读写服务。

## Follower
为客户端提供读服务，并且能够通过选举称为Leader，参与“过半写成功”策略。

## Observer
仅提供读服务。

## 当Leader机器出现问题时，Follower机器如何成为Leader机器
1. Leader election(选举阶段)：所有Follower机器参与选举，只要有一个Follower机器节点得到超过半数节点的票数，就能够当选准Leader机器。
2. Discovery(发现阶段)：在这个阶段，Followers将最近接收的事务提议同步给准Leader。
3. Synchronization(同步阶段)：利用Leader前一阶段获得的最新提议历史，同步集群中所有的副本，同步完成后准Leader才会成为真正的Leader。
4. Broadcast(广播阶段)：到了这个阶段，ZooKeeper集群才能正式对外提供事务服务，并且Leader可以进行消息广播。同时如果有新的节点加入，还需要对新节点进行同步。

# ZooKeeper的ZAB协议
ZAB协议有两种模式：
1. 崩溃恢复：发现Leader崩溃后，进入该模式。
2. 消息广播：Leader正常后，进入该模式。


