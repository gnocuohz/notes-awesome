## 一、特性
### 1. 消息顺序
分区顺序，对于指定的一个 Topic，所有消息根据 sharding key 进行区块分区

### 2. 消息可靠性
同步刷盘：Broker异常关闭、OS Crash、掉电都可恢复
单点故障：可通过主从解决，但不使用同步双写也会丢消息

### 3. 回溯消息
RocketMQ支持按照时间回溯消费，时间维度精确到毫秒。

### 4. 事务消息

### 5. 定时消息（延迟队列）
broker有配置项messageDelayLevel，默认值为“1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h”，18个level。
定时消息会暂存在名为SCHEDULE_TOPIC_XXXX的topic中，并根据delayTimeLevel存入特定的queue，queueId = delayTimeLevel – 1，即一个queue只存相同延迟的消息，保证具有相同发送延迟的消息能够顺序消费。broker会调度地消费SCHEDULE_TOPIC_XXXX，将消息写入真实的topic。

### 6. 消息重试
RocketMQ会为每个消费组都设置一个Topic名称为“%RETRY%+consumerGroup”的重试队列（这里需要注意的是，这个Topic的重试队列是针对消费组，而不是针对每个Topic设置的），用于暂时保存因为各种异常而导致Consumer端无法消费的消息。
RocketMQ对于重试消息的处理是先保存至Topic名称为“SCHEDULE_TOPIC_XXXX”的延迟队列中，后台定时任务按照对应的时间进行Delay后重新保存至“%RETRY%+consumerGroup”的重试队列中。
http://tech.dianwoda.com/2018/02/09/rocketmq-reconsume/

## 二、架构
### 1. NameServer
无状态节点，可集群部署，节点之间无任何信息同步。
### 2. Broker
BrokerId为0表示Master，非0表示Slave。注意：当前RocketMQ版本在部署架构上支持一Master多Slave，但只有BrokerId=1的从服务器才会参与消息的读负载。  
每个Broker与NameServer集群中的**所有节点建立长连接**，定时注册Topic信息到所有NameServer。**心跳包中包含当前Broker信息(IP+端口等)以及存储所有Topic信息**。
### 3. Producer
与NameServer集群中的其中**一个节点（随机选择）建立长连接**，定期从NameServer获取Topic路由信息，并向提供Topic 服务的**所有Master建立长连接**（从NameServer中获取当前发送的Topic存在哪些Broker上，轮询从队列列表中选择一个队列，然后与队列所在的Broker建立长连接从而向Broker发消息），且**定时向Master发送心跳**。Producer完全无状态，可集群部署。
### 4. Consumer
与NameServer集群中的其中**一个节点（随机选择）建立长连接**，定期从NameServer获取Topic路由信息，并向提供Topic服务的**所有Master、Slave建立长连接**，且**定时向Master、Slave发送心跳**。  
Consumer既可以从Master订阅消息，也可以从Slave订阅消息，消费者在向Master拉取消息时，Master服务器会根据拉取偏移量与最大偏移量的距离（判断是否读老消息，产生读I/O），以及从服务器是否可读等因素建议下一次是从Master还是Slave拉取。

Producer、Consumer连一台NameServer，Broker连所有NameServer，Producer连一台Broker，Consumer连所有Broker。