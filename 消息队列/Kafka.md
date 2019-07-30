# Kafka

要点

- 消息模型
- Kafka 能否保证消息的顺序消费

## 消息模型

Kafka 也是使用的标准的发布-订阅模型。与 [RocketMQ](RocketMQ.md) 类似。唯一的区别是 Kafka 中队列不叫队列，而叫做分片（Partition），含义和功能是一样的。

## Kafka 能否保证消息的顺序消费

Kafka分布式的单位是partition，同一个partition用一个write ahead log组织，所以可以保证FIFO的顺序。不同partition之间不能保证顺序。

