# Kafka

要点

- Kafka 能否保证消息的顺序消费

## Kafka 能否保证消息的顺序消费

Kafka分布式的单位是partition，同一个partition用一个write ahead log组织，所以可以保证FIFO的顺序。不同partition之间不能保证顺序。

