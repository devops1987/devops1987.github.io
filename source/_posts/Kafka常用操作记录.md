---
title: Kafka常用操作记录
date: 2019/11/1
author: qfxl
catalog: true
tags:
    - Kafka
---

> 记录kafka日常操作，持续更新。

# 启动kafka
```shell
# /opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties &
或者（以守护进程的方式启动）：
# /opt/kafka/bin/kafka-server-start.sh -daemon /opt/kafka/config/server.properties
```

# kafka常用命令
```bash
# cd /opt/kafka
列出所有topic
# bin/kafka-topics.sh --zookeeper 192.168.98.53:2181 --list
查看某个topic详情
# bin/kafka-topics.sh --zookeeper 192.168.98.53:2181 --describe --topic metric
修改topic分区大小
# bin/kafka-topics.sh --zookeeper 192.168.98.53:2181 --alter --topic TransactionTopic --partitions 16
查看所有groups
# bin/kafka-consumer-groups.sh --bootstrap-server 192.168.98.28:9092 --list
```

**示例**：列出集群里的所有主题
```bash
# /opt/kafka/bin/kafka-topics.sh --zookeeper localhost:2181 --list
```

怎么知道每个节点的信息呢？运行“"describe topics”命令就可以了
```bash
# /opt/kafka/bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
```
下面解释一下这些输出。第一行是对所有分区的一个描述，然后每个分区都会对应一行，因为我们只有一个分区所以下面就只加了一行。
leader：负责处理消息的读和写，leader是从所有节点中随机选择的。
replicas：列出了所有的副本节点，不管节点是否在服务中。
isr：是正在服务中的节点。


## 查看groups详情，这个是动态的，一般可以作为是否采集到数据判断依据
```bash
# bin/kafka-consumer-groups.sh --bootstrap-server 192.168.98.28:9092 --describe --group hoa-server_event|column -t
```
> 其中依次展示group名称、消费的topic名称、partition id、consumer group最后一次提交的offset、最后提交的生产消息offset、消费offset与生产offset之间的差值、当前消费topic-partition的group成员id(不一定包含hostname)

## 手动设置kafka-logs日志存储时间，删除积压topic数据
```bash
# kafka-configs.sh --zookeeper 192.168.98.53:2181 --alter --entity-name hoa-server --entity-type topics --add-config retention.ms=5000     //5s
# kafka-configs.sh --zookeeper 192.168.98.53:2181 --alter --entity-name hoa-server --entity-type topics --add-config retention.ms=604800000      //168h，恢复为配置文件中设置
```

使用 --topics-with-overrides 参数可以找出所有包含覆盖配置的主题，它只会列出包含了与集群不一样配置的主题。
有两个参数可用于找出有问题的分区。使用 --under-replicated-partitions 参数可以列出所有包含不同步副本的分区 。 使用 --unavailable-partitions 参数可以列出所有没有首领的分区，这些分区已经处于离线状态，对于生产者和消费者来说是不可用的 。
**示例** ：列出包含不同步副本的分区。
```bash
# /opt/kafka/bin/kafka-topics.sh --zookeeper localhost:2181 --describe --under-replicated-partitions
# /opt/kafka/bin/kafka-topics.sh --zookeeper localhost:2181 --describe --unavailable-partitions
```


# 重点概念
## 分区平衡
在讲分区平衡前，先讲几个概念：
AR： assigned replicas，已分配的副本。每个partition都有自己的AR列表，里面存储着这个partition最初分配的所有replica。注意AR列表不会变化，除非增加分区。
PR（优先replica）：AR列表中的第一个replica就是优先replica，而且永远是优先replica。最初，优先replica和leader replica是同一个replica。
ISR：in sync replicas，同步副本。每个partition都有自己的ISR列表。ISR是会根据同步情况动态变化的。

最初ISR列表和AR列表是一致的，但由于某个节点死掉，或者某个节点的follower replica落后leader replica太多，那么该节点就会被从ISR列表中移除。此时，ISR和AR就不再一致。