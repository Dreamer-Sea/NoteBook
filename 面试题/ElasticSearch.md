# ElasticSearch

[toc]

# 选举流程
1. 节点node(第一个发现master宕机的节点)向所有比自己大的节点发送选举信息(选举为election信息)。
2. 如果节点node得不到任何回复(回复alive信息)，那么节点node称为master，并向所有其它节点宣布自己是master(宣布为Victory信息)。
3. 如果node得到了任何回复，node节点就一定不是master，同时等待victory信息，如果等待victory超时那么重新发起选举。