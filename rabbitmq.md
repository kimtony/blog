
```
消息队列
交换器exchange 
队列绑定queue binding
消息消费
消息确认
持久化 durable
高可用和负载均衡 rabbitmq集群部署
```




### 常见问题
* 1、消息持久化
* 2、消息确认
* 3、死信队列 一种特殊的消息队列，用于存储无法被成功处理的消息
```
1、消息处理失败:当一个消息处理失败多次（超过了重试次数），可以将其放入死信队列，以避免消息不断重试影响系统性能。
2、消息被拒绝：如果消费者主动拒绝处理某条消息，可以将其转发到死信队列。
3、消息过期：消息在队列中等待消费的时间超过了预设的TTL（Time to Live），也可以将其转移到死信队列。
```






## 运行
### 拉取镜像
```
docker pull rabbitmq:3.8

docker run -d --hostname my-rabbit --name rabbit -p 15672:15672 -p 5672:5672 rabbitmq:3.8
```

### 启动管理界面
```
docker ps 
docker exec -it rabbit /bin/bash

## 宿主主机
wget -O rabbitmq_delayed_message_exchange-3.8.0.ez https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases/download/v3.8.0/rabbitmq_delayed_message_exchange-3.8.0.ez

docker cp rabbitmq_delayed_message_exchange-3.8.0.ez rabbit:/plugins/

## 开启插件
rabbitmq-plugins enable rabbitmq_management  rabbitmq_delayed_message_exchange
```

### 访问地址
ip:15672

guest guest





