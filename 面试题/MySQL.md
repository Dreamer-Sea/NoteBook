# MySQL

[toc]

# 索引的缺点
1. 降低了更新速度，每次更新表的数据就要更新索引。
2. 索引会占据一定的物理空间，如果在大表上创建了多种组合索引，那么索引文件会膨胀很快。
3. 如果使用索引的列存在很多重复的数据，那么索引的效果就不明显。
4. 数据量少的表，走全表搜索更高效。

# 聚簇索引的限制
1. 插入速度**严重依赖于**插入顺序，按照主键的顺序插入是最快的方式。
2. 更新主键的代价很高，一般定义主键不可更新。
3. 二级索引访问需要两次索引查找。
4. 主键建议使用整型。

# 如何排查MySQL执行慢的原因
## 查询慢查询的数量
MySQL的慢查询数量保存在MySQL数据库的**slow_log表**。
```sql
SELECT * FROM slow_log where start_time > '2019/05/19 00:00:00';
```
## 查看当前进行的查询状态
```sql
select * from information_schema.processlist；
```
## explain命令查看慢的SQL语句的查询情况

## 设置慢查询时间

# 一张表，里面有 ID 自增主键，当 insert 了 17 条记录之后，删除了第 15,16,17 条记录，再把 MySQL 重启，再 insert 一条记录，这条记录的 ID 是 18 还是 15？
这个需要分使用的数据引擎讨论。
## InnoDB
在InnoDB的时候，答案是15。因为重启MySQL或对表进行优化(OPTIMIZE)操作，都会导致最大ID丢失。
## MyISAM
在使用MyISAM的时候，答案是18。因为MyISAM表会将自增主键的最大ID记录到数据文件里面。重启MySQL也不会令这个ID丢失。

# 如果表中有大字段，且该字段不经常更新，请问需不需要拆表？
分别讨论拆和不拆带来的问题。在实际中，会采取拆的方案。
## 拆
如果拆表，那么会带来**连接消耗** + **存储拆分空间**这两个问题。
## 不拆
如果不拆表，那就会影响**查询性能**。

# 主键和索引的区别
这里的主键和索引的区别一般指的是主键索引和唯一索引的区别。
- 主键
1. 主键的选择的字段最好是跟业务是不相关的，如MySQL给每个记录分配的id。
2. 每张表有且仅有一个主键(只有一个字段能作为主键)。
3. 字段的值不能存在空值。
4. 主键是索引的一种。

- 索引
1. 什么字段都能作为索引，只是拿重复内容越少的字段作为索引会有更好的性能表现。
2. 一张表能有多个索引。
3. 字段的值能够存在空值。
4. (唯一)索引不一定是主键。

# 为什么索引的底层实现是B+树而不是红黑树？
1. 为了减少IO的次数，B+树的节点大小刚好是一个页的大小。
2. 为了提高模糊搜索的性能，B+树用链表将叶子节点连接在一起。
3. B+树是多叉树，即一个节点可以有多个子节点，而红黑树的本质是二叉树，这样就会导致树特别深。

# MySQL如何保证一致性和持久性
使用了WAL(Write-Ahead Logging，先写日志再写磁盘)。Redo log就是一种WAL的应用。当数据库忽然断电，再重新启动时，MySQL可以通过Redo log还原数据。也就是说，每次事务提交时，不用同步刷新磁盘数据文件，只需要同步刷新Redo log就足够了。

# 索引在什么时候会失效？
1. 模糊查询：%like。
2. 索引列参与计算，使用了函数。
3. 非最左前缀顺序。
4. where对null判断。
5. where不等于。
6. or操作有至少一个字段没有索引。
7. 需要回表的查询结果集过大(超过配置的范围)。

# 最左匹配原则
原则是指在使用联合索引的情况下(例如：联合索引(a，b，c))，联合索引也是通过B+树来建立的，在这种情况下会按a先排序，再按b排序，最后按c排序。所以(a，c)的情况下，联合索引会失效。

# 事务的原子性是如何保证的？
事务的原子性是指一组数据库操作要么都成功要么都失败。

事务提交后会先写入**redo log**中，然后将相应的逆操作写入**undo log**中，成功之后才会写入**bin log**中，两个都成功后才会在内存满了的情况下写入磁盘。

**redo log**记录的是修改了数据库的SQL语句。
**undo log**记录的是相应操作的逆操作的SQL语句。
**bin log**记录的是数据库的变动。

# 聚合函数(组函数)
1. AVG：求平均数。
2. COUNT：统计。
3. MAX：求最大值。
4. MIN：求最小值。
5. SUM：求和。

# 连接
1. 内连接：连接两表共有的记录。
2. 左外连：保留左表的所有记录。
3. 右外连：保留右表的所有记录。
4. 全外连：MySQL不支持全外连，需要使用union联合左外连和右外连。

# 幻读是如何被解决的？
使用**间隙锁(Gap Lock)**能够解决(在InnoDB存储引擎中)。
当向数据库中加入一些值时(3个值，分别是0，5，10)，会产生一定的间隙((0, 5)，(5, 10))，当对数据表值进行修改的时候(例如在表中加入值6)，那么不仅会触发行锁，还会触发间隙锁，锁住有记录行和记录行之间的间隙(Next-key Lock)。这样就能够避免幻读的发生。
